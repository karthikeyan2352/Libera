<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>

  <button v-if="total_items && data_type=='answer' && has_multichoice_fields"
          class="btn btn-sm btn-outline-secondary"
          v-on:click="toggleItemFilters()">
    {{ total_filters }}
  </button>

  <button v-if="total_items && data_type=='answer' && has_numeric_fields"
          class="btn btn-sm btn-outline-secondary"
          v-on:click="toggleNumericTotals()">
    {{ $t("Numeric") }}
  </button>

  <button v-if="total_items && can_edit"
          class="btn btn-sm btn-outline-secondary"
          :disabled="!has_items"
          v-on:click="toggleDeleteMode()">
        {{ $t("Delete options") }}
  </button>

  <button v-if="has_deleted_fields && has_items"
          class="btn btn-sm btn-outline-secondary"
          v-on:click="displayDeletedFields()">
    {{ $t("Include deleted fields") }}
    <svg v-if="deleted_fields_enabled" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-check"><polyline points="20 6 9 17 4 12"></polyline></svg>
  </button>

</template>

<script>
import { useI18n } from "vue-i18n";
import { computed, inject } from "vue";
import { dataDisplayStore } from '@/store.js'
import { isNumeric } from '@/composables/fieldProperties.js'

export default {
  name: 'OtherOptions',
  components: {

  },
  setup() {
    const store = dataDisplayStore()
    const { t } = useI18n({ useScope: "global" })

    function toggleOptions() {
      store.ui.other_options = !store.ui.other_options
    }

    function displayDeletedFields() {
      store.include_deleted_fields = !store.include_deleted_fields
      store.searched_items = []
    }
    function toggleNumericTotals() {
      store.ui.numeric_totals = !store.ui.numeric_totals
      store.ui.extended_filters = false
    }
    function toggleItemFilters() {
      store.ui.extended_filters = !store.ui.extended_filters
      store.ui.numeric_totals = false
    }
    function toggleDeleteMode() {
      store.ui.delete_mode = !store.ui.delete_mode
      store.ui.extended_filters = false
      store.ui.numeric_totals = false
    }

    const total_filters = computed(() => {
      if (store.extended_filters === undefined) {
        return t("Filters")
      }
      var total = 0
      Object.keys(store.extended_filters).forEach((field_name) => {
        total = total + store.extended_filters[field_name].length
      })
      return total==0 ? t("Filters") : t("Filters") + ' (' + total + ')'
    })

    const has_numeric_fields = computed(() => {
      for (let field in store.meta.form_structure) {
        if (isNumeric(store.meta.form_structure[field])) {
          return true
        }
      }
      return false
    })

    return {
      has_items: inject('has_items'),
      can_edit: store.can_edit,
      data_type: store.data_type,
      has_deleted_fields: store.deleted_fields.length,
      show_options: computed(() => store.ui.other_options),
      deleted_fields_enabled: computed(() => store.include_deleted_fields),
      total_filters,
      has_multichoice_fields: computed(() => store.all_multichoice_fields.length),
      has_numeric_fields, displayDeletedFields,
      toggleOptions, toggleItemFilters,
      toggleDeleteMode, toggleNumericTotals,
      total_items: computed(() => store.items.length),
    }
  },
}
</script>

<style scoped>
.ds-controls-height {
  min-height: 31px !important;
}
.ds-controls-title svg {
  color: var(--lf-link-color);
  cursor: pointer;
}
</style>
