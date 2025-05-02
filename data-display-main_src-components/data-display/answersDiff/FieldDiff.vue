<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div class="fw-bold mt-3">
    {{ field.label }}
  </div>
  <div v-html="field_diff ? field_diff : empty_field" class="mt-1" />
</template>

<script>
import Diff from 'vue-jsdiff';
import DOMPurify from 'dompurify';
import { useI18n } from "vue-i18n";
import { computed } from "vue";
import { dataDisplayStore } from '@/store.js'
import { isMultiChoice } from '@/composables/fieldProperties.js'

export default {
  name: 'FieldDiff',
  components: {

  },
  props: {
    field: Object,
    selected_edition: Object,
    compared_with: Object,
  },
  setup(props) {
    const store = dataDisplayStore()
    const { t } = useI18n({ useScope: "global" });

    const diff_template = "<span class='%class%'>%word%</span>"

    //const escapeHTML = (str) => {
    //  return new Option(str).innerHTML
    //}

    function getFormattedValue(value) {
      if (props.field.name.endsWith('__html')) {
        return value.html
      }
      if (props.field.name.startsWith('file')) {
        /* Need to improve: Compares file names. Does not compare file URLs */
        const parser = new DOMParser();
        let doc = parser.parseFromString(value, "text/html");
        return DOMPurify.sanitize(doc.body.firstChild.textContent)
      }
      return store.getFormattedValue({'field_name': props.field.name, 'field_value': value})
    }
    function getWordSpan(word, changed) {
      let word_class = changed=='added' ? "ds-diff-added" : "ds-diff-removed"
      return diff_template.replace(/%class%/, word_class).replace(/%word%/, word)
    }

    function getSelectedFieldValue() {
      let field = props.field
      if (!Object.keys(props.selected_edition.data).includes(field.name)) {
        return null
      }
      let formatted_value = getFormattedValue(String(props.selected_edition.data[field.name]))
      if (field.name.startsWith('file')) {
        return DOMPurify.sanitize(formatted_value)
      }
      return DOMPurify.sanitize(formatted_value, {USE_PROFILES: {html: false}}).trim()
    }

    function getComparedWithFieldValue() {
      let field = props.field
      if (props.compared_with===null ||
          props.compared_with.data[field.name]===null ||
          props.compared_with.data[field.name]===undefined) {
        return ""
      }
      let formatted_value = getFormattedValue(String(props.compared_with.data[field.name]))
      if (field.name.startsWith('file')) {
        return DOMPurify.sanitize(formatted_value)
      }
      return DOMPurify.sanitize(formatted_value, {USE_PROFILES: {html: false}}).trim()
    }

    const field_diff = computed(() => {
      function generateDiffResult(difference) {
        var result = ""
        difference.forEach((diff) => {
          let word_class = diff.added ? 'ds-diff-added' : diff.removed ? 'ds-diff-removed' : ''
          let html = diff_template.replace(/%class%/, word_class)
                                  .replace(/%word%/, getFormattedValue(diff.value))
          result = result + html
        })
        return result
      }
      let selected_value = getSelectedFieldValue()
      if (selected_value===null) {
        return ""
      }
      let compared_with = getComparedWithFieldValue()
      if (props.compared_with==null) {
        return getFormattedValue(selected_value)
      }
      if (selected_value == compared_with) {
        return getFormattedValue(selected_value)
      }
      if (isMultiChoice(props.field.name)) {
        let selected_option_values = selected_value.split(', ')
        let compared_option_values = compared_with.split(', ')
        let all_options = store.getFieldStructure(props.field.name).values
        var _values = []
        all_options.forEach((option) => {
          let option_label = store.getOptionLabel(
            {field_name: props.field.name, option_value: option.value}
          )
          if (selected_option_values.includes(option_label) &&
              compared_option_values.includes(option_label)) {
                _values.push(option_label)
          } else if (!selected_option_values.includes(option_label) &&
                     compared_option_values.includes(option_label)) {
            _values.push(getWordSpan(option_label, 'removed'))
          } else if (selected_option_values.includes(option_label) &&
                     !compared_option_values.includes(option_label)) {
            _values.push(getWordSpan(option_label, 'added'))
          }
        })
        return _values.join(' ')
      }
      if (props.field.name.startsWith('file-')) {
        return generateDiffResult(Diff.diffSentences(compared_with, selected_value))
      }
      return generateDiffResult(Diff.diffWords(compared_with, selected_value))
    })

    return {
      field_diff,
      empty_field: "<span class='ds-empty-field'>%empty%</span>".replace(/%empty%/, t("Empty")),
    }
  },
}
</script>

<style scoped>

</style>
