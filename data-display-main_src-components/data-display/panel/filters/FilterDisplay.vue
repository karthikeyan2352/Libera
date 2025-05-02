<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>

  <div class="my-2 fw-bold">
    {{ field.label }}
  </div>

  <div class="mt-1">
      <div v-for="option in options"
           class="mt-1 input-group ds-option-filter"
           :key="option.value">
        <button type="button" class="btn btn-sm btn-outline-danger"
                v-on:click="deleteFilter(field.name, option.value)">
          <TrashIcon size="1.25x"/>
        </button>
        <button type="button" class="btn btn-sm btn-outline-secondary"
               v-on:click="editFilter(field.name)">
          <EditIcon size="1.25x"/>
        </button>
        <span class="ds-filter-label text-truncate">{{option.label}}</span>
      </div>
  </div>

</template>

<script>
import { computed, inject } from "vue";
import { TrashIcon, EditIcon } from '@zhuowenli/vue-feather-icons'
import { dataDisplayStore } from '@/store.js'

export default {
  name: 'FilterDisplay',
  components: {
    TrashIcon, EditIcon,
  },
  props: {
    field: Object,
  },

  setup(props) {
    const store = dataDisplayStore()

    const options = computed(() => {
      var _options = []
      if (store.extended_filters[props.field.name]!==undefined) {
        let filters = store.extended_filters[props.field.name]
        store.getFieldOptions(props.field.name).forEach((option) => {
          if (filters.includes(option.value)) {
            _options.push(option)
          }
        })
      }
      return _options
    })

    return {
      options,
      deleteFilter: inject("deleteFilter"),
      editFilter: inject("editFilter"),
    }
  },
}
</script>

<style scoped>

.input-group {
  flex-wrap: initial;
}
.ds-filter-label {
  align-items: center;
  padding: 0.375rem 0.75rem;
  background-color: var(--lf-gray-100);
  border: 1px solid var(--lf-gray-300);
  white-space: nowrap;
}

.ds-option-filter {
  white-space: nowrap;
}

</style>
