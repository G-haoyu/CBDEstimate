<template>
  <q-list bordered padding>
    <q-item-label header>参数设置</q-item-label>
    <q-item>
      <q-table
        dense
        flat
        style="width: 100%"
        :rows="rowData"
        :columns="columns"
        row-key="name"
        hide-bottom
      >
        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td key="id" :props="props">{{ props.row.id }}</q-td>
            <q-td key="depth" :props="props">
              {{ props.row.depth }}
              <q-popup-edit v-model="props.row.depth" buttons label-set="确认" label-cancel="取消" v-slot="scope">
                <q-input type="number" v-model="scope.value" dense autofocus @keypress.enter="scope.set" />
              </q-popup-edit>
            </q-td>
            <q-td key="wetWeight" :props="props">
              <div class="text-pre-wrap">{{ props.row.wetWeight }}</div>
              <q-popup-edit v-model="props.row.wetWeight" buttons label-set="确认" label-cancel="取消" v-slot="scope">
                <q-input type="number" v-model="scope.value" dense autofocus @keypress.enter="scope.set"/>
              </q-popup-edit>
            </q-td>
            <q-td key="dryWeight" :props="props">
              {{ props.row.dryWeight }}
              <q-popup-edit v-model="props.row.dryWeight" buttons label-set="确认" label-cancel="取消"  v-slot="scope">
                <q-input type="number" v-model="scope.value" dense autofocus @keypress.enter="scope.set"/>
              </q-popup-edit>
            </q-td>
          </q-tr>
        </template>
      </q-table>
    </q-item>

    <q-separator spaced />
    <q-item-label header>计算结果</q-item-label>

    <q-item>
      <q-item-section>
        <q-item-label>含水量</q-item-label>
        <q-item-label caption>
          试验次数1：<span class="text-bold">{{(waterContent1*100)+'%'}}</span>
        </q-item-label>
        <q-item-label caption>
          试验次数2：<span class="text-bold">{{(waterContent2*100)+'%'}}</span>
        </q-item-label>
        <q-item-label caption>
          试验次数3：<span class="text-bold">{{(waterContent3*100)+'%'}}</span>
        </q-item-label>
      </q-item-section>
    </q-item>

    <q-item>
      <q-item-section>
        <q-item-label>2mm对应含水量差距（点击修改）：
          <span v-if="gap2mm < 0.02" class="text-bold text-green">{{(gap2mm*100)+'%'}}</span>
          <span v-else class="text-bold text-red">{{(gap2mm*100)+'%'}}</span>

          <q-popup-edit v-model="gap2mm" auto-save v-slot="scope" @save="changeGap2mm">
            <q-input v-model="scope.value" dense autofocus counter @keyup.enter="scope.set" />
          </q-popup-edit>
        </q-item-label>
      </q-item-section>
    </q-item>

    <q-separator spaced />
    <q-item-label header>试验结果</q-item-label>

    <q-item>
      <q-item-section>
        <q-item-label>17mm液限：
          <span class="text-bold text-green">{{waterContent17mmR*100}}</span>
        </q-item-label>
        <q-item-label>10mm液限：
          <span class="text-bold text-green">{{waterContent10mmR*100}}</span>
        </q-item-label>
        <q-item-label>2mm塑限：
          <span class="text-bold text-green">{{waterContent2mmR*100}}</span>
        </q-item-label>
      </q-item-section>
    </q-item>

    <q-item>
      <q-item-section>
        <q-item-label>17mm塑性指数：
          <span class="text-bold text-green">{{gap17mmR * 100}}</span>
        </q-item-label>
        <q-item-label>10mm塑性指数：
          <span class="text-bold text-green">{{gap10mmR * 100}}</span>
<!--          <q-popup-edit v-model="gap10mmR" auto-save v-slot="scope" @save="changeGap10mmR">-->
<!--            <q-input v-model="scope.value" dense autofocus counter @keyup.enter="scope.set" />-->
<!--          </q-popup-edit>-->
        </q-item-label>
      </q-item-section>
    </q-item>

    <q-separator spaced />
    <q-item-label header>Other settings</q-item-label>

  </q-list>
</template>

<script>
import {computed, onMounted, ref, watch} from "vue";

const columns = [
  { name: 'id', label: '试验次数', align: 'center', field: 'id' },
  { name: 'depth', align: 'left', label: '锥入深度 (mm)', field: 'depth' },
  { name: 'wetWeight', align: 'left', label: '湿土质量 (g)', field: 'wetWeight' },
  { name: 'dryWeight', align: 'left', label: '干土质量 (g)', field: 'dryWeight' }
]

