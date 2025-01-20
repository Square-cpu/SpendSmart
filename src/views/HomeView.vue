<template>
    <nav>
        <button @click="showMensal = true; showAnnual = false">Monthly Expenses</button>
        <button @click="showMensal = false; showAnnual = true">Annual Expenses</button>
    </nav>

    <div class="container" v-if="showMensal">
        <select v-model="selectedMonth">
            <option v-for="month in months" :key="month" :value="month">{{ month }}</option>
        </select>
        <div class="table">
        <table>
            <thead>
            <tr>
                <th>Day</th>
                <th v-for="day in daysInSelectedMonth" :key="day">{{ day }}</th>
            </tr>
            </thead>
            <tbody>
            <tr>
                <td>Amount</td>
                <td v-for="day in daysInSelectedMonth" :key="day">
                <input v-model="filteredTableDataMensal.expenses[day].amount" type="number" @change="saveData(day, selectedMonth, $event)"/>
                </td>
            </tr>
            <tr>
                <td>
                    Description
                    (what did you buy?)
                </td>
                <td v-for="day in daysInSelectedMonth" :key="day">
                <textarea v-model="filteredTableDataMensal.expenses[day].description" type="text" @change="saveData(day, selectedMonth, $event)"></textarea>
                </td>
            </tr>
            </tbody>
        </table>
        </div>
    </div>

    <div class="container" v-if="showAnnual">
      <h1>2025</h1>
      <div class="table">
        <table>
          <thead>
            <tr>
              <th>Year</th>
              <th v-for="month in months" :key="month">{{ month }}</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>{{ selectedYear }}</td>
              <td v-for="month in months" :key="month">
                {{ monthlySums[month] }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <button @click="clearData" class="clear-button">Clear All Data</button>

    <div class="container">
        <h1 style="text-align: center; color: black; font-size: 50px;">Ask AI</h1>
        <p style="text-align: center; color: black; font-size: 20px;">
            Ask for AI advice about your spending habits.
            The AI will analyze your data and provide you with suggestions to improve your spending habits.
        </p>
        <h5 style="text-align: center; color: black; font-size: 20px;">
            Since this site is just a demo, we will provide you with a custom prompt based on the data you entered and you can copy and paste it into the
            <a href="https://chatgpt.com" target="_blank">chatgpt.com</a> website.
        </h5>
        <button @click="showModal = true" class="clear-button" style="background-color: #0f9926;">Ask AI</button>
    </div>

    <div v-if="showModal" class="modal-overlay">
        <div class="modal">
            <h2>AI Advice</h2>
            <h3>Selected month: {{ selectedMonth }}</h3>
            <pre class="modal-content">{{ aiAdvice }}</pre>
            <button @click="showModal = false" class="close-button">Close</button>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue';

const showMensal = ref(true);
const showAnnual = ref(false);

const selectedMonth = ref('January');

const createEmptyExpenses = (days) => {
  const expenses = {};
  for (let i = 1; i <= days; i++) {
    expenses[i] = { amount: 0, description: '' };
  }
  return expenses;
};

const tableDataMensal = ref([
  { month: 'January', expenses: createEmptyExpenses(31) },
  { month: 'February', expenses: createEmptyExpenses(28) },
  { month: 'March', expenses: createEmptyExpenses(31) },
  { month: 'April', expenses: createEmptyExpenses(30) },
  { month: 'May', expenses: createEmptyExpenses(31) },
  { month: 'June', expenses: createEmptyExpenses(30) },
  { month: 'July', expenses: createEmptyExpenses(31) },
  { month: 'August', expenses: createEmptyExpenses(31) },
  { month: 'September', expenses: createEmptyExpenses(30) },
  { month: 'October', expenses: createEmptyExpenses(31) },
  { month: 'November', expenses: createEmptyExpenses(30) },
  { month: 'December', expenses: createEmptyExpenses(31) },
]);

const months = ref([
  'January',
  'February',
  'March',
  'April',
  'May',
  'June',
  'July',
  'August',
  'September',
  'October',
  'November',
  'December',
]);

const aiAdvice = computed(() => {
  const monthData = tableDataMensal.value.find(data => data.month === selectedMonth.value);
  if (!monthData) return '';

  let adviceText = `Analyze my spending habits on ${selectedMonth.value} and provide advice to improve them. Here is my recent financial data:\n\n`;
  let remainingAmount = 0;

  Object.entries(monthData.expenses).forEach(([day, { amount, description }]) => {
    amount = Number(amount);
    if (amount > 0) {
      adviceText += `${amount.toFixed(2)} - ${description}\n`;
      remainingAmount += amount;
    }
  });

  adviceText += `\nTotal: ${remainingAmount.toFixed(2)}\n\nFocus on identifying areas where I can save more, optimize expenses, or adjust my budget.\n\nThank you!`;

  return adviceText;
});

const daysInSelectedMonth = computed(() => {
  const currentDate = new Date();
  const currentYear = currentDate.getFullYear();
  const selectedMonthIndex = months.value.indexOf(selectedMonth.value);
  const daysInMonth = new Date(currentYear, selectedMonthIndex + 1, 0).getDate();

  return Array.from({ length: daysInMonth }, (_, i) => i + 1);
});

const selectedYear = ref(new Date().getFullYear());

const filteredTableDataMensal = computed(() => {
  return tableDataMensal.value.find(data => data.month === selectedMonth.value) || { expenses: {} };
});

const saveData = (day, month, event) => {
  const value = event.target.value;
  const monthData = tableDataMensal.value.find(data => data.month === month);
  if (monthData) {
    if (event.target.type === 'number') {
      monthData.expenses[day].amount = Number(value);
    } else if (event.target.type === 'text') {
      monthData.expenses[day].description = value;
    }
  }
};

const monthlySums = computed(() => {
  const sums = {};
  months.value.forEach(month => {
    const monthData = tableDataMensal.value.find(data => data.month === month);
    if (monthData) {
      sums[month] = Object.values(monthData.expenses).reduce((acc, { amount }) => acc + Number(amount), 0);
    } else {
      sums[month] = 0;
    }
  });
  return sums;
});

// Watch for changes in tableDataMensal and store them in local storage
watch(tableDataMensal, (newValue) => {
  localStorage.setItem('tableDataMensal', JSON.stringify(newValue));
}, { deep: true });

// Load data from local storage on mounted
onMounted(() => {
  const storedData = localStorage.getItem('tableDataMensal');
  if (storedData) {
    tableDataMensal.value = JSON.parse(storedData);
  } else {
    tableDataMensal.value.forEach(month => {
        localStorage.setItem('tableDataMensal', JSON.stringify(tableDataMensal.value));        
    });
  }
});

const clearData = () => {
  tableDataMensal.value.forEach(month => {
    Object.keys(month.expenses).forEach(day => {
      month.expenses[day].amount = 0;
      month.expenses[day].description = '';
    });
  });
  localStorage.setItem('tableDataMensal', JSON.stringify(tableDataMensal.value));
};

const showModal = ref(false);
</script>

<style scoped lang="scss">
body {
  background-color: #f4f4f9;
  margin: 0;
  padding: 0;
}

nav {
  width: 500px;
  margin: 20px auto;
  margin-top: 100px;
  display: flex;
  justify-content: center;
  gap: 20px;
  background-color: #004b9b;
  padding: 1em;
  border-bottom: 1px solid #ccc;
  border-radius: 20px;

  button {
    background-color: #fff;
    border: none;
    padding: 0.5em 1em;
    cursor: pointer;
    border-radius: 5px;
    font-size: 1em;
    transition: background-color 0.3s ease;
  }

  button:hover {
    background-color: #0466cf;
    color: #fff;
  }
}

.container {
  background-color: #fff;
  border: #ccc solid 1px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin: 20px auto;
  width: 80%;
  padding: 20px;
  border-radius: 10px;
}

select {
  margin-top: 10px;
  margin-bottom: 10px;
  width: 50%;
  padding: 0.5em;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 1em;
}

.table {
  width: 100%;
  overflow-x: auto;
}

table {
  border-collapse: collapse;
  width: 100%;
  margin-top: 20px;
}

th, td {
  border: 1px solid #ddd;
  padding: 1em;
  width: 30px;
  text-align: center;
}

th {
  background-color: #007BFF;
  color: #fff;
}

input[type="number"] {
  width: 80px;
  padding: 0.5em;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 1em;
}

textarea {
  padding: 0.5em;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 1em;
}

button[type="button"], button[type="submit"] {
  background-color: #28a745;
  color: #fff;
  padding: 0.75em 1.5em;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 1em;
  transition: background-color 0.3s ease;
  margin-top: 20px;
}

button[type="button"]:hover, button[type="submit"]:hover {
  background-color: #218838;
}

.clear-button {
  background-color: #dc3545;
  color: #fff;
  padding: 0.75em 1.5em;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 1em;
  transition: background-color 0.3s ease;
  margin-top: 20px;

  margin: 20px auto;
  display: block;
}

h1 {
  font-size: 2em;
  margin-bottom: 20px;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1500;
}

.modal {
  background-color: #fff;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  width: 80%;
  max-width: 1000px;
  max-height: 80%;
  overflow-y: auto;
  text-align: center;
}

.modal-content {
  max-height: 60vh;
  overflow-y: auto;
  white-space: pre-wrap;
  text-align: left;
}

.close-button {
  background-color: #dc3545;
  color: #fff;
  padding: 0.5em 1em;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 1em;
  transition: background-color 0.3s ease;
  margin-top: 20px;
}

.close-button:hover {
  background-color: #c82333;
}
</style>