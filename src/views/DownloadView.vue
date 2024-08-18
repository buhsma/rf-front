<template>
    <main>
            <OrangePortal v-if="$route.name === 'download'" :id="$route.params.id" :cryptoKey="$route.params.cryptoKey" />
            <Secret v-else :id="$route.params.id" :cryptoKey="$route.params.cryptoKey" />
    </main>
</template>

<script>
import OrangePortal from '@/components/OrangePortal.vue';
import Secret from '@/components/Secret.vue';

export default {
    components: {
        OrangePortal,
        Secret
    },
};

</script>