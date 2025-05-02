<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div v-if="!is_loading && display_items_as=='graphs' && total_items && !pdf_builder_mode">

    <GoToTop v-if="xs_screen" />

    <div class="grid mt-3">
      <div class="g-col-12">
        <ChronoAnswers />
      </div>
    </div>

    <div v-if="graph_fields.length" class="mt-3">
      <div class="grid">
        <div v-for="(field, index) in graph_fields"
             :key="'bar'+index"
             :class="chartClasses(field.values)"
             class="g-col-12 mt-3 ds-chart-container">

          <BarChart :field="field"
                    :field_type="getFieldType(field.name)"
                    :color_code="getColorCode(field.name)"/>

        </div>
      </div>
    </div>

  </div>

</template>

<script>
import { useMq } from "vue3-mq";
import { computed, provide, ref, reactive } from "vue";
import { dataDisplayStore } from '@/store.js'
import { isMultiChoice } from '@/composables/fieldProperties.js'

import GoToTop from '@/components/GoToTop.vue'
import ChronoAnswers from '@/components/graphs/ChronoAnswers.vue'
import BarChart from '@/components/graphs/BarChart.vue'

export default {
  name: 'GraphRenderer',
  components: {
    ChronoAnswers, BarChart,
    GoToTop
  },

  setup() {
    const store = dataDisplayStore()

    const mq = useMq();
    const xs_screen = computed(() => mq.current == 'xs')
    provide("xs_screen", xs_screen)

    var field_colors = {}
    var color_code_index = 0
    const color_codes = ["rgba(233, 132, 0, 0.7)", "rgba(182, 0, 181, 0.7)",
                         "rgba(255, 0, 0, 0.7)", "rgba(0, 123 ,255, 0.7)",
                         "rgba(0, 190, 0, 0.7)"]


    function getColorCode(field_id) {
      if (!Object.keys(field_colors).includes(field_id)) {
        color_code_index += 1
        if (color_code_index == color_codes.length) {
          color_code_index = 0
        }
        field_colors[field_id] = color_codes[color_code_index]
      }
      return field_colors[field_id]
    }

    const graph_fields = computed(() => {
      var fields = []
      store.field_index.forEach((index_field) => {
        store.all_field_structures.forEach((field) => {
          if (index_field.name==field.name && field.type !== undefined) {
            if (isMultiChoice(field.type)) {
              fields.push(field)
            }
            if (field.type == "number") {
              fields.push(field)
            }
          }
        })
      })
      return fields
    })

    function getFieldType(field_name) {
      if (field_name.startsWith("number")) {
        return "numeric"
      }
      return "multi"
    }

    function chartClasses(values) {
      if (values === undefined) { // a numeric field
        return "g-col-md-4"
      }
      let values_cnt = values.length
      if (values_cnt > 8) {
        return "g-col-md-12"
      }
      if (values_cnt > 6) {
        return "g-col-md-8"
      }
      return "g-col-md-4"
    }

    return {
      xs_screen,
      is_loading: computed(() => store.downloading_items),
      graph_fields,
      display_items_as: computed(() => store.display_items_as),
      total_items: computed(() => store.items.length),
      getColorCode, getFieldType, chartClasses,
      is_filtered: computed(() => store.items.length != store.itemsToDisplay.length),
      pdf_builder_mode: computed(() => store.ui.pdf_builder),
    }
  }
}
</script>

<style scoped>
.ds-chart-container {
  height: 250px;
  max-height: 250px !important;
  width: calc(100% - 0.5rem) !important;
}
</style>
