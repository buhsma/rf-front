<template>
    <div class="grid">
    <main>

        <h1>Share Smart with RelayFox.</h1>
        <BluePortal />
        <p>RelayFox makes sharing big files effortless and secure. With top-notch encryption and no limits on what you
            can send, your data is always safe and sound. Send with confidence, share with ease.</p>

    </main>
    </div>
</template>

<script>
import authState from '@/tools/authState';
import BluePortal from '@/components/BluePortal.vue';
export default {
    components: {
        BluePortal,
    },
    setup() {
        const { userIsAuthenticated } = authState();
        return { userIsAuthenticated };
    },
};
</script>

<style lang="scss" scoped>
@import '@/assets/style/home.scss';
</style>