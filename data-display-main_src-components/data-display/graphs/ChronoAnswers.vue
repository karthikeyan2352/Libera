<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>

  <span v-on:click="toggleGraph()">
    <span class="ds-link">
      <span>{{$t("Chronology")}}</span>
      <ChevronUpIcon v-if="show_graph" size="1.5x" />
      <ChevronDownIcon v-else size="1.5x" />
    </span>
    <span>{{time_range}}</span>
  </span>

  <div v-if="show_graph" class="ds-chart-container mt-2" style="height: 350px">
    <Line :data="{
            datasets: [{
                data: data,
                backgroundColor: color,
                borderColor: color,
                pointRadius: point_radius,
                pointHitRadius: 15,
                pointBorderWidth: 0,
                pointHoverRadius: 8,
            }]
          }"
          :options="{
              responsive: true,
              maintainAspectRatio: false,
              animation: false,
              scales: {
              x: {
                type: 'time',
                time: {
                  unit: time_unit,
                  displayFormats: {
                    second: 'kk:mm:ss',
                    minute: 'kk:mm',
                    hour: 'kk:mm',
                    day: 'yyyy-MM-dd',
                    week: 'yyyy-MM-dd',
                    month: 'yyyy-MM',
                    year: 'yyyy-MM',
                  }
                },
                distribution: 'linear',
              },
              y: {
                beginAtZero: false,
                min: 1,
                ticks: { precision: 0 }
              },
            },
            animation: false,
            plugins: {
              legend: {
                display: false,
              }
            }
          }" />
  </div>
</template>

<script>
import axios from 'axios';
import { computed } from "vue";
import 'chartjs-adapter-date-fns';
import { Line } from 'vue-chartjs'
import { ChevronUpIcon, ChevronDownIcon } from '@zhuowenli/vue-feather-icons'
import { format,
         differenceInMinutes,
         differenceInHours,
         differenceInDays,
         differenceInWeeks,
         differenceInMonths,
         differenceInYears} from "date-fns";

import { Chart as ChartJS,
         TimeScale, Filler, LineElement, Tooltip,
         CategoryScale, LinearScale, PointElement } from 'chart.js'
import { dataDisplayStore } from '@/store.js'
import { orderItems } from '@/modules/items.js'


export default {
  name: 'ChronoAnswers',
  components: {
    Line,
    ChevronUpIcon, ChevronDownIcon,
  },

  setup() {
    const store = dataDisplayStore()
    ChartJS.register(Tooltip, TimeScale, Filler, LineElement, PointElement, CategoryScale, LinearScale)

    const ordered_items = computed(() => orderItems(store.itemsToDisplay, "created", true))

    function setChartParams() {
      let items = orderItems(store.itemsToDisplay, "created", true)
      var answer_cnt = 1
      var _data = []
      items.forEach((answer) => {
        _data.push({'x': answer["created"], 'y': answer_cnt})
        answer_cnt = answer_cnt + 1
      })
      data.value = _data
    }

    const color = "rgba(0, 190, 0, 1)"

    const data = computed(() => {
      let items = ordered_items.value
      var answer_cnt = 1
      var _data = []
      items.forEach((answer) => {
        _data.push({'x': answer["created"], 'y': answer_cnt})
        answer_cnt += 1
      })
      return _data
    })

    const time_range = computed(() => {
      if (ordered_items.value.length==0) {
        return ""
      }
      let start_time = new Date(ordered_items.value[0]['created'])
      if (ordered_items.value.length == 1) {
        return format(start_time, "yyyy-LL-dd HH:mm")
      }
      let stop_time = new Date(ordered_items.value[ordered_items.value.length - 1]['created'])
      let start_str = format(start_time, "yyyy-LL-dd")
      let stop_str = format(stop_time, "yyyy-LL-dd")
      return start_str == stop_str ? start_str : start_str + " - " + stop_str
    })

    const time_unit = computed(() => {
      if (ordered_items.value.length < 2) {
        return "day"
      }
      let start_time = ordered_items.value.length ? new Date(ordered_items.value[0]['created']) : undefined
      let stop_time = new Date(ordered_items.value[ordered_items.value.length-1]['created'])
      if (start_time!==undefined && stop_time!==undefined) {
        if (differenceInMinutes(stop_time, start_time) < 1) {
          return 'second'
        }
        if (differenceInHours(stop_time, start_time) < 1) {
          return 'minute'
        }
        if (differenceInDays(stop_time, start_time) < 1) {
          return 'hour'
        }
        if (differenceInWeeks(stop_time, start_time) < 1) {
          return 'day'
        }
        if (differenceInMonths(stop_time, start_time) < 1) {
          return 'week'
        }
        if (differenceInYears(stop_time, start_time) < 1) {
          return 'month'
        }
        return 'year'
      }
      return 'day'
    })

    const point_radius = computed(() => {
      var radius = 8
      if (ordered_items.value.length==0) {
        return radius
      }
      let total_items = ordered_items.value.length
      radius = total_items > 10 ? 7 : radius
      radius = total_items > 30 ? 6 : radius
      radius = total_items > 60 ? 5 : radius
      radius = total_items > 90 ? 3 : radius
      return radius
    })

    function toggleGraph() {
      store.user_prefs.show_chrono_graph = !store.user_prefs.show_chrono_graph
      axios.patch(store.endpoint + '/toggle-chrono-graph',
                    JSON.stringify({visible: store.user_prefs.show_chrono_graph}),
                    {headers: { 'Content-Type': 'application/json' }})
      .then(function (response) {
        //
      })
      .catch(function (error) {
        console.log(error);
      });
    }

    return {
      time_range,
      data, time_unit, point_radius, color,
      show_graph: computed(() => store.user_prefs.show_chrono_graph),
      toggleGraph,
    }
  }
}
</script>

<style scoped>
.ds-link {
  color: var(--lf-link-color);
  text-decoration: underline;
  cursor: pointer;
}
</style>
