<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2025 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div>
    <span v-if="is_html_value"
          v-html="card_title">
    </span>
    <span v-else>
      {{ card_title }}
    </span>
  </div>
</template>

<script>
import DOMPurify from 'dompurify';
import { useI18n } from "vue-i18n"
import { computed, } from "vue";
import { dataDisplayStore } from '@/store.js'
import { fieldProperties } from '@/composables/fieldProperties.js'

export default {
  name: 'CardTitle',
  components: {

  },
  props: {
    field_name: String,
    field_value: undefined,
  },
  setup(props) {
    const store = dataDisplayStore()
    const { t } = useI18n({ useScope: "global" })

    const { has_value, is_html_value } = fieldProperties(props);

    const card_title = computed(() => {
      if (!has_value.value) {
        return '<span class="ds-empty-field">'+t("Empty")+'</span>'
      }
      if (props.field_name.startsWith('file')) {
        const parser = new DOMParser();
        let doc = parser.parseFromString(props.field_value, "text/html");
        return DOMPurify.sanitize(
          doc.body.firstChild.textContent,
          {USE_PROFILES: {html: false}}
        )
      }
      if (is_html_value.value && props.field_value.html.includes('lf-avatar')) {
        return DOMPurify.sanitize(
          props.field_value.html,
          {FORBID_ATTR: ['href']}
        )
      }
      let formatted_data = store.getFormattedValue({
        'field_name': props.field_name,
        'field_value': props.field_value
      })
      return DOMPurify.sanitize(formatted_data,{USE_PROFILES: {html: false}})
    })
    return {
      is_html_value,
      card_title,
    }
  }
}
</script>
