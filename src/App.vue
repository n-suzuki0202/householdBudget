<template>
    <Header />
    <div class="container">
        <Balance :total="total" />
        <IncomeExpenses :income="+income" :expenses="+expenses" />
        <TransactionList :transactions="transactions" @transactionDeleted="handleTransactionDeleted" />
        <AddTransaction @transactionSubmitted="handleTransactionSubmitted" />
    </div>
</template>

<script setup>
import Header from './components/Header.vue';
import Balance from './components/Balance.vue';
import IncomeExpenses from './components/IncomeExpenses.vue';
import TransactionList from './components/TransactionList.vue';
import AddTransaction from './components/AddTransaction.vue';

import { useToast } from 'vue-toastification';
import { ref, computed, onMounted } from 'vue';

const toast = useToast();

const transactions = ref([]);

onMounted(() => {
    const savedTransactions = JSON.parse(localStorage.getItem('transactions'));
    if (savedTransactions) transactions.value = savedTransactions;
})

const total = computed(() => transactions.value.reduce((sum, t) => sum + t.amount, 0));
const income = computed(() => transactions.value.filter(t => t.amount > 0).reduce((sum, t) => sum + t.amount, 0).toFixed(2));
const expenses = computed(() => transactions.value.filter(t => t.amount < 0).reduce((sum, t) => sum + t.amount, 0).toFixed(2));

const handleTransactionSubmitted = transactionData => {
    transactions.value.push({
        id: generateUniqueid(),
        text: transactionData.text,
        amount: transactionData.amount,
    });
    saveTransactionsToLocalStorage();
    toast.success('Transaction added');
};
const generateUniqueid = () => transactions.value.length > 0 ? transactions.value[transactions.value.length - 1].id + 1 : 1;

const handleTransactionDeleted = id => {
    transactions.value = transactions.value.filter(t => t.id !== id);
    saveTransactionsToLocalStorage();
    toast.success('Transaction deleted');
};

const saveTransactionsToLocalStorage = () => localStorage.setItem('transactions', JSON.stringify(transactions.value));
</script>
