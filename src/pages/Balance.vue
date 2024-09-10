<template>
  <div class="balance">
    <h2><strong>Баланс</strong> {{ balance }}</h2>
    <p><strong>Всего заработано:</strong> {{ totalIncome }} UA</p>
    <p><strong>Всего потрачено:</strong> {{ totalExpense }} UA</p>

    <h3>Прогресс до целей:</h3>
    <ul>
      <li v-for="goal in goals" :key="goal.id">
        <p><strong>{{ goal.title }}:</strong> {{ goalProgress(goal.amount) }}% выполнено</p>
        <div class="progress-bar">
          <div class="progress" :style="{ width: goalProgress(goal.amount) + '%' }"></div>
        </div>
      </li>
    </ul>
  </div>
</template>

<script>
import { computed } from 'vue';
import { useBudgetStore } from '@/stores/budget';

export default {
  setup() {
    const budgetStore = useBudgetStore();

    const totalIncome = computed(() => budgetStore.totalIncome);  // Общая сумма доходов
    const totalExpense = computed(() => budgetStore.totalExpense);  // Общая сумма расходов
    const balance = computed(() => budgetStore.balance);  // Текущий баланс
    const goals = computed(() => budgetStore.goals);  // Список целей

    // Функция для расчета прогресса по целям
    const goalProgress = (goalAmount) => {
      const remaining = goalAmount - balance.value;
      const progress = Math.min((balance.value / goalAmount) * 100, 100);
      return progress.toFixed(2);
    };

    return {
      totalIncome,
      totalExpense,
      balance: computed(() => budgetStore.balance),
      goals,
      goalProgress,
    };
  },
};
</script>

<style scoped>
.balance {
  background-color: #aa80ee;
  color: white;
  padding: 20px;
  margin-bottom: 20px;
  text-align: center;
  border-radius: 8px;
}

h2 {
  margin-bottom: 20px;
}

h3 {
  margin-top: 20px;
}

.progress-bar {
  background-color: #ddd;
  border-radius: 6px;
  width: 100%;
  height: 20px;
  margin-top: 10px;
}

.progress {
  background-color: #6200ea;
  height: 100%;
  border-radius: 6px;
}

li {
  margin-bottom: 15px;
}
</style>
