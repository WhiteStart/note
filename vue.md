- echarts5与vue3似乎不兼容，出现了echarts undefined
- 这种写法使用 "echarts": "^4.9.0" 解决

```js
export default {
  mounted() {
    this.initChart()
  },
  methods: {
    initChart() {
      const chartElement = this.$refs.chart
      const myChart = this.$echarts.init(chartElement)

      const option = {
        // ECharts配置选项和数据
        title: {
          text: 'ECharts 入门示例',
        },
        tooltip: {},
        legend: {
          data: ['销量'],
        },
        xAxis: {
          data: ['衬衫', '羊毛衫', '雪纺衫', '裤子', '高跟鞋', '袜子'],
        },
        yAxis: {},
        series: [
          {
            name: '销量',
            type: 'bar',
            data: [5, 20, 36, 10, 10, 20],
          },
        ],
      }

      myChart.setOption(option)
    },
  },
}
</script>
```

- 使用了setup()解决了undefined问题

```vue
<template>
  <div ref="chartContainer" style="width: 100%; height: 400px"></div>
</template>

<script>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as echarts from 'echarts'

export default {
  setup() {
    const chartContainer = ref(null)
    let chart = null

    onMounted(() => {
      chart = echarts.init(chartContainer.value)

      // 示例数据
      const data = [
        { name: 'Category 1', value: 100 },
        { name: 'Category 2', value: 200 },
        { name: 'Category 3', value: 300 },
        { name: 'Category 4', value: 400 },
        { name: 'Category 5', value: 500 },
      ]

      // ECharts 配置项
      const options = {
        xAxis: {
          type: 'category',
          data: data.map((item) => item.name),
        },
        yAxis: {
          type: 'value',
        },
        series: [
          {
            data: data.map((item) => item.value),
            type: 'bar',
          },
        ],
      }

      // 使用示例数据和配置项渲染图表
      chart.setOption(options)
    })

    onBeforeUnmount(() => {
      if (chart) {
        chart.dispose()
        chart = null
      }
    })

    return {
      chartContainer,
    }
  },
}
</script>
```







