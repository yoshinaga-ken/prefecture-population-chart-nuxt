<template>
  <div class="pref-chart-container">
    <fieldset class="prefs-chart-checkboxs">
      <legend>都道府県</legend>
      <ul>
        <li v-for="(item, i) in series" :key="i">
          <label
            ><input
              v-model="item.visible"
              type="checkbox"
              @change="onChangePrefCheckbox(item)"
            />{{ item.name }}</label
          >
        </li>
      </ul>
    </fieldset>

    <div ref="chart" class="pref-chart-chart" />
  </div>
</template>

<script>
import _ from 'lodash'
import Highcharts from 'highcharts'
import axios from 'axios'

// RESAS(地域経済分析システム) API
// API: https://opendata.resas-portal.go.jp/docs/api/v1/index.html
const X_API_KEY = '3BKB29A7OA7kXnQ55wyn8Y3lKmGMrT6SnHH24ASv'
const axiosGetFromRESAS = (url) =>
  axios.get(url, { headers: { 'X-API-KEY': X_API_KEY } })
const API_PREFECTURES = 'https://opendata.resas-portal.go.jp/api/v1/prefectures'
const API_PREF_POPULATION =
  'https://opendata.resas-portal.go.jp/api/v1/population/composition/perYear'

const POPULATION_TYPE = ['total', 'yang', 'working', 'aged']
const POPULATION_TYPE_NAME = ['総人口', '年少人口', '生産年齢人口', '老年人口']

