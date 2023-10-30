
<template>
    <div>
        <div class="date-and-measure-container">
            <div class="date-panel">
                <form class="date-form">
                    <label>Date Range:</label>
                    <div class="date-button-panel">
                        <button @click.prevent="updateDateRange('week')" :class="{ active: setDateRange === 'week' }">
                            Week
                        </button>

                        <button @click.prevent="updateDateRange('month')" :class="{ active: setDateRange === 'month' }">
                            Month
                        </button>

                        <button @click.prevent="updateDateRange('year')" :class="{ active: setDateRange === 'year' }">
                            Year
                        </button>

                        <button @click.prevent="updateDateRange('all')" :class="{ active: setDateRange === 'all' }">
                            All
                        </button>

                        <button @click.prevent="toggleCustomDateInput" :class="{ active: setDateRange === 'custom' }">
                            Custom
                        </button>
                    </div>

                    <div v-if="showCustomDateInput">
                        <label for="start-date">Start Date:</label>
                        <input type="date" v-model="startDate" id="start-date" required />

                        <label for="end-date">End Date:</label>
                        <input type="date" v-model="endDate" id="end-date" required />

                        <button @click.prevent="handleCustomDateSubmit">Submit</button>

                    </div>
                </form>

            </div>

            <div class="measure-panel">
                <form class="measure-form">
                    <label>Metric:</label>
                    <div class="measure-button-panel">
                        <button @click.prevent="updateMetric('volatility')" :class="{ active: setMetric === 'volatility' }">
                            Volatility
                        </button>

                        <button @click.prevent="updateMetric('open')" :class="{ active: setMetric === 'open' }">
                            Open
                        </button>

                        <button @click.prevent="updateMetric('close')" :class="{ active: setMetric === 'close' }">
                            Close
                        </button>

                        <button @click.prevent="updateMetric('volume')" :class="{ active: setMetric === 'volume' }">
                            Volume
                        </button>

                        <button @click.prevent="updateMetric('market-cap')" :class="{ active: setMetric === 'market-cap' }">
                            Market Cap
                        </button>
                    </div>
                </form>

            </div>
        </div>


        <div class="chart-container">
            <LineChart :data="formattedChartData" />
        </div>
    </div>
</template>
  
<script setup>
import { ref, onMounted, computed, watch } from 'vue';
import axios from 'axios';
import LineChart from '../components/LineChart.vue';
import Sidebar from '../components/SideBar.vue';

const startDate = ref('2016-01-01');
const endDate = ref('2023-01-01');
const currentDate = ref(new Date("2021-07-06"));
const setDateRange = ref('all');
let setMetric = ref('volatility');
const showCustomDateInput = ref(false);

function handleCustomDateSubmit() {
    if (!startDate.value || !endDate.value) {
        console.error('Dates must be provided before fetching data.');
        return;
    }
    fetchData();
}

watch(setDateRange, (newValue) => {
    const tempDate = new Date(currentDate.value);
    switch (newValue) {
        case 'week':
            startDate.value = formatDate(new Date(tempDate.setDate(tempDate.getDate() - 7)));
            endDate.value = formatDate(currentDate.value);
            break;
        case 'month':
            startDate.value = formatDate(new Date(tempDate.setMonth(tempDate.getMonth() - 1)));
            endDate.value = formatDate(currentDate.value);
            break;
        case 'year':
            startDate.value = formatDate(new Date(tempDate.setFullYear(tempDate.getFullYear() - 1)));
            endDate.value = formatDate(currentDate.value);
            break;
        case 'all':
            startDate.value = "2013-01-01";
            endDate.value = "2023-01-01";
            break;
    }
    fetchData();
});


function toggleCustomDateInput() {
    showCustomDateInput.value = !showCustomDateInput.value;
    if (showCustomDateInput.value) {
        setDateRange.value = 'custom';
    }
}

function updateDateRange(range) {
    setDateRange.value = range;
    if (range !== 'custom') {
        showCustomDateInput.value = false;
    }
}

function formatDate(date) {
    return date.toISOString().split('T')[0];
}

function updateMetric(range) {
    setMetric.value = range;
    fetchData();
}

const lineChartData = ref([]);

const fetchData = async () => {

    const metricMap = {
        'volatility': 'Volatility',
        'open': 'Open',
        'close': 'Close',
        'volume': 'Volume',
        'market-cap': 'Marketcap'
    };

    try {
        const response = await axios.post('http://localhost:5000/api/crypto', {
            start_date: startDate.value,
            end_date: endDate.value,
        });

        if (response.data) {
            const formatData = (data) => {
                const metricValue = metricMap[setMetric.value];
                return Object.values(data).map(item => ({
                    x: item.Date,
                    y: item[metricValue]
                }));
            };

            lineChartData.value = [
                {
                    label: 'Cardano',
                    data: formatData(JSON.parse(response.data['coin_Cardano'] || '{}')),
                },
                {
                    label: 'Bitcoin',
                    data: formatData(JSON.parse(response.data['coin_Bitcoin'] || '{}')),
                },
                {
                    label: 'Ethereum',
                    data: formatData(JSON.parse(response.data['coin_Ethereum'] || '{}')),
                }
            ];
        } else {
            console.error('No data received');
        }
    } catch (error) {
        console.error('Error fetching data:', error);
    }
};

onMounted(fetchData);

const formattedChartData = computed(() => lineChartData.value);
</script>

<style scoped>
.date-and-measure-container {
    display: flex;
    justify-content: space-between;
    width: 100%;
    flex-wrap: wrap;
}

.date-panel,
.measure-panel {
    flex: 1;
    margin: 0 10px;
    box-sizing: border-box;
    /* This ensures that padding and borders don't add to the width */
}

.date-form,
.measure-form {
    margin: 20px 0;
}

.chart-container {
    background: linear-gradient(135deg, #2b3160 0%, #181616 100%);
    padding: 10px;
    width: 100%;
}

.date-button-panel button.active,
.measure-button-panel button.active {
    background-color: #4caf50;
    color: white;
}
</style>
  