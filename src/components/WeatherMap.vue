<script setup>
import { ref } from 'vue';
import axios from 'axios';
import {Button as AButton, FormItem as AFormItem, Form as AForm, Input as AInput,
  Table as ATable, DatePicker as ADatePicker} from "ant-design-vue";
import moment from 'moment';
const cityName = ref('');
const startDate = ref('');
const endDate = ref('');
const result = ref(null);
const polDataSource = ref([]);
const showTable = ref(false);
const columns = ref([
  { title: 'Date', dataIndex: 'date', key: 'date' },
  { title: 'CO', dataIndex: 'co', key: 'co' },
  { title: 'SO2', dataIndex: 'so2', key: 'so2' },
  { title: 'O3', dataIndex: 'o3', key: 'o3' },
]);
const fetchResults = async () => {
  const url = `http://localhost:8080/api/pollutions`;
  const formattedStartDate = moment(startDate.value.toString()).format('YYYY-MM-DD');
  const formattedEndDate = moment(endDate.value.toString()).format('YYYY-MM-DD');
  const params = {
    cityName: cityName.value,
    start: formattedStartDate,
    end: formattedEndDate,
  };

  try {
    console.log('City name received: ', cityName.value);
    console.log('Start Date: ', formattedStartDate);
    console.log('End Date: ', formattedEndDate);
    const response = await axios.get(url, { auth: {
        username: 'user1',
        password: ''
      }, params });
    result.value = response.data;
    polDataSource.value = result.value.results.map(pollution => ({
      date: pollution.date,
      co: pollution.categories.co,
      so2: pollution.categories.so2,
      o3: pollution.categories.o3,
    }));
    showTable.value = true;
  } catch (error) {
    console.error('Error fetching data:', error);
    result.value = null;
  }
};
const getColorClass = (value) => {
  switch (value.toLowerCase()) {
    case 'good':
      return 'text-green';
    case 'satisfactory':
      return 'text-lightgreen';
    case 'moderate':
      return 'text-yellow';
    case 'poor':
      return 'text-orange';
    case 'severe':
      return 'text-red';
    case 'hazardous':
      return 'text-maroon';
    default:
      return 'text-black'; // Default color if the value doesn't match any criteria
  };
};
</script>

<template>
  <div>
    <a-form>
      <a-form-item label="City Name">
        <a-input v-model:value="cityName" />
      </a-form-item>
      <a-form-item label="Start Date">
        <a-date-picker v-model:value="startDate" style="width: 100%" format="YYYY-MM-DD" />
      </a-form-item>
      <a-form-item label="End Date">
        <a-date-picker v-model:value="endDate" style="width: 100%" format="YYYY-MM-DD" />
      </a-form-item>
      <a-form-item>
        <a-button type="primary" @click="fetchResults">Get Results</a-button>
      </a-form-item>
    </a-form>
    <div class="table-container" v-if="showTable">
      <a-table
          :columns="columns"
          :dataSource="polDataSource"
          :pagination="{ pageSize: 5 }"
      >
        <template #bodyCell="{ column, text }">
          <template v-if="column.key === 'co'">
            <span :class="getColorClass(text)"> {{text}} </span>
          </template>
          <template v-else-if="column.key === 'so2'">
            <span :class="getColorClass(text)"> {{text}} </span>
          </template>
          <template v-else-if="column.key === 'o3'">
            <span :class="getColorClass(text)"> {{text}} </span>
          </template>
        </template>
      </a-table>
    </div>

  </div>
</template>

<style scoped>
.table-container {
  overflow: auto;
  max-height: 400px;
  margin-top: 20px;
}
.text-green {
  color: #00FC2E;
}

.text-lightgreen {
  color: #229236;
}

.text-yellow {
  color: #D8DE1F;
}

.text-orange {
  color: #F9AD00;
}

.text-red {
  color: red;
}

.text-maroon {
  color: #7C3012;
}

.text-black {
  color: black;
}
</style>