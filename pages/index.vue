<template>
  <div>
    Index page!
    <section class="flex items-center justify-between mb-10">
      <h1 class="text-4xl font-extrabold">Summary</h1>
      <div>
        <!-- <USelectMenu :options="people" v-model="selected" /> -->
        <USelectMenu :options="transactionsViewOptions" v-model="selectedView" />
      </div>
      <UIcon name="i-heroicons-light-bulb" class="w-5 h-5" />
    </section>

    <section class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 sm:gap-16 mb-10">
      <Trend :title="'Savings'" :color="'green'" :amount="incomeTotal" :last-amount="500" :loading="isLoading" />
      <Trend :title="'Expenses'" :color="'red'" :amount="expensesTotal" :last-amount="1225" :loading="isLoading" />
      <Trend :title="'Investments'" :color="'green'" :amount="1000" :last-amount="100" :loading="isLoading" />
      <Trend :title="'Savings'" :color="'red'" :amount="1000" :last-amount="0" :loading="isLoading" />
    </section>

    <section class="flex justify-between mb-10">
      <div>
        <h2 class="text-2xl font-extrabold">Transactions</h2>
        <div class="text-gray-500 dark:text-gray-400">You have {{ incomeCount }} Income Records and {{ expensesCount }}
          Expense Records.</div>
      </div>
      <div>
        <UButton icon="i-heroicons-plus-circle" color="white" variant="solid" label="Add" @click="isOpen = true"/>
        <UModal v-model="isOpen">
<h2 class="w-max h-max">tets</h2>
        </UModal>
      </div>
    </section>

    <section v-if="!isLoading">
      <div v-for="(transactionsOnDay, date) in transactionsGroupedByDate" :key="date" class="mb-2">
        <DailyTransactionSummary :date="date" :transactions="transactionsOnDay" />
        <Transaction v-for="transaction in transactionsOnDay" :key="transaction.id" :transaction="transaction"
          @deleted="refreshTransactions()" />
      </div>
      <!-- <DailyTransactionSummary :transactions="transactions" :date="new Date().toISOString().split('T')[0]"/> -->
      <!-- <Transaction v-for="transaction in transactions" :key="transaction.id" :transaction="transaction" /> -->
    </section>
    <section v-else>
      <USkeleton class="h-8 w-full mb-2" v-for="i in 4" :key="i" />
    </section>
  </div>
</template>

<script setup>

import { transactionsViewOptions } from '~/constants';
const isOpen = ref(false)
const selectedView = ref(transactionsViewOptions[1]);

const supabase = useSupabaseClient()
const transactions = ref([]);
const isLoading = ref(false);

const income = computed(() => {
  return transactions.value.filter(each => each.type === 'Income');
})

const expenses = computed(() => transactions.value.filter(each => each.type === 'Expense'));

const incomeCount = computed(() => income.value.length);
const expensesCount = computed(() => expenses.value.length);

const incomeTotal = computed(() => income.value.reduce((accumulator, currentElement) => accumulator += currentElement.amount, 0));
const expensesTotal = computed(() => expenses.value.reduce((accumulator, currentElement) => accumulator += currentElement.amount, 0));


const fetchTransactions = async () => {
  try {
    isLoading.value = true;
    const { data } = await useAsyncData('transactions', async () => {
      const { data, error } = await supabase.from('transactions').select();

      if (error) return [];

      return data;
    })

    return data.value;
  } finally {
    isLoading.value = false;
  }
}

const refreshTransactions = async () => transactions.value = await fetchTransactions();

await refreshTransactions();

const transactionsGroupedByDate = computed(() => {
  let grouped = {};

  for (const transaction of transactions.value) {
    const date = new Date(transaction.created_at).toISOString().split('T')[0];

    if (!grouped[date]) {
      grouped[date] = [];
    }

    grouped[date].push(transaction);
  }

  console.log(grouped);
  return grouped;
})


console.log(transactionsGroupedByDate.value);
</script>

<style></style>