<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div class="mt-1 fw-bold">
    {{ $t("Numeric field totals") }}
  </div>

  <div v-for="[field_name, total], index in Object.entries(totals)" :key="index">
    <div class="mt-2">
      <span class="">{{getLabel(field_name)}}</span>&nbsp;
      <span class="fw-bold">{{total}}</span>
    </div>
  </div>

  <hr class="mb-2 ds-mx-n1"/>

</template>

<script>
import { computed } from "vue";
import { dataDisplayStore } from '@/store.js'

export default {
  name: 'NumericTotals',
  components: {
  },

  setup() {
    const store = dataDisplayStore()

    const totals = computed(() => {
      var result = {}
      store.field_index.forEach((field) => {
        if (field.name.startsWith('number')) {
          result[field.name] = 0
        }
      })
      store.itemsToDisplay.forEach((item) => {
        let fields = Object.keys(item.data)
        for (let f in fields) {
          let field_name = fields[f]
          if (field_name.startsWith('number')) {
            if (!Object.keys(result).includes(field_name)) {
              result[field_name] = 0
            }
            result[field_name] = result[field_name] + Number(item.data[field_name])
          }
        }
      })
      return result
    })

    return {
      totals,
      getLabel: store.getFieldLabel
    }
  },
}
</script>

<style scoped>
.ds-mx-n1 {
  margin-left: -1em !important;
  margin-right: -1em !important;
}
</style>
