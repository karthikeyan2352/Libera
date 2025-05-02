<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <template v-if="is_html_value">
    <span v-if="with_modal"
          v-html="fieldValue"
          :class="fieldClasses"
          class="ds-field-value"
          v-on:click="showModal()">
    </span>
    <span v-else
          v-html="fieldValue"
          :class="fieldClasses"
          class="ds-field-value">
    </span>
  </template>
  <template v-else>
    <span v-if="with_modal"
          :class="fieldClasses"
          class="ds-field-value"
          v-on:click="showModal()">
      {{ fieldValue }}
    </span>
    <span v-else
          :class="fieldClasses"
          class="ds-field-value">
      {{ fieldValue }}
    </span>
  </template>
</template>

<script>
import { useI18n } from "vue-i18n"
import { computed, inject } from "vue";
import { dataDisplayStore } from '@/store.js'
import { fieldProperties } from '@/composables/fieldProperties.js'

export default {
  name: 'FieldRender',
  components: {

  },
  props: {
    field_name: String,
    field_value: undefined,
    item_id: Number,
  },
  setup(props) {

    const store = dataDisplayStore()
    const showFieldEditor = inject('showFieldEditor')
    const { t } = useI18n({ useScope: "global" })
    const { has_value, is_html_value } = fieldProperties(props);

    const fieldValue = computed(() => {
      if (props.field_name.endsWith('__html')) {
        return props.field_value.html
      }
      if (!has_value.value) {
        return "<span class='ds-empty-field'>%empty%</span>".replace(/%empty%/, t("Empty"))
      }
      let _data = {'field_name': props.field_name,
                   'field_value': props.field_value}
      return store.getFormattedValue(_data)
    })

    function is_field_type(type) {
      return props.field_name.startsWith(type)
    }
    const fieldClasses = computed(() => {
      if (typeof props.field_value == 'boolean') {
        return props.field_value ? "badge rounded-pill bg-success" : "badge rounded-pill bg-secondary"
      }
      var classes = ""
      if (store.data_type == "answer" &&
          !is_field_type('created') &&
          !is_field_type('consent') &&
          !is_field_type('file')) {
        classes = "ds-with-pointer"
      }
      if (!has_value.value) {
        classes = classes + " ds-empty-field"
      }
      if (is_field_type('file') && store.is_e2ee) {
        classes = classes + " e2ee-attachment"
      }
      return classes
    })

    const with_modal = computed(() => {
      if (store.data_type == "answer" &&
          !is_field_type('marked') &&
          !is_field_type('created') &&
          !is_field_type('consent') &&
          !is_field_type('file') &&
          !props.field_name.endsWith('__html')) {
        return true
      }
      return false
    })

    function showModal() {
      showFieldEditor({
        "item_id": props.item_id,
        "field_name": props.field_name,
        "field_value": props.field_value
      })
    }

    return {
      fieldValue, fieldClasses, with_modal,
      showModal,
      is_html_value,
    }
  }
}
</script>

<style scoped>
.ds-with-pointer {
  cursor: pointer;
}
</style>

<style>
.ds-field-value > a > span.ds-avatar > img {
  height: 22px;
  width: 22px;
  margin-right: 0.25em;
}
</style>
