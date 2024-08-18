<template>
    <div class="grid grid-cols-2 py-4 border-b border-gray-200 dark:border-gray-600 text-gray-900 dark:text-gray-100">
        <div class="flex items-center justify-between">
            <div class="flex items-center space-x-2">
                <!-- <UIcon :name="icon" :class="{'text-green-600': transaction.type === 'Income', 'text-red-600': transaction.type === 'Expense'}" /> -->
                <UIcon :name="icon" :class="[iconColor]" />
                <div>{{ transaction.description }}</div>
            </div>
            <div>
                <UBadge color="white" v-if="transaction.category">{{ transaction.category }}</UBadge>
            </div>
        </div>
        <div class="flex items-center justify-end space-x-2">
            <div>{{ currency }}</div>
            <div>
                <UDropdown :items="items" :popper="{ placement: 'bottom-start' }">
                    <UButton color="white" variant="ghost" trailing-icon="i-heroicons-ellipsis-horizontal"
                        :loading="isLoading" />
                </UDropdown>
            </div>
        </div>
    </div>
</template>

<script setup>
const toast = useToast();
const isLoading = ref(false);
const supabase = useSupabaseClient();
const emit = defineEmits(['deleted']);


const deleteTransaction = async () => {
    console.log('deleted');
    isLoading.value = true;
    try {
        await supabase.from('transactions')
            .delete()
            .eq('id', props.transaction.id)
        toast.add({
            title: 'transaction deleted',
            icon: 'i-heroicons-check-circle',
            color: 'green'
        })
        emit('deleted', props.transaction.id);
    } catch (error) {
        toast.add({
            title: 'something went wrong',
            icon: 'i-heroicons-exclamation-circle',
            color: 'red'
        })
    } finally {
        isLoading.value = false;
    }
}

const props = defineProps({
    transaction: Object
})

const consumeEdit = function () {
    console.log('Edited')
}
const { currency } = useCurrency(props.transaction.amount);
const items = [
    [
        {
            label: 'Edit',
            icon: 'i-heroicons-pencil-square-20-solid',
            click: consumeEdit
        },
        {
            label: 'Delete',
            icon: 'i-heroicons-trash-20-solid',
            click: deleteTransaction
        }
    ],

];

const isIncome = computed(() => {
    return props.transaction.type === 'Income';
})

const icon = computed(() => {
    return isIncome.value ? 'i-heroicons-arrow-up-right' : 'i-heroicons-arrow-down-left'
})

const iconColor = computed(() => {
    return isIncome.value ? 'text-green-600' : 'text-red-600';
})

</script>

<style scoped></style>