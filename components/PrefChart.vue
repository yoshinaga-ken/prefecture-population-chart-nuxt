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

export default {
  name: 'PrefChart',
  mounted() {
    this.getPrefData('total').then((prefData) => {
      this.createChart(prefData.series, prefData.pointStart)
    })
  },
  methods: {
    getPrefData(poplationType) {
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

        const prefCodes = response.data.result.map((d) => d.prefCode)

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

      const options = {
        title: {
          text: '',
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
