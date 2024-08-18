<template>
    <main>
        <!-- <h1>About</h1> -->
        <p class="aboutText">{{ aboutText }}</p>
        <div class="grid">
            <section v-for="(card, index) in cards" :key="index" :class="{ 'active': activeCardIndex === index }"
                @click="activeCardIndex === index ? activeCardIndex = null : activeCardIndex = index">
                <div class="card">
                    <h2>{{ card.title }}</h2>
                    <p>{{ card.content }}</p>
                </div>
            </section>
        </div>
    </main>
</template>
<script>

export default {
    data() {
        return {
            activeCardIndex: null,
            aboutText: 'At RelayFox, we’re redefining how you share and protect your data. Experience effortless, secure file transfers with a service that’s built to handle anything you need to send.',
            cards: [
                {
                    title: 'End-to-End Encryption',
                    content: 'At RelayFox, your privacy is our top priority. That’s why we utilize end-to-end encryption to secure your files from the moment they leave your device until they reach their intended recipient. This means that only you and the person you’re sending files to can decrypt and access the data. No one else—not even us—can access your files. With end-to-end encryption, you can trust that your sensitive information remains confidential and protected, every step of the way.'
                },
                {
                    title: '256-Bit Encryption',
                    content: 'RelayFox employs 256-bit encryption, the gold standard in data security. This level of encryption is used by governments and financial institutions worldwide, making it virtually impossible for unauthorized parties to crack. When you send files with RelayFox, they’re encrypted with 256-bit keys, ensuring that your data is protected against even the most sophisticated cyber threats. Whether you’re sharing personal documents or critical business data, 256-bit encryption provides the ultimate safeguard, giving you peace of mind that your information is secure.'
                },
                {
                    title: 'Zero-Knowledge Architecture',
                    content: 'With RelayFox, you can share files with absolute confidence using one-time links. These links are designed for single use, meaning once your recipient downloads the file, the link automatically expires. This eliminates the risk of unauthorized access, ensuring your data remains private. Additionally, you can set a time limit on the link, after which it will self-destruct, even if it hasn’t been used. Whether you’re sharing sensitive documents or large files, one-time links provide an extra layer of security, making sure your data is only accessed by the intended recipient, within a timeframe that you control.'
                },
                {
                    title: 'Unlimited File Sizes',
                    content: 'RelayFox is built to handle your biggest files with ease, supporting truly unlimited file sizes. During our current testing phase, we’re offering a generous 128GB limit per transfer to all users for free. This allows you to share large files like HD videos, extensive datasets, or complex project files effortlessly, with our chunk-based system ensuring smooth and secure transfers. As we finalize our service, we plan to roll out premium options that unlock the full potential of RelayFox, enabling completely unlimited file transfers. For now, enjoy the robust 128GB limit as we fine-tune our service to meet all your data-sharing needs.'
                },
                {
                    title: 'Privacy Policy',
                    content: 'At RelayFox, we take your privacy seriously. Our commitment to trust and security means we never collect or store your personal data. The only thing we ever know is the estimated file size—nothing more. All file names, types, and metadata are fully encrypted on your device before transfer, ensuring that no one, not even us, can access this information. With end-to-end encryption and zero data collection, RelayFox ensures your files remain completely private and secure from the moment they leave your device until they reach their destination.'
                },
                {
                    title: 'Our Goals',
                    content: 'At RelayFox, our mission is to provide a secure, user-friendly file-sharing service that meets the needs of individuals and businesses alike. We’re committed to delivering a seamless experience that prioritizes privacy and security, ensuring that your data remains confidential and protected at all times. With RelayFox, you can share files with confidence, knowing that your information is safeguarded by the latest encryption standards and best practices. Our goal is to redefine how you share and protect your data, making secure file transfers effortless and accessible to everyone.'
                }
            ],
        }
    }
}
</script>
<style lang="scss" scoped>
@import '@/assets/style/grid.scss';

.grid {
    @include grid-layout1();
}

main {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    .aboutText {
        align-self: center;
        margin: 3rem 20%;
        font-size: 1.5rem;
        background-color: var(--background-dark);
        position: relative;
        padding: .8rem;
        

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

        @media (max-width: 768px) {
            margin: 1.5rem 15%;
        }

        @media (max-width: 480px) {
            margin: 1.5rem 10%;
            font-size: 1.3rem;
        }

        @media (max-width: 350px) {
            margin: 1.5rem 5%;
            font-size: 1.1rem;
            // width: 200px;
        }
    }
}

section {
    grid-column: 2;
    display: flex;
    background-image: var(--linear-gradient);
    border-radius: 20px;
    transition: all .3s;
    cursor: pointer;
    margin-bottom: 2rem;

    @media (min-width: 1000px) {
        margin-bottom: 0;
    }
    &:hover,
    &.active {
        box-shadow: 0px 0px 30px 1px rgba(0, 255, 117, 0.30);
        scale: 1.1;

        .card {
            transform: scale(0.98);
            border-radius: 20px;
        }
    }

    .card {
        background-color: var(--background-dark);
        border-radius: 0;
        transition: all .2s;
        padding: 1.5rem;

        &:hover {
            transform: scale(0.98);
            border-radius: 20px;
        }
    }
}

@media (min-width: 1000px) {
    section:nth-child(odd) {
        grid-column: 2;
    }

    section:nth-child(even) {
        grid-column: 3;
    }
}

h2 {
    font-size: 1.5rem;
    margin-bottom: 1rem;
    font-family: "Roboto", sans-serif;
    font-weight: 500;
    font-style: normal;
    color: var(--primary);
    border-bottom: 2px solid var(--primary);
}

p {
    font-size: 1rem;
    margin-bottom: 1rem;
    color: var(--primary-light);

}
</style>
