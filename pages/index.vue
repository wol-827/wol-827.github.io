<template>
  <div id="contents">
    <GChart
      :settings="chartSettings"
      type="GeoChart"
      :data="chartData"
      :options="chartOptions"
      :legend="legend"
    />
  </div>
</template>

<script>
import { GChart } from 'vue-google-charts'

export default {
  components: {
    GChart
  },
  async asyncData({ app }) {
    const baseUrl = 'https://covid19-japan-web-api.now.sh/api/v1/prefectures'
    const getUrl = encodeURI(baseUrl)
    const response = await app.$axios.$get(getUrl)

    const chartData = [['Pref', '感染者数', '死者']]
    let maxCount = 0
    response.forEach((element) => {
      if (maxCount < element.cases) {
        maxCount = element.cases
      }

      chartData.push([element.name_ja, element.cases, element.deaths])
    })
    return {
      chartData,
      maxCount
    }
  },
  data() {
    return {
      chartSettings: {
        packages: ['geochart']
      },
      chartData: [],
      chartOptions: {
        colorAxis: {
          values: [],
          colors: ['#ffffff', '#ededec', '#FF0000']
        },
        region: 'JP',
        resolution: 'provinces',
        displayMode: 'regions'
      },
      maxCount: 0,
      legend: {
        textStyle: { color: 'blue', fontSize: 16 }
      }
    }
  },
  mounted() {
    this.chartOptions.colorAxis.values = [0, 1, this.maxCount]
  }
}
</script>
