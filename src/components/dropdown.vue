<template>
    <div class="dropdown" @mouseleave="hideTooltip">
        <div class="dropdown-container">
            <button @click="isOpen = !isOpen">
                {{ selectedOption }}
            </button>
            <div class="dropdown-content" v-show="isOpen">
                <div class="dropdown-item" v-for="option in options" :key="option.value" @click="selectOption(option)"
                    @mouseover="showTooltip(option)" :class="{ 'disabled-option': option.disabled }"
                    v-show="selectedOption !== option.text">
                    {{ option.text }}
                    <div v-if="showTooltipText === option.text" class="tooltip">
                        Login or register to access this option
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import { ref } from 'vue';
import authState from '@/tools/authState';

export default {
    setup() {
        const { userIsAuthenticated } = authState();
        const options = ref([
            { text: '1 hour', value: '1', disabled: false },
            { text: '4 hours', value: '4', disabled: !userIsAuthenticated.value },
            { text: '12 hours', value: '12', disabled: !userIsAuthenticated.value },
            { text: '1 Day', value: '24', disabled: !userIsAuthenticated.value },
            { text: '2 Days', value: '48', disabled: !userIsAuthenticated.value },
            { text: '3 Days', value: '72', disabled: !userIsAuthenticated.value },
        ]);

        const isOpen = ref(false);
        const selectedOption = ref(options.value[0].text); // Set the first option as the default selected option
        const showTooltipText = ref('');

        const selectOption = (option) => {
            if (!option.disabled) {
                selectedOption.value = option.text;
                isOpen.value = false;
            } else {
                showTooltip(option);
                setTimeout(() => {
                    hideTooltip();
                }, 3000);
            }
        };

        const showTooltip = (option) => {
            if (option.disabled) {
                showTooltipText.value = option.text;
            }
        };

        const hideTooltip = () => {
            showTooltipText.value = '';
        };

        return {
            userIsAuthenticated,
            isOpen,
            selectedOption,
            showTooltipText,
            options,
            selectOption,
            showTooltip,
            hideTooltip,
        };
    },
};
</script>

<!-- Styles remain the same -->

<style lang="scss" scoped>
@import '@/assets/style/grid.scss';

.dropdown {
    margin-top: .3rem;
    position: relative;
    width: 110px;
    border: 2px solid var(--primary-light);
    box-shadow: 0 0 20px rgba(66, 184, 131, 0.5);
    border-radius: 6px;
    z-index: 10;

}

button {
    display: block;
    width: 100%;
    padding: 10px 16px;
    text-align: left;
    border: none;
    background-color: var(--background-dark);
    cursor: pointer;
    color: var(--primary);
    font-size: 1rem;
    font-weight: bold;
    border-radius: 6px;
    text-align: center;
    
}

.dropdown-content {
    width: 110px;
    position: absolute;
    background-color: var(--background-dark);
    border-bottom-right-radius: 6px;
    border-bottom-left-radius: 6px;
    z-index: 15;
    border: 2px solid var(--primary-light);
    border-top: none;
    left: 50%;
    transform: translateX(-50%);
    margin-top: -8px;
}

.dropdown-item {
    color: var(--primary);
    padding: .5rem 1.3rem;
    cursor: pointer;
}

.dropdown-item:hover {
    background-color: var(--primary-dark);
}

.disabled-option {
    color: #999;
}

.tooltip {
    position: absolute;
    left: 100%;
    width: 200px;
    transform: translateY(-42px);
    margin: 10px 0 0 10px;
    color: var(--primary);
    background-color: var(--background-dark);
    border: 1px var(--primary-dark) solid;
    padding: 5px;
    z-index: 100;
}
</style>
