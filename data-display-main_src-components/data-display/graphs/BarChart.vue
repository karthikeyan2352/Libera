<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>

  <Bar :options="chartOptions"
       :data="chartData"/>

</template>

<script>
import { useI18n } from "vue-i18n"
import { computed, ref } from "vue";
import { Bar } from 'vue-chartjs'
import { Chart as ChartJS,
         SubTitle, Tooltip, Legend, BarElement, CategoryScale, LinearScale } from 'chart.js'
import { dataDisplayStore } from '@/store.js'
import { isNumeric } from '@/composables/fieldProperties.js'

export default {
  name: 'BarChart',
  components: {
    Bar
  },
  props: {
    field: Object,
    field_type: String,
    color_code: String,
  },

  setup(props) {
    const store = dataDisplayStore()
    const { t } = useI18n({ useScope: "global" })
    const totalcount = ref(null)

    ChartJS.register(SubTitle, Tooltip, Legend, BarElement, CategoryScale, LinearScale)

    function getMultiLabels() {
      var _labels = []
      Object.values(props.field.values).forEach((value, index) => {
        let _label = shortenString(value.label, 24)
        _labels.push(_label)
      })
      return _labels
    }

    function getNumericLabels() {
      return ['Minimum', 'Average', 'Maximum']
    }

    function shortenString(string, length) {
      return string.length < length ? string : string.slice(0, length -2) + '..'
    }

    function getTitle() {
      let x_axis_cnt = props.field_type=="numeric" ? 3 : props.field.values.length
      var max_length = 45
      if (x_axis_cnt > 12) {
        max_length = 185
      }
      if (x_axis_cnt > 6) {
        max_length = 115
      }
      return shortenString(props.field.label, max_length)
    }

    function getMultiData() {
      var _data = []
      Object.values(props.field.values).forEach((value, index) => {
        _data.push(0) // start counting at zero
      })

      store.itemsToDisplay.forEach((item) => {
        let data = item.data
        if (data[props.field.name] !== undefined && data[props.field.name]) {
            let item_values = data[props.field.name].split(', ')
            Object.values(props.field.values).forEach((values, index) => {
              if (item_values.includes(values.value)) {
                _data[index] += 1
              }
            })
        }
      })
      return _data
    }

    function getNumericData() {
      var minimum = undefined
      var maximum = 0
      var number_total = 0

      store.itemsToDisplay.forEach((item) => {
        let data = item.data
        if (data[props.field.name] !== undefined) {
            let item_value = Number(data[props.field.name])
            if (item_value != NaN) {
              if (minimum === undefined || item_value < minimum) {
                minimum = item_value
              }
              if (item_value > maximum) {
                maximum = item_value
              }
              number_total = number_total + item_value
            }
        }
      })
      totalcount.value = isNumeric(props.field) ? number_total : null
      minimum = minimum !== undefined ? minimum : 0
      let average = store.itemsToDisplay.length ? number_total / store.itemsToDisplay.length : 0
      return [minimum, average, maximum]
    }

    return {
      chartOptions: computed(() => {
        return {
          responsive: true,
          maintainAspectRatio: false,
          animation: false,

          scales: {
            y: {
              beginAtZero: true,
              ticks: { precision: 0 },
            },
          },
          plugins: {
            legend: {
              onHover: function (e) {
                e.native.target.style.cursor = 'pointer';
              },
              onLeave: function (e) {
                e.native.target.style.cursor = 'default';
              },
              labels: {
                boxWidth: 20,
                usePointStyle: true,
                pointStyle: "rectRounded",
              },
            },
            subtitle: {
              display: Boolean(totalcount.value),
              text: t("Total") + ": " + totalcount.value,
              position: 'bottom',
              padding: 2,
              font: {
                size: 13
              },
            }
          }
        }
      }),
      chartData: computed(() => {
        return {
          labels: props.field_type=="multi" ? getMultiLabels() : getNumericLabels(),
          datasets: [{
            label: getTitle(),
            backgroundColor: props.color_code,
            fill: true,
            borderWidth: 0,
            barPercentage: 1,
            data: props.field_type=="multi" ? getMultiData() : getNumericData(),
          }]
        }
      }),
    }
  },
}
</script>

<style scoped>

</style>
