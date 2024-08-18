<template class="template" >
    <!-- <h1>BluePortal</h1> -->
    <div v-if="status === 'input'">
        <!-- <label for="file">Select a file:</label>
        <input type="file" @change="handleFileUpload">
        <div v-if="userIsAuthenticated">
            <label for="linkLifetime">Link lifetime:</label>
            <select v-model="linkLifetime">
                <option value="1">1 hour</option>
                <option value="4">4 hours</option>
                <option value="12">12 hours</option>
                <option value="24">1 Day</option>
                <option value="48">2 Days</option>
                <option value="72">3 Days</option>
            </select>
        </div>
        <div v-else>
            <p>
                Please log in to upload files larger than 25MB. Note that for unauthenticated users, the link lifetime
                is fixed at 1 hour.
            </p>
        </div> -->
        <InputField @file="handleFileUpload" @input="handleSecret" />
    </div>

    <div class="progressbar" v-else-if="status === 'uploading'">
        <div>
            <label>Encryption Progress:</label>
            <progress :value="encryptProgress" max="100"></progress>
            <span>{{ Math.floor(encryptProgress) }}%</span>
        </div>
        <div>
            <label>Upload Progress:</label>
            <progress :value="isNaN(uploadProgress) ? 0 : uploadProgress" max="100"></progress>
            <span>{{ isNaN(uploadProgress) ? 0 : Math.floor(uploadProgress) }}%</span>
        </div>
    </div>
    <div class="showBox" v-else-if="status === 'done'">
        <span>{{ link }}</span>
        <button class="btn1" @click="copyLink">Copy Link</button>
    </div>
</template>

<style lang="scss" scoped>
@import '@/assets/style/progressbar.scss';
@import '@/assets/style/showBox.scss';
    

</style>

<script>

import { v4 as uuidv4 } from 'uuid';
import axios from 'axios';
import { ref } from 'vue';
import { getKey, convertKey } from '@/tools/crypto';
import protobuf from 'protobufjs';
import authState from '@/tools/authState';
import InputField from '@/components/InputField.vue';
// import FileChunk from '@/protobuf/protoLoader';


export default {
    components: {
        InputField
    },
    setup() {
        const { userIsAuthenticated } = authState();
        const maxSimultaneousUploads = 5;
        const chunkSizeMB = 1;
        const status = ref('input');
        const id = ref(uuidv4());
        const link = ref('');
        const uploadQueue = ref([]);
        const uploadProgress = ref(0);
        const encryptProgress = ref(0);
        const worker = new Worker(new URL('@/tools/blueWorker.js', import.meta.url));
        const FRONTEND_URL = import.meta.env.VITE_FRONTEND_URL;
        //load proto files
        //just for development
        let FileChunk;
        fetch('src/protobuf/fileChunk.proto')
            .then(response => response.text())
            .then(proto => {
                const root = protobuf.parse(proto).root;

                FileChunk = root.lookupType("fileChunk.FileChunk");
            });
        let Meta;
        fetch('src/protobuf/meta.proto')
            .then(response => response.text())
            .then(proto => {
                const root = protobuf.parse(proto).root;

                Meta = root.lookupType("meta.Meta");
            });

        let chunksUploaded = 0;
        let chunksEncrypted = 0;


        const handleSecret = async (secret) => {
            const key = await getKey();
            const urlSaveKey = await convertKey(key);
            link.value = `${FRONTEND_URL}/secret/${id.value}/${urlSaveKey}`;
            // encrypt secret
            const iv = crypto.getRandomValues(new Uint8Array(12));
            const secretBuffer = new TextEncoder().encode(secret.text);
            const secretEncrypted = await crypto.subtle.encrypt(
                {
                    name: "AES-GCM",
                    iv: iv
                },
                key,
                secretBuffer
            );
            const ivString = btoa(String.fromCharCode.apply(null, iv));
            const secretString = btoa(String.fromCharCode.apply(null, new Uint8Array(secretEncrypted)));

            let headers = {
                'Content-Type': 'application/json',
            };
            if (userIsAuthenticated.value) {
                headers['Authorization'] = `Bearer ${localStorage.getItem('access')}`;
            }

            try {
                await axios.post('/api/secretup/', {
                    secret: secretString,
                    id: id.value,
                    iv: ivString,
                    linkLifetime: secret.linkLifetime
                }, {
                    headers: headers
                });
                status.value = 'done';
            } catch (error) {
                console.error(`failed to upload secret ${error}`);
            }
        }


        const handleFileUpload = async (file) => {
            const linkLifetime = file.linkLifetime;
            file = file.file;
            console.log('userIsAuthenticated', userIsAuthenticated);
            const fileName = file.name;
            const chunkSize = 1024 * 1024 * chunkSizeMB;
            const totalChunks = Math.ceil(file.size / chunkSize);
            if (!userIsAuthenticated.value && totalChunks > 25) {
                alert('Please login to upload files larger than 25MB');
                return;
            }
            const key = await getKey();
            const urlSaveKey = await convertKey(key);

            //var in env
            link.value = `${FRONTEND_URL}/download/${id.value}/${urlSaveKey}`;
            status.value = 'uploading';

            const metaData = Meta.create({
                fileName: fileName,
                totalChunks: totalChunks
            });
            const metaBuffer = Meta.encode(metaData).finish();
            const iv = crypto.getRandomValues(new Uint8Array(12));
            const ivString = btoa(String.fromCharCode.apply(null, iv));
            const metaEncrypted = await crypto.subtle.encrypt(
                {
                    name: "AES-GCM",
                    iv: iv
                },
                key,
                metaBuffer
            );
            console.log('iv', ivString);
            await uploadMetaChunk(metaEncrypted, id.value, totalChunks, linkLifetime, ivString);

            let ongoingUploads = 0;


            worker.onmessage = async (event) => {
                const encryptedChunk = event.data;
                chunksEncrypted++;
                encryptProgress.value = (chunksEncrypted / totalChunks) * 100;

                let message = FileChunk.create({
                    chunk: new Uint8Array(encryptedChunk.chunk),
                    iv: encryptedChunk.iv
                });

                let buffer = FileChunk.encode(message).finish();

                uploadQueue.value.push({ buffer, index: encryptedChunk.index });

                if (ongoingUploads < maxSimultaneousUploads) {
                    uploadNextChunk();
                }
            }

            async function uploadNextChunk() {
                if (uploadQueue.value.length === 0) {
                    return;
                }

                ongoingUploads++;
                const { buffer, index } = uploadQueue.value.shift();
                try {
                    await uploadChunk(buffer, id.value, index, totalChunks);
                } catch (error) {
                    console.error(`failed to upload chunk ${error}`);
                } finally {
                    ongoingUploads--;
                    if (uploadQueue.value.length > 0) {
                        uploadNextChunk();
                    }
                }
            }

            let index = 0;
            for (let start = 0; start < file.size; start += chunkSize, index++) {
                const chunk = file.slice(start, start + chunkSize);
                const iv = crypto.getRandomValues(new Uint8Array(12));

                while (index > chunksUploaded + (maxSimultaneousUploads * 2)) {
                    await new Promise(resolve => setTimeout(resolve, 100));
                }

                worker.postMessage({
                    index,
                    chunk: chunk,
                    key,
                    iv
                });
            }
        }
        const uploadMetaChunk = async (buffer, id, totalChunks, linkLifetime, iv, retryCount = 0) => {
            // Create headers object
            console.log('linkLifetime', linkLifetime);
            let headers = {
                'Content-Type': 'application/octet-stream',
                'id': id,
                'index': 'meta',
                'linkLifetime': linkLifetime,
                'totalChunks': totalChunks,
                'iv': iv
            };


            if (userIsAuthenticated.value) {
                headers['Authorization'] = `Bearer ${localStorage.getItem('access')}`;
            }

            try {
                await axios.post(`/api/upload/`, buffer, {
                    headers: headers,
                    responseType: 'arraybuffer'
                });
            } catch (error) {
                console.error(`failed to upload meta chunk ${error}`);
                if (retryCount < 3) {
                    console.log(`retrying upload... attempt ${retryCount + 1}`);
                    await uploadMetaChunk(buffer, id, totalChunks, linkLifetime, iv, retryCount + 1);
                } else {
                    console.error(`upload of meta chunk failed after 3 attempts`);
                    throw error;
                }
            }
        }


        const uploadChunk = async (buffer, id, index, totalChunks, retryCount = 0) => {
            try {
                await axios.post(`/api/upload/`, buffer, {
                    headers: {
                        'Content-Type': 'application/octet-stream',
                        'id': id,
                        'index': index,
                    },
                    responseType: 'arraybuffer'
                });

                chunksUploaded++;
                uploadProgress.value = (chunksUploaded / totalChunks) * 100;
                buffer = null;
                if (chunksUploaded === totalChunks) {
                    console.log('All chunks uploaded');
                    worker.terminate();
                    status.value = 'done';

                }
            } catch (error) {
                console.error(`failed to upload chunk ${error}`);
                if (retryCount < 3) {
                    console.log(`retrying upload... attempt ${retryCount + 1}`);
                    await uploadChunk(buffer, id, index, totalChunks, retryCount + 1);
                } else {
                    console.error(`upload of chunk ${index} failed after 3 attempts`);
                    throw error;
                }
            }
        }

        const copyLink = () => {
            navigator.clipboard.writeText(link.value)
                .then(() => {
                    console.log('Link copied to clipboard');
                })
                .catch(err => {
                    console.error('Could not copy link: ', err);
                });
        };


        return {
            userIsAuthenticated,
            handleFileUpload,
            handleSecret,
            uploadProgress,
            encryptProgress,
            status,
            link,
            copyLink
        }
    }
}

</script>