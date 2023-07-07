<template>
  <q-page class="q-pa-md">
    <div class="row items-center q-gutter-lg justify-center">
      <div class="col-9 col-md-3 ">
        <setting-list @updateChart="updateChart"/>
      </div>

      <div class="col-10 col-md-6 ">
        <div id="myChart" :style="{height: height*0.75+'px'}"></div>
      </div>
    </div>
  </q-page>
</template>

<script>
import {defineComponent, onMounted, onUnmounted, ref} from 'vue'
import {echarts} from "src/echarts";
import SettingList from "components/SettingList.vue";

function initChart() {
  const chartDom = document.getElementById('myChart');
  return echarts.init(chartDom);
}

export default defineComponent({
  name: 'IndexPage',
  components: {SettingList},
  setup() {
    const optionData = ref({})

    const height = ref(window.innerHeight ||
      document.documentElement.clientHeight ||
      document.body.clientHeight)

    let myChart = null

    function updateChart(data) {
      if (myChart) {
        window.onload = myChart.setOption(data)
      } else {
        optionData.value = data
      }
    }

    onMounted( () => {
      myChart = initChart()
      myChart.setOption(optionData.value)
      myChart.resize();
      window.onresize = () => {
        height.value = window.innerHeight ||
          document.documentElement.clientHeight ||
          document.body.clientHeight;
        myChart.resize();
      };
    })

    onUnmounted(() => {
      myChart.dispose()
    })

    return {
      height,

      updateChart
    }
  }
})
</script>
