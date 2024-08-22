<template>
    <div class="grid">
        <div v-if="status === 'ready'">
            <p>To keep your data safe, we’ve encrypted everything before it was sent. Click the button below to reveal
                the secret and access your file. Remember, this is a one-time link—once you view the secret, the link
                will
                expire. </p>
            <button class="btn1" @click="fetchSecret">Show Secret</button>
        </div>
        <div class="showBox" v-else-if="status === 'done'">
            <span>{{ secret }}</span>
            <button class="btn1" @click="copySecret">Copy Secret</button>
        </div>
    </div>
</template>
<style lang="scss" scoped>
@import '@/assets/style/btn1.scss';
@import '@/assets/style/showBox.scss';
@import '@/assets/style/grid.scss';

.grid {
    @include grid-layout2();
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
                padding: .3rem 1.5rem;
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
    }
}
</style>
<script>
import { ref } from 'vue';
import axios from 'axios';
import { revertKey } from '@/tools/crypto';

export default {
    props: {
        id: String,
        cryptoKey: String
    },
    setup(props) {
        const status = ref('ready');
        const secret = ref('');
        const BACKEND_URL = import.meta.env.VITE_BACKEND_URL;   

        const handleDecryptSecret = async (secretString, ivString, key) => {
            // Convert the iv and secret back to their original formats
            const iv = Uint8Array.from(atob(ivString), c => c.charCodeAt(0));
            const secretEncrypted = Uint8Array.from(atob(secretString), c => c.charCodeAt(0));
            key = await revertKey(key);
            const importedKey = await crypto.subtle.importKey(
                'jwk',
                key,
                { name: 'AES-GCM', length: 256 },
                false,
                ['decrypt']
            );
            // Decrypt the secret
            const secretBuffer = await crypto.subtle.decrypt(
                {
                    name: "AES-GCM",
                    iv: iv
                },
                importedKey,
                secretEncrypted
            );

            // Convert the decrypted secret back to a string
            const decryptedSecret = new TextDecoder().decode(secretBuffer);

            return decryptedSecret;
        };

        const fetchSecret = async () => {
            try {
                const response = await axios.post(`${BACKEND_URL}/api/secret/${props.id}/`);
                secret.value = await handleDecryptSecret(response.data.secret, response.data.iv, props.cryptoKey);
                status.value = 'done';

                await axios.post(`${BACKEND_URL}/api/delete/${props.id}/`);
            } catch (error) {
                console.error(error);
            }
        };

        const copySecret = () => {
            navigator.clipboard.writeText(secret.value)
                .then(() => {
                    console.log('Link copied to clipboard');
                })
                .catch(err => {
                    console.error('Could not copy link: ', err);
                });
        };

        return {
            status,
            secret,
            fetchSecret,
            copySecret
        }
    }
};
</script>