<script setup>
    import { ref, shallowRef, computed, watch, nextTick } from 'vue';
    import Chart from 'chart.js/auto';

    const weights = ref([])
    const weightChartEl = ref(null)
    const weightChart = shallowRef(null)
    const weightInput = ref(60.0)

    const currentWeight = computed(() => {
        return weights.value.sort((a, b) => b.date - a.date)[0] || { weight:0 }
    })

    const addWeight = () => {
        weights.value.push ({
        weight: weightInput.value,
        date: new Date().getTime()
        })
    }

    watch(weights, newWeights => {
        const ws = [...newWeights ]

        if(weightChart.value){
            weightChart.value.data.labels = ws  
                .sort((a,b) => a.date - b.date)
                .map(w => new Date(w.date).toLocaleDateString())
                .slice(-7)

            weightChart.value.data.datasets[0].data = ws  
                .sort((a,b) => a.date - b.date)
                .map(w => w.weight)
                .slice(-7)

            weightChart.value.update()

            return
        }

        nextTick(() => {
            weightChart.value = new Chart(weightChartEl.value.getContext('2d'), {
                type: 'line',
                data: {
                labels: ws
                    .sort((a,b) => a.date - b.date)
                    .map(w => new Date(w.date).toLocaleDateString()),
                datasets: [
                    {
                    label: 'WEIGHT',
                    data: ws
                        .sort((a,b) => a.date - b.date)
                        .map(w => w.weight),
                    backgroundColor: '#bbd0ff',
                    borderColor: '#2a9d8f',
                    borderWidth: 2,
                    fill: true,
                    }
                ]
                },
                options: {
                responsive: true,
                maintainAspectRatio: false,
                }
            })
        })
        
    },{ deep: true })


</script>

<template>
  <div class="mx-[20rem]">
    <h1 class="text-[2rem] text-center mb-8 font-bold">Weight Tracker</h1>
    <div id="current" class="flex flex-col justify-center items-center w-[200px] h-[200px] text-center bg-white rounded-[999px] mx-auto mt-0 mb-8">
      <span class="block text-[2rem] font-medium mb-2">{{ currentWeight.weight }}</span>
      <small class="text-[#888] italic">Current Weight (kg)</small>
    </div>

    <form @submit.prevent="addWeight" class="flex mb-8 rounded-lg overflow-hidden focus-within: hover:border-[#2a9d8f] hover:border-2 ">
      <input type="number" step="0.1" v-model="weightInput" class="appearance-none outline-none bg-white flex-1 text-xl py-4 px-6"/>
      <input type="submit" value="Add weight" id="submit" class="appearance-none outline-none  cursor-pointer bg-[#2a9d8f] text-white text-xl font-medium border-transparent border-none py-2 px-4 hover:bg-white hover:text-[#2a9d8f] hover:border-l-[10px] hover:border-l-[#2a9d8f]"/>
    </form>
      
    <div v-if="weights && weights.length > 0">
      <h2 class="mb-4 text-[#888] font-normal">Last 7 days</h2>
      <div class="w-[100%] bg-white p-4 rounded-lg shadow-md mb-8">
        <canvas ref="weightChartEl"></canvas>
      </div>
      <div class="weight-history">
        <h2 class="mb-4 text-[#888] font-normal">Weight History</h2>
        <ul class="list-none p-0 m-0">
          <li v-for="weight in weights" class="flex justify-between items-center p-2 cursor-pointer">
            <span class="block text-xl font-medium mr-4">
              {{  weight.weight }} kg
            </span>
            <small class="text-[#888] italic"> 
              {{ new Date(weight.date).toLocaleDateString() }}
            </small>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<style>
    #current {
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
	    border: 5px solid #2a9d8f;
    }

    form {
        border: 1px solid #AAA;
        transition: 200ms linear;
    }

    #submit {
        transition: 200ms linear;
	    border-left: 3px solid transparent;
    }

    .weight-history ul li:nth-child(even) {
	background-color: #dfdfdf;
    }

    .weight-history ul li:hover {
        background-color: #f8f8f8;
    }
    
    .weight-history ul li:last-of-type {
        border-bottom: none;
    }
</style>
