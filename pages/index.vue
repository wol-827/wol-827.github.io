<template>
  <div>
    <header class="navbar">
      <div class="navbar-brand">
        <a class="navbar-item" href="">
          <img src="~/assets/icon_virus.png" />
        </a>
      </div>
      <div class="navbar-menu">
        <div class="navbar-start">
          <a
            class="navbar-item is-tab"
            :class="{ 'is-active': isActive === 'caseCount' }"
            @click="caseCount"
          >
            感染者数
          </a>
          <a
            class="navbar-item is-tab"
            :class="{ 'is-active': isActive === 'casePerf' }"
            @click="casePerf"
          >
            人口10万対
          </a>
          <a
            class="navbar-item is-tab"
            :class="{ 'is-active': isActive === 'deathPerf' }"
            @click="isActive = 'deathPerf'"
          >
            死亡率
          </a>
        </div>
      </div>
    </header>
    <main class="container">
      <div ref="chart" class="columns">
        <div class="column is-one-third">
          <GChart
            :settings="{ packages: ['bar'] }"
            type="BarChart"
            :data="chartData"
            :options="barChartOptions"
            :create-chart="(el, google) => new google.charts.Bar(el)"
          />
        </div>
        <div class="column is-marginless">
          <GChart
            :settings="{ packages: ['geochart'] }"
            type="GeoChart"
            :data="chartData"
            :options="geoChartOptions"
            :legend="geoLegend"
            :resize-debounce="500"
          />
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import { GChart } from 'vue-google-charts';

export default {
  components: {
    GChart,
  },
  async asyncData({ app }) {
    const baseUrl = 'https://covid19-japan-web-api.now.sh/api/v1/prefectures';
    const getUrl = encodeURI(baseUrl);
    const response = await app.$axios.$get(getUrl);

    const caseCountData = app.$COLUMN_NAME.CASE_COUNT;
    let maxCount = 0;
    response.forEach((element) => {
      if (maxCount < element.cases) {
        maxCount = element.cases;
      }

      caseCountData.push([element.name_ja, element.cases]);
    });

    caseCountData.sort((a, b) => {
      if (a[0] === '都道府県') return 0;
      return b[1] - a[1];
    });

    return {
      chartData: caseCountData,
      chartMaxCount: maxCount,
      caseCountData,
      caseCountMax: maxCount,
      apiResponse: response,
    };
  },
  data() {
    // パターンごとにオブジェクトきってmaxとdataもたせる
    return {
      isActive: 'caseCount',
      chartData: [],
      chartMaxCount: 0,
      caseCountData: [],
      caseCountMax: 0,
      casePerfData: [],
      casePerfMax: 0,
      deatchCountData: [],
      deatchPerfData: [],
      apiResponse: {},
      barChartOptions: {
        bars: 'horizontal',
        colors: ['#1b9e77', '#d95f02', '#7570b3'],
        titlePosition: 'none',
        height: 700,
        legend: {
          position: 'none',
        },
        axes: {
          y: {
            0: { side: 'right' },
          },
        },
      },
      geoChartOptions: {
        colorAxis: {
          values: [],
          colors: ['#ffffff', 'fa0c18', '#910007'],
        },
        region: 'JP',
        resolution: 'provinces',
        displayMode: 'regions',
      },
      geoLegend: {
        textStyle: { color: 'blue', fontSize: 16 },
      },
    };
  },
  mounted() {
    this.geoChartOptions.colorAxis.values = [
      0,
      this.caseCountMax / 2,
      this.caseCountMax,
    ];
  },
  methods: {
    // computedで再計算させないようにしたい
    caseCount() {
      this.isActive = 'caseCount';
      this.chartData = this.caseCountData;
      this.geoChartOptions.colorAxis.values = [
        0,
        this.caseCountMax / 2,
        this.caseCountMax,
      ];
    },
    casePerf() {
      this.isActive = 'casePerf';

      if (this.casePerfData.length === 0) {
        const casePerfData = this.$COLUMN_NAME.CASE_PERF;
        const POPULATION = this.$POPULATION; // 見えないので別定義

        this.apiResponse.forEach((res) => {
          const population = POPULATION[res.name_ja];
          const perf = Math.round((res.cases / population) * 100000 * 10) / 10;

          casePerfData.push([res.name_ja, perf]);

          if (this.casePerfMax < perf) {
            this.casePerfMax = perf;
          }
        });

        casePerfData.sort((a, b) => {
          return b[1] - a[1];
        });

        this.casePerfData = casePerfData;
      }

      this.chartData = this.casePerfData;

      this.geoChartOptions.colorAxis.values = [
        0,
        this.casePerfMax / 2,
        this.casePerfMax,
      ];
    },
    deathCount() {},
  },
};
</script>
