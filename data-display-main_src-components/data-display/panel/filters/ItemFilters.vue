<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>

  <AddFieldFilter :edit_field="edit_field"
                  @cancelEdition="cancelEdition"
                  @saveFilters="saveFilters" />
  <FiltersDisplay />
  <hr class="mt-3"/>

</template>

<script>
import axios from 'axios';
import { provide, ref } from "vue";
import AddFieldFilter from '@/components/panel/filters/AddFieldFilter.vue'
import FiltersDisplay from '@/components/panel/filters/FiltersDisplay.vue'
import { dataDisplayStore } from '@/store.js'

export default {
  name: 'ItemFilters',
  components: {
    AddFieldFilter, FiltersDisplay
  },

  setup() {
    const store = dataDisplayStore()
    const edit_field = ref({})
    const filters = ref({})
    provide("filters", filters)

    resetFilters()

    function resetFilters() {
      filters.value = JSON.parse(JSON.stringify(store.extended_filters))
    }

    function toggleFilter(field_name, option_value) {
      edit_field.value = {}
      if (filters.value[field_name] === undefined) {
        filters.value[field_name] = []
      }
      if (filters.value[field_name].includes(option_value)) {
        let pos = filters.value[field_name].indexOf(option_value)
        if (pos > -1) {
          filters.value[field_name].splice(pos, 1)
          if (filters.value[field_name]==0) {
            delete filters.value[field_name]
          }
        }
      } else {
        filters.value[field_name].push(option_value)
      }
    }
    provide("toggleFilter", toggleFilter)

    function editFilter(field_name) {
      let field_label = store.getFieldLabel(field_name)
      edit_field.value = {name: field_name,
                          label: field_label}
    }
    provide("editFilter", editFilter)

    function deleteFilter(field_name, option_value) {
      toggleFilter(field_name, option_value)
      saveFilters()
    }
    provide("deleteFilter", deleteFilter)

    function saveFilters() {
      store.extended_filters = JSON.parse(JSON.stringify(filters.value))
      axios.patch(store.endpoint + '/set-filters',
                    JSON.stringify({extended_filters: store.extended_filters}),
                    {headers: { 'Content-Type': 'application/json' }})
      .then(function () { // We could use the response argument
        //
      })
      .catch(function (error) {
        console.log(error);
      });
      cancelEdition()
    }
    function cancelEdition() {
      edit_field.value = {}
      resetFilters()
    }

    return {
      edit_field, saveFilters, cancelEdition,
    }
  },
}
</script>

<style scoped>
hr {
  margin: 0;
  margin-left: -1em;
  margin-right: -1em;
}
</style>
