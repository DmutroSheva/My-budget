<template>
  <div class="tracker">
    <h2>Доходы</h2>
    <form @submit.prevent="addIncomeHandler" class="income-form">
      <select v-model="incomeCategory" required class="input-field">
        <option disabled value="">Выберите категорию</option>
        <option value="Зарплата">Зарплата</option>
        <option value="Фриланс">Фриланс</option>
        <option value="Инвестиции">Инвестиции</option>
        <option value="Прочее">Прочее</option>
      </select>
      <input v-model.number="incomeAmount" placeholder="Сумма" type="number" required class="input-field" />
      <div v-if="showAllocation" class="allocation-form">
        <h3>Отложить деньги на цели</h3>
        <select v-model="selectedGoal" required class="input-field">
          <option disabled value="">Выберите цель</option>
          <option v-for="goal in goals" :key="goal.id" :value="goal">{{ goal.title }}</option>
        </select>
        <input v-model.number="allocationAmount" placeholder="Сумма для откладывания" type="number" required class="input-field" />
        <button @click="allocateToGoal" class="submit-button">Отложить</button>
      </div>
      <button @click="showAllocationForm(income)" class="allocate-button">Сколько отложить на цели</button>
      <button type="submit" class="submit-button">Добавить Доход</button>
    </form>

    <ul class="income-list">
      <li v-for="(income, index) in incomes" :key="index" class="income-item">
        {{ income.category }}: {{ income.amount }} UA
        <!-- <v-icon :icon="`mdiSvg:${mdiDelete}`"></v-icon> -->
        <button @click="removeIncomeHandler(income.id)" class="delete-button">Удалить</button>
      </li>
    </ul>

  </div>
</template>

<script>
import { computed, ref } from 'vue';
import { useBudgetStore } from '@/stores/budget';
// import { mdiDelete } from '@mdi/js'


export default {
    // data: () => ({
    //   mdiDelete
    // }),
  setup() {
    const budgetStore = useBudgetStore();
    const incomeCategory = ref(''); // Категория дохода
    const incomeAmount = ref(0);
    const showAllocation = ref(false); // Показ формы распределения
    const selectedGoal = ref(''); // Выбранная цель
    const allocationAmount = ref(0); // Сумма для откладывания
    const incomeToAllocate = ref(null); // Текущий доход, который распределяем

    // Форма для добавления дохода
    const addIncomeHandler = async () => {
      if (incomeCategory.value && incomeAmount.value > 0) {
        await budgetStore.addIncome({
          category: incomeCategory.value,
          amount: incomeAmount.value,
        });
        incomeCategory.value = ''; // Сброс категории
        incomeAmount.value = 0; // Сброс суммы
      }
    };

    // Форма для распределения денег на цель
    const showAllocationForm = (income) => {
      showAllocation.value = true;
      incomeToAllocate.value = income;
    };

    // Логика распределения денег на выбранную цель
    const allocateToGoal = async () => {
      if (selectedGoal.value && allocationAmount.value > 0 && allocationAmount.value <= incomeToAllocate.value.amount) {
        selectedGoal.value.amount += allocationAmount.value; // Добавляем к сумме цели
        await budgetStore.addGoal(selectedGoal.value); // Обновляем цель в базе данных
        incomeToAllocate.value.amount -= allocationAmount.value; // Вычитаем сумму из дохода
        await budgetStore.addIncome(incomeToAllocate.value); // Обновляем доход в базе данных
        resetAllocationForm(); // Сброс формы
      } else {
        console.error("Некорректные данные для распределения");
      }
    };

    // Сброс формы распределения
    const resetAllocationForm = () => {
      showAllocation.value = false;
      selectedGoal.value = '';
      allocationAmount.value = 0;
      incomeToAllocate.value = null;
    };

    const removeIncomeHandler = async (incomeId) => {
      try {
        await budgetStore.removeIncome(incomeId); // Удаление дохода
      } catch (error) {
        console.error("Ошибка при удалении дохода:", error);
      }
    };

    return {
      incomeCategory,
      incomeAmount,
      incomes: computed(() => budgetStore.incomes),
      goals: computed(() => budgetStore.goals),
      addIncomeHandler,
      removeIncomeHandler,
      showAllocation,
      showAllocationForm,
      selectedGoal,
      allocationAmount,
      allocateToGoal,
    };
  },
};
</script>



  
  <style>
  .tracker {
  max-width: 400px;
  margin: 0 auto;
  background-color: #f5f5f5;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

h2 {
  text-align: center;
  color: #333;
  margin-bottom: 20px;
}

.income-form
.allocation-form {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.input-field {
  padding: 10px;
  border-radius: 4px;
  border: 1px solid #ccc;
  font-size: 16px;
  outline: none;
  transition: border-color 0.3s ease;
}

.input-field:focus {
  border-color: #6200ea;
}

.submit-button, .allocate-button {
  background-color: #6200ea;
  color: #ffffff;
  padding: 10px;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.submit-button:hover,
.allocate-button:hover {
  background-color: #3700b3;
}

.income-list {
  list-style-type: none;
  padding: 0;
  margin-top: 20px;
}

.income-item {
  background-color: #ffffff;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  margin-bottom: 10px;
  font-size: 16px;
}
.delete-button {
  background-color: #ff5252;
  color: #ffffff;
  border: none;
  padding: 8px 12px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  margin-left: 15px;
  transition: background-color 0.3s ease;
}

.delete-button:hover {
  background-color: #e04040;
}

input,
button {
  margin: 10px 0;
  padding: 10px;
  width: 90%;
}

.calculate-button {
  margin-top: 20px;
  width: 100%;
}

p {
  margin-top: 15px;
  text-align: center;
  color: #333;
  font-size: 16px;
}
  </style>
  