<template>
  <div ref="chart" class="pref-chart-chart" />
</template>

<script>
// import _ from 'lodash'
import Highcharts from 'highcharts'

export default {
  name: 'PrefChart',
  mounted() {
    this.createChart()
  },
  methods: {
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
            pointStart: 1980,
            pointInterval: 5,
          },
        },

        series: [
          {
            name: '東京',
            data: [43934, 52503, 57177, 69658, 97031, 119931, 137133, 154175],
          },
          {
            name: '大阪',
            data: [24916, 24064, 29742, 29851, 32490, 30282, 38121, 40434],
          },
          {
            name: '福岡',
            data: [11744, 17722, 16005, 19771, 20185, 24377, 32147, 39387],
          },
        ],

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
