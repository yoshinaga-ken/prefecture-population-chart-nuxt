<template>
  <div ref="chart" class="pref-chart-chart" />
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
    prefectures: {
      type: Array,
      default: null,
    },
  },
  mounted() {
    this.getPrefData(this.$props.poplationType, this.$props.prefectures).then(
      (prefData) => {
        this.createChart(prefData.series, prefData.pointStart)
      }
    )
  },
  methods: {
    /**
     * [都道府県情報の取得]
     * @param  {[String]} poplationType [人口タイプ] ('total', 'yang', 'working','aged')
     * @param  {[Array]} prefs           [対象都道府県名]
     * @return {[Object]}                [description]
     */
    getPrefData(poplationType, prefs) {
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
          },
        },

        xAxis: {
          title: {
            text: '年度',
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

      Highcharts.chart(this.$refs.chart, options)
    },
  },
}
</script>

<style>
.pref-chart-chart {
  width: 1200px;
  height: 600px;
}
</style>