const rows = [
  {
    id: '1',
    depth: 16.5,
    wetWeight: 12.45,
    dryWeight: 8.74
  },
  {
    id: '2',
    depth: 9.9,
    wetWeight: 11.02,
    dryWeight: 7.98
  },
  {
    id: '3',
    depth: 4.2,
    wetWeight: 13.55,
    dryWeight: 10.37
  },
]

function getY(x1, y1, x2, y2, x) {
  if(x2 - x1 === 0) {
    return null
  }
  const k = (y2 - y1) / (x2 - x1)
  return k * (x - x1) + y1
}

function getX(x1, y1, x2, y2, y) {
  if(y2 - y1 === 0) {
    return null
  }
  const kd = (x2 - x1) / (y2 - y1)
  return kd * (y - y1) + x1
}

function getHByW(w) {
  return  w / (0.524 * w - 7.606)
}

function getHByW2(w) {
  return  29.6 - 1.22 * w + 0.017 * w * w - 0.0000744 * w * w * w
}



export default {
  name: "SettingList",
  setup(props, ctx) {
    const rowData = ref(rows)

    const gap2mm = ref(0)
    const waterContent2mmR = ref(0)
    const waterContent10mmR = ref(0)
    const waterContent17mmR = ref(0)

    const waterContent1 = computed(() => {
      const data = rowData.value
      return (data[0].wetWeight - data[0].dryWeight) / data[0].dryWeight
    })
    const waterContent2 = computed(() => {
      const data = rowData.value
      return (data[1].wetWeight - data[1].dryWeight) / data[1].dryWeight
    })
    const waterContent3 = computed({
      get(){
        const data = rowData.value
        return (data[2].wetWeight - data[2].dryWeight) / data[2].dryWeight
      },
      set(value){
        const data = rowData.value
        data[2].dryWeight = data[2].wetWeight / (value+1)
      }
    })

    const gap10mmR = computed({
      get() {
        return waterContent10mmR.value - waterContent2mmR.value
      },
      set() {}
    })

    const gap17mmR = computed(() => {
      return waterContent17mmR.value - waterContent2mmR.value
    })

    function changeGap2mm(nowGap2mm) {
      console.log(nowGap2mm, "changeGap")
      if (nowGap2mm !== gap2mm.value) {
        const data = rowData.value
        const dp1 = data[0].depth
        const wc1 = waterContent1.value
        const dp2 = data[1].depth
        const wc2 = waterContent2.value
        const dp3 = data[2].depth
        const lg_dp1 = Math.log10(dp1)
        const lg_wc1 = Math.log10(wc1)
        const lg_dp2 = Math.log10(dp2)
        const lg_wc2 = Math.log10(wc2)
        const lg_dp3 = Math.log10(dp3)

        const lg_wc2mm1 = getX(lg_wc1, lg_dp1, lg_wc2, lg_dp2, Math.log10(2))
        const true_wc2mm1 = Math.pow(10, lg_wc2mm1)
        const now_true_wc2mm2 = true_wc2mm1 - nowGap2mm

        const now_lg_wc3 = getX(lg_wc1, lg_dp1, Math.log10(now_true_wc2mm2), Math.log10(2), lg_dp3)
        // change wc3
        const now_true_wc3 = Math.pow(10, now_lg_wc3)
        waterContent3.value = now_true_wc3
        console.log(now_true_wc3)
      }
    }

    function changeGap10mmR(nowGap10mmR) {
      console.log(nowGap10mmR, "change Gap10mmR")
      if (nowGap10mmR !== gap10mmR.value) {
        waterContent10mmR.value - waterContent2mmR.value
        waterContent2mmR.value = Math.pow(10, getX(lg_wc1, lg_dp1, Math.log10((true_wp2mm1 + true_wp2mm2) / 2), lg_dp2mmR, Math.log10(2)))
        waterContent10mmR.value = Math.pow(10, getX(lg_wc1, lg_dp1, Math.log10((true_wp2mm1 + true_wp2mm2) / 2), lg_dp2mmR, Math.log10(10)))
      }
    }

    function generateOp(dp1, wc1, dp2, wc2, dp3, wc3) {
      const lg_dp1 = Math.log10(dp1)
      const lg_wc1 = Math.log10(wc1)
      const lg_dp2 = Math.log10(dp2)
      const lg_wc2 = Math.log10(wc2)
      const lg_dp3 = Math.log10(dp3)
      const lg_wc3 = Math.log10(wc3)
      const draw_dp1a = dp1 * 1.2
      const draw_wc1a = getX(wc1, dp1, wc2, dp2, draw_dp1a)
      const draw_dp1b = 0
      const draw_wc1b = getX(wc1, dp1, wc2, dp2, draw_dp1b)
      const draw_dp2a = draw_dp1a
      const draw_wc2a = getX(wc1, dp1, wc3, dp3, draw_dp2a)
      const draw_dp2b = 0
      const draw_wc2b = getX(wc1, dp1, wc3, dp3, draw_dp2b)
      // 取 dp = 2mm
      const draw_dp2mm = 2
      const lg_dp2mm = Math.log10(draw_dp2mm)
      const lg_wc2mm1 = getX(lg_wc1, lg_dp1, lg_wc2, lg_dp2, lg_dp2mm)
      const lg_wc2mm2 = getX(lg_wc1, lg_dp1, lg_wc3, lg_dp3, lg_dp2mm)
      // 2mm true w
      const true_wc2mm1 = Math.pow(10, lg_wc2mm1)
      const true_wc2mm2 = Math.pow(10, lg_wc2mm2)
      const true_wc2mmR = (true_wc2mm1 + true_wc2mm2) / 2
      // 2mm含水量之差
      gap2mm.value = Math.abs(true_wc2mm1 - true_wc2mm2)

      // 2mm塑限时的入土深度 对数
      const lg_dp2mmR = Math.log10(getHByW(true_wc2mmR*100))

      // wp1 wp2
      const true_wp2mm1 = Math.pow(10, getX(lg_wc1, lg_dp1, lg_wc2, lg_dp2, lg_dp2mmR))
      const true_wp2mm2 = Math.pow(10, getX(lg_wc1, lg_dp1, lg_wc3, lg_dp3, lg_dp2mmR))

      // 存
      waterContent2mmR.value = Math.pow(10, getX(lg_wc1, lg_dp1, Math.log10((true_wp2mm1 + true_wp2mm2) / 2), lg_dp2mmR, Math.log10(2)))
      waterContent10mmR.value = Math.pow(10, getX(lg_wc1, lg_dp1, Math.log10((true_wp2mm1 + true_wp2mm2) / 2), lg_dp2mmR, Math.log10(10)))
      waterContent17mmR.value = Math.pow(10, getX(lg_wc1, lg_dp1, Math.log10((true_wp2mm1 + true_wp2mm2) / 2), lg_dp2mmR, Math.log10(17)))
      console.log(waterContent2mmR.value, "2mm true")
      console.log(waterContent10mmR.value, "10mm true")
      console.log(waterContent17mmR.value, "17mm true")
      console.log(waterContent17mmR.value - waterContent2mmR.value, "17 diff")
      console.log(waterContent10mmR.value - waterContent2mmR.value, "10 diff")
      // // 正确线画点
      // const draw_dp3a = draw_dp1a
      // const draw_wc3a = getX(waterContent17mmR.value, 17, waterContent2mmR.value, getHByW(true_wc2mmR*100), draw_dp3a)
      // const draw_dp3b = 0
      // const draw_wc3b = getX(waterContent17mmR.value, 17, waterContent2mmR.value, getHByW(true_wc2mmR*100), draw_dp3b)
      let op = {
        xAxis: {},
        yAxis: {},
        series: [
          {
            data: [
              [wc1, dp1],
              [wc2, dp2],
              [wc3, dp3],
            ],
            type: 'scatter',
            symbolSize: 8
          },
          {
            data: [
              [draw_wc1a, draw_dp1a],
              [draw_wc1b, draw_dp1b]
            ],
            type: 'line'
          },
          {
            data: [
              [draw_wc2a, draw_dp2a],
              [draw_wc2b, draw_dp2b]
            ],
            type: 'line'
          },
          {
            type: 'line',
            markLine: {
              symbol: ['none', 'none'],
              itemStyle: {
                normal: { lineStyle: { type: 'solid', color:'blue'}
                  ,label: { show: false, position:'left' } }
              },
              data: [
                {yAxis: 2},
                {yAxis: 10},
                {yAxis: 17}
              ]
            }
          }
        ]
      };

      return op
    }

    watch(rowData, (newData, oldData) => {
      const op = generateOp(newData[0].depth, waterContent1.value,
        newData[1].depth, waterContent2.value,
        newData[2].depth, waterContent3.value
      )
      ctx.emit("updateChart", op)
    },{ deep: true })

    onMounted(() => {
      const data = rowData.value
      const op = generateOp(data[0].depth, waterContent1.value,
        data[1].depth, waterContent2.value,
        data[2].depth, waterContent3.value
      )
      ctx.emit("updateChart", op)
    })

    return {
      columns,
      rowData,
      gap2mm,
      waterContent1,
      waterContent2,
      waterContent3,

      waterContent2mmR,
      waterContent10mmR,
      waterContent17mmR,

      gap10mmR,
      gap17mmR,

      changeGap2mm,
      changeGap10mmR,

    }
  }
}
</script>

<style scoped>

</style>
