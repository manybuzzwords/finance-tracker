<template>
    <div>
        <div class="font-bold" v-bind:class="{ 'green': trendingUp, 'red': !trendingUp }">{{ title }}</div>
        <div class="text-2xl font-extrabold text-black dark:text-white mb-2">
            <USkeleton class="h-8 w-full" v-if="loading" />
            <div v-else>{{ formattedAmount }}</div>
        </div>
        <div>
            <USkeleton class="h-6 w-full" v-if="loading" />
            <div v-else class="flex space-x-1 items-center text-sm">
                <UIcon :name="icon" class="w-6 h-6" :class="{ 'green': trendingUp, 'red': !trendingUp }" />
                <div class="text-gray-500 dark:text-gray-400">30% vs last period</div>
                <div>{{ trendPercentage }}</div>
            </div>
        </div>
    </div>
</template>

<script setup>
const props = defineProps({
    title: String,
    amount: Number,
    lastAmount: Number,
    color: String,
    loading: Boolean
});

const trendingUp = computed(() => {
    return props.amount >= props.lastAmount
})

const icon = computed(() => {
    return trendingUp.value ? 'i-heroicons-arrow-trending-up' : 'i-heroicons-arrow-trending-down'
})

const { currency: formattedAmount } = useCurrency(props.amount);
const { currency: formattedLastAmount } = useCurrency(props.lastAmount);

const trendPercentage = computed(() => {

    if (props.amount === 0 || props.lastAmount === 0) return 'No Change';
    const bigger = Math.max(props.amount, props.lastAmount);
    const lower = Math.min(props.amount, props.lastAmount);

    const ratio = ((bigger - lower) / lower) * 100;
    return `${Math.ceil(ratio)}%`

    //return Math.round((100 - (props.amount * 100) / props.lastAmount)) + '%';
})

</script>

<style scoped>
.green {
    @apply text-green-600 dark:text-green-400
}

.red {
    @apply text-red-600 dark:text-red-400
}
</style>