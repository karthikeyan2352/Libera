<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div class="ds-controls-height ds-button-container ds-mt-n1">
    <button class="btn btn-sm btn-outline-primary"
            v-on:click="show_modal=true">
          {{ $t("Add filters") }}
    </button>
    <button class="btn btn-sm btn-outline-secondary"
            v-on:click="clearAll()">
          {{ $t("Clear") }}
    </button>
  </div>

  <div v-if="show_modal">
    <div class="modal fade show d-block"
         @keydown.esc="closeModal()"
         tabindex="-1"
         role="dialog">
      <div class="modal-dialog modal-lg" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <h5 v-if="selected_field.name" class="modal-title">
              {{ $t("Select field options") }}
            </h5>
            <h5 v-else class="modal-title">
              {{ $t("Select a field") }}
            </h5>
            <button type="button"
                    class="close"
                    data-dismiss="modal"
                    aria-label="Close"
                    v-on:click="closeModal()">
              <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1" stroke-linecap="round" stroke-linejoin="round" class="feather feather-x" aria-hidden="true"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>
            </button>
          </div>

          <div v-if="!selected_field.name" class="modal-body">
              <div v-for="field in field_index" :key="'selector'+field.name"
                   v-on:click="selectField(field)"
                   class="mt-1">
                <span class="badge rounded-pill"
                      :class="isFiltered(field.name) == true ? 'bg-success' : 'bg-secondary'">
                  {{ $t('filtered') }}
                </span>
                {{field.label}}
              </div>
          </div>

          <div v-if="selected_field.name" class="modal-body">
            <div class="fw-bold">
              {{ selected_field.label }}
            </div>
            <div class="mt-2">
              <FieldFilter :field="selected_field" />
            </div>
          </div>

          <div class="modal-footer">
            <button v-if="selected_field.name"
                    type="button" class="btn btn-outline-primary"
                    v-on:click="showAllFields()">
              {{ $t("All fields")}}
            </button>
            <button type="button" class="btn btn-primary"
                    v-on:click="saveFilters()">
                {{ $t("Save") }}
            </button>
            <button type="button"
                    class="btn btn-secondary"
                    v-on:click="closeModal()">
                {{ $t("Cancel") }}
            </button>
          </div>

        </div>
      </div>
    </div>
    <div class="modal-backdrop fade show"></div>
  </div>
</template>

<script>
import _ from 'underscore';
import { computed, ref, reactive, watch, inject } from "vue";
import FieldFilter from '@/components/panel/filters/FieldFilter.vue'
import { isMultiChoice } from '@/composables/fieldProperties.js'
import { dataDisplayStore } from '@/store.js'

export default {
  name: 'AddFieldFilter',
  components: {
    FieldFilter
  },
  props: {
    edit_field: Object,
  },
  emits: ['cancelEdition', 'saveFilters'],

  setup(props, {emit}) {
    const store = dataDisplayStore()
    const show_modal = ref(false)
    const selected_field = reactive({name: "", label: ""})
    const filters = inject('filters')

    function clearSelectedField() {
      selected_field.name = ""
      selected_field.label = ""
    }
    // open modal to edit field options
    watch(() => props.edit_field.name, () => {
      if (props.edit_field.name) {
        selected_field.name = props.edit_field.name
        selected_field.label = props.edit_field.label
        show_modal.value = true
      }
    })

    function selectField(field) {
      selected_field.name = field.name
      selected_field.label = field.label
    }
    function showAllFields() {
      clearSelectedField()
    }
    function saveFilters() {
      show_modal.value = false
      emit('saveFilters')
      clearSelectedField()
    }
    function clearAll() {
      filters.value = {}
      clearSelectedField()
      store.ui.extended_filters = false
      emit('saveFilters')
    }
    function closeModal() {
      show_modal.value = false
      clearSelectedField()
      emit('cancelEdition')
    }

    const isFiltered = computed(() => (field_name) => {
      return Boolean(filters.value[field_name] &&
                     filters.value[field_name].length)
    })

    const field_index = computed(() => {
      var _index = []
      store.field_index.forEach((field) => {
        if (isMultiChoice(field.name)) {
          let deleted = _.find(store.deleted_fields, function(f){ return f.name == field.name; })
          if (!deleted) {
            _index.push(field)
          }
        }
      })
      return _index
    })

    return {
      show_modal, closeModal,
      field_index,
      selected_field, isFiltered, selectField, showAllFields, saveFilters,
      clearAll
    }
  },
}
</script>

<style scoped>
.ds-mt-n1 {
  margin-top: -1em !important;
}
</style>
