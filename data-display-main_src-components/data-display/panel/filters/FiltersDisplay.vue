<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div v-for="field in field_index" :key="field.name">
    <FilterDisplay :field="field" />
  </div>

</template>

<script>
import { computed } from "vue";
import FilterDisplay from '@/components/panel/filters/FilterDisplay.vue'
import { dataDisplayStore } from '@/store.js'

export default {
  name: 'FiltersDisplay',
  components: {
    FilterDisplay,
  },

  setup() {
    const store = dataDisplayStore()

    const field_index = computed(() => {
      var _index = []
      for(let field_name in store.extended_filters) {
        _index.push({'name': field_name, 'label': store.getFieldLabel(field_name)})
      }
      return _index
    })

    return {
      field_index
    }
  },
}
</script>

<style scoped>

</style>