export default {
  name: 'PrefChart',
  props: {
    chartTitle: {
      type: String,
      default: null,
    },
    poplationType: {
      type: String,
      default: 'total',
      validator(value) {
        return POPULATION_TYPE.includes(value)
      },
    },
    isDynamicLoad: {
      type: Boolean,
      default: true,
    },
    prefectures: {
      type: Array,
      default: null,
    },
  },
  data() {
    return {
      series: [],
      prefCodes: {},
    }
  },
  mounted() {
    if (this.$props.isDynamicLoad) {
      this.getPrefData(this.$props.poplationType, this.$props.prefectures).then(
        (prefData) => {
          this.prefCodes = prefData.prefCodes
          this.createChart(prefData.series, 1960) // TODO:
        }
      )
    } else {
      this.getAllPrefData(
        this.$props.poplationType,
        this.$props.prefectures
      ).then((prefData) => {
        this.createChart(prefData.series, prefData.pointStart)
      })
    }
  },
  methods: {
    /**
     * [都道府県情報の取得]
     * @return {[Object]}                [description]
     */
    getPrefData() {
      const ret = {
        prefCodes: {},
        series: [],
      }

      // 都道府県のデータを取得
      return axiosGetFromRESAS(API_PREFECTURES).then((response) => {
        for (let i = 0; i < response.data.result.length; i++) {
          ret.prefCodes[response.data.result[i].prefName] =
            response.data.result[i].prefCode
          const item = {
            name: response.data.result[i].prefName,
          }
          ret.series.push(item)
        }
        return Promise.resolve(ret)
      })
    },
    /**
     * [都道府県の人口データを取得]
     * @return {[Object]}                [description]
     */
    getPrefSeriesData(prefCode, poplationType) {
      const ret = {
        data: [],
        pointStart: Number.MAX_SAFE_INTEGER,
      }
      const idx = POPULATION_TYPE.indexOf(poplationType)
      return axiosGetFromRESAS(
        API_PREF_POPULATION + '?cityCode=-&prefCode=' + prefCode
      ).then((responses) => {
        const popTotal = responses.data.result.data[idx].data
        const years = popTotal.map((d) => d.year)
        const values = popTotal.map((d) => d.value)

        // 最小の年度を取得
        const min = _.min(years)
        ret.pointStart = min
        ret.data = values
        return Promise.resolve(ret)
      })
    },

    /**
     * [都道府県情報の取得]
     * @param  {[String]} poplationType [人口タイプ] ('total', 'yang', 'working','aged')
     * @param  {[Array]} prefs           [対象都道府県名]
     * @return {[Object]}                [description]
     */
    getAllPrefData(poplationType, prefs) {
      const ret = {
        prefectures: {},
        series: [],
        pointStart: Number.MAX_SAFE_INTEGER,
      }

      // 都道府県のデータを取得
      return axiosGetFromRESAS(API_PREFECTURES).then((response) => {
        for (let i = 0; i < response.data.result.length; i++) {
          ret.prefectures[response.data.result[i].prefCode] =
            response.data.result[i].prefName
        }

        const prefsFiltered = prefs
          ? response.data.result.filter((d) => prefs.includes(d.prefName))
          : response.data.result

        const prefCodes = prefsFiltered.map((d) => d.prefCode)

        // 全ての都道府県の総人口のデータを取得 (非同期)
        return Promise.all(
          prefCodes.map((prefCode) =>
            axiosGetFromRESAS(
              API_PREF_POPULATION + '?cityCode=-&prefCode=' + prefCode
            )
          )
        ).then((responses) => {
          const idx = POPULATION_TYPE.indexOf(poplationType)
          for (let i = 0; i < responses.length; i++) {
            const popTotal = responses[i].data.result.data[idx].data
            const years = popTotal.map((d) => d.year)
            const values = popTotal.map((d) => d.value)

            // 全ての都道府県で最小の年度を取得
            const min = _.min(years)
            if (min < ret.pointStart) {
              ret.pointStart = min
            }

            const item = {
              name: ret.prefectures[prefCodes[i]],
              data: values,
            }
            ret.series.push(item)
          }

          return Promise.resolve(ret)
        })
      })
    },
    createChart(series, pointStart) {
      Highcharts.setOptions({
        lang: {
          thousandsSep: ',',
          shortMonths: [
            '1月',
            '2月',
            '3月',
            '4月',
            '5月',
            '6月',
            '7月',
            '8月',
            '9月',
            '10月',
            '11月',
            '12月',
          ],
        },
      })

      const chartTitle =
        this.$props.chartTitle === null
          ? '都道府県別 ' +
            POPULATION_TYPE_NAME[
              POPULATION_TYPE.indexOf(this.$props.poplationType)
            ] +
            ' の推移'
          : this.$props.chartTitle

      const options = {
        title: {
          text: chartTitle,
        },

        subtitle: {
          text: '',
        },

        yAxis: {
          title: {
            text: '人口数',
            align: 'high',
            rotation: 0,
            x: 80,
            y: -20,
            style: {
              'font-size': '1.2em',
              'font-weight': 'bold',
            },
          },
          labels: {
            formatter() {
              return Highcharts.numberFormat(this.value, 0, '', ',')
            },
          },
        },

        xAxis: {
          title: {
            text: '年度',
            align: 'high',
            x: 50,
            y: -20,
            style: {
              'font-size': '1.2em',
              'font-weight': 'bold',
            },
          },
          accessibility: {
            rangeDescription: '年度',
          },
        },

        legend: {
          layout: 'vertical',
          align: 'right',
          verticalAlign: 'top',
        },

        plotOptions: {
          series: {
            label: {
              connectorAllowed: false,
            },
            pointStart,
            pointInterval: 5,
          },
        },

        series,

        responsive: {
          rules: [
            {
              condition: {
                maxWidth: 500,
              },
              chartOptions: {
                legend: {
                  layout: 'horizontal',
                  align: 'center',
                  verticalAlign: 'bottom',
                },
              },
            },
          ],
        },
      }

      const chart = Highcharts.chart(this.$refs.chart, options)

      this.series = chart.series

      if (this.$props.isDynamicLoad) {
        // 全てのシリーズを非表示
        for (let i = 0; i < chart.series.length; i++) {
          chart.series[i].update({ visible: false, showInLegend: false }, true)
        }
      }
    },
    onChangePrefCheckbox(item) {
      if (this.$props.isDynamicLoad) {
        if (item.data.length === 0) {
          const prefCode = this.prefCodes[item.name]
          this.getPrefSeriesData(prefCode, this.$props.poplationType).then(
            (d) => {
              item.setData(d.data, false)
              item.update({
                visible: item.visible,
                showInLegend: item.visible,
                pointStart: d.pointStart,
              })
            }
          )
        } else {
          item.update({ visible: item.visible, showInLegend: item.visible })
        }
      } else {
        item.update({ visible: item.visible, showInLegend: item.visible })
      }
    },
  },
}
</script>

<style>
.pref-chart-container {
  background-color: #eee;
  margin-bottom: 1em;
  padding: 1em;
}
.prefs-chart-checkboxs {
  background-color: #fff;
  margin-bottom: 1em;
}
.prefs-chart-checkboxs ul {
  list-style: none;
  text-align: center;
}
.prefs-chart-checkboxs ul li {
  display: inline-block;
  width: 20%;
}
.pref-chart-chart {
  margin-left: auto;
  margin-right: auto;
  width: 100%;
  height: 600px;
}
</style>
