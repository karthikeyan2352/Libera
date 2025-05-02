<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>

  <div class="mt-2">
    <div class="form-check" v-for="option in getFieldOptions()" :key="option.value">
      <input type="checkbox"
             class="form-check-input"
             :id="'check'+option.value"
             :value="option.value"
             :checked="isSelected(option.value)"
             v-on:click="toggleFilter(field.name, option.value)"/>
      <label :for="'check'+option.value" class="form-check-label">
        {{ option.label }}
      </label>
    </div>
  </div>

</template>

<script>
import { computed, inject } from "vue";
import { dataDisplayStore } from '@/store.js'

export default {
  name: 'FieldFilter',
  components: {

  },
  props: {
    field: Object,
  },

  setup(props) {
    const store = dataDisplayStore()
    const filters = inject('filters')

    function getFieldOptions() {
      let structure = store.getFieldStructure(props.field.name)
      if (structure.values) {
        return structure.values
      }
    }
    const isSelected = computed(() => (option_value) => {
      return Boolean(filters.value[props.field.name] &&
                     filters.value[props.field.name].includes(option_value))
    })

    return {
      getFieldOptions,
      isSelected,
      toggleFilter: inject("toggleFilter")
    }
  },
}
</script>

<style scoped>

</style>
