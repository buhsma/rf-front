<template>
    <div class="grid">
        <div v-if="status === 'ready'">
            <p>To keep your data safe, we’ve encrypted everything before it was sent. Click the button below to download and access your file. Remember, this is a one-time link—once you download the file, the link will
                expire. </p>
            <button class="btn1" @click="handleDownload">Download</button>
        </div>
        <div v-else-if="status === 'downloading'" class="progressbar">
            <div>
                <label for="downloadProgress">Download Progress:</label>
                <progress id="downloadProgress" :value="downloadProgress" max="100"></progress>
                <span>{{ Math.floor(downloadProgress) }}%</span>
            </div>
        </div>
        <div class="done" v-else-if="status === 'done'">
            <p>Download complete. Your file is ready.</p>
        </div>
    </div>
</template>

<style lang="scss" scoped>
@import '@/assets/style/progressbar.scss';
@import '@/assets/style/btn1.scss';
@import '@/assets/style/showBox.scss';
@import '@/assets/style/grid.scss';

.grid {
    @include grid-layout();
    align-items: center;
    height: 100vh;

    & div {
        grid-column: 2;
            display: flex;
            flex-direction: column;
            align-items: center;

        & button {
            width: 16rem;

            @media (max-width: 768px) {
                width: 100%;
            }
        }
        & span {
            font-size: 1.5rem;
            text-align: left;
            overflow: auto;
            
        }
    }
}

p {
    display: inline-block;
    width: 100%;
    margin: 3rem 0;
    background-color: var(--background-dark);
    position: relative;
    padding: .3rem 1.5rem;
    font-size: 1.5rem;
    
    color: var(--primary-light);

    @media (max-width: 768px) {
        font-size: 1rem;
    }

    &:after {
        content: '';
        display: block;
        width: 100%;
        height: 100%;
        background-image: var(--linear-gradient);
        position: absolute;
        top: +.5rem;
        left: -.5rem;
        z-index: -1;
    }
}
.done {
    text-align: center;
}
</style>

<script>
import { ref, onBeforeMount } from 'vue';
import { revertKey } from '@/tools/crypto';
import streamSaver from 'streamsaver';
import protobuf from 'protobufjs';
import { decodeFileChunk } from '@/protobuf/fileChunk.js';
import { decodeMeta } from '@/protobuf/meta.js';




export default {
    props: {
        id: String,
        cryptoKey: String
    },
    setup(props) {
        const worker = new Worker(new URL('@/tools/orangeWorker.js', import.meta.url));
        const status = ref('ready');
        const downloadProgress = ref(0);
        const chunkBuffer = [];
        const chunkBufferMax = 3;
        const BACKEND_URL = import.meta.env.VITE_BACKEND_URL;
        

        let writer;



        // let FileChunk;
        // //half a day to find a / in front of src/protobuf/ was missing. works jusr fine without it in the bluePortal
        // fetch('/src/protobuf/fileChunk.proto')
        //     .then(response => response.text())
        //     .then(proto => {
        //         const root = protobuf.parse(proto).root;

        //         FileChunk = root.lookupType("fileChunk.FileChunk");
        //     });
        // let Meta;
        // fetch('/src/protobuf/meta.proto')
        //     .then(response => response.text())
        //     .then(proto => {
        //         const root = protobuf.parse(proto).root;

        //         Meta = root.lookupType("meta.Meta");
        //     });

        const processChunkBuffer = async (key, totalChunks) => {
            while (status.value === 'downloading' || chunkBuffer.length > 0) {
                if (chunkBuffer.length === 0) {
                    await new Promise(resolve => setTimeout(resolve, 100));
                    continue;
                }
                const { index, chunk, iv } = chunkBuffer.shift();
                worker.postMessage({
                    chunk: chunk,
                    iv: iv,
                    key: key,
                    index: index
                });
                await new Promise(resolve => {
                    worker.onmessage = (event) => {
                        if (writer) {
                            const decryptedChunk = new Uint8Array(event.data.decryptedChunk);
                            writer.write(decryptedChunk);
                            downloadProgress.value = (index + 1) / totalChunks * 100;
                            if (index === totalChunks - 1) {
                                writer.close();
                            }
                        }
                        resolve();
                    }
                });
                if (index === totalChunks - 1) {
                    break;
                }
            }
        };
        const fetchData = async (writer, totalChunks) => {
            console.log('fetchData');
            for (let index = 0; index < totalChunks; index++) {
                let retryCount = 0;
                while (retryCount < 3) {
                    try {
                        console.log('fetching chunk', index);
                        const response = await fetch(`${BACKEND_URL}/api/download/${props.id}/${index}`);
                        console.log('response', response);
                        const value = await response.arrayBuffer();
                        console.log('value', value);
                        const decoded = decodeFileChunk(new Uint8Array(value));
                        console.log('decoded', decoded);
                        chunkBuffer.push({ index: index, chunk: decoded.chunk, iv: decoded.iv });

                        while (chunkBuffer.length >= chunkBufferMax) {
                            await new Promise(resolve => setTimeout(resolve, 100));
                        }

                        break;
                    } catch (error) {
                        console.error('An error occurred:', error);
                        retryCount++;
                        if (retryCount >= 3) {
                            throw new Error(`Failed to fetch chunk ${index} after 3 attempts`);
                        }
                    }
                }
            }
        };
        const fetchMeta = async (id, key) => {
            console.log('fetchMeta');
            const response = await fetch(`${BACKEND_URL}/api/download/${id}/meta`);
            console.log('response', response);
            const ivString = response.headers.get('iv');
            console.log('ivString', ivString);
            const value = await response.arrayBuffer();
            console.log('value', value);
            const importedKey = await crypto.subtle.importKey(
                'jwk',
                key,
                { name: 'AES-GCM', length: 256 },
                false,
                ['decrypt']
            );
            console.log('importedKey', importedKey);
            const iv = Uint8Array.from(atob(ivString), c => c.charCodeAt(0));
            const metaDecrypted = await crypto.subtle.decrypt(
                {
                    name: 'AES-GCM',
                    iv: iv
                },
                importedKey,
                value
            );
            console.log('metaDecrypted', metaDecrypted);
            const meta = decodeMeta(new Uint8Array(metaDecrypted));
            console.log('meta', meta);
            return meta;
        };
        const sendConfirmation = async () => {
            await fetch(`${BACKEND_URL}/api/confirm/${props.id}/`, {
                method: 'POST'
            });
        };


        const handleDownload = async () => {
            status.value = 'downloading';
            // console.log('props.cryptoKey', props.cryptoKey);
            // console.log('props.id', props.id);
            const key = await revertKey(props.cryptoKey);
            const metadata = await fetchMeta(props.id, key);
            console.log(metadata);
            const fileStream = streamSaver.createWriteStream('ScureShare-' + metadata.fileName);
            writer = fileStream.getWriter();
            fetchData(writer, metadata.totalChunks);
            await processChunkBuffer(key, metadata.totalChunks);
            worker.terminate();
            if (writer) {
                writer.releaseLock();
            }
            status.value = 'done';
            sendConfirmation();
        };

        return {
            handleDownload,
            status,
            downloadProgress
        };
    }
};

</script>