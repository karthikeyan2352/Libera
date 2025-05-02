<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div v-if="enabled_exports.length" class="ds-exports">
    <strong v-if="!xs_screen">{{ $t("Export:") }}</strong>

    <button v-if="enabled_exports.includes('csv')"
       v-on:click="generateCSV">
       <DownloadIcon aria-hidden="true" size="1x" />
       CSV
    </button>
    <button v-if="enabled_exports.includes('json')"
            v-on:click="generateJSON">
      <DownloadIcon aria-hidden="true" size="1x" />
      JSON
    </button>
    <button v-if="enabled_exports.includes('pdf')"
            v-on:click="togglePdfBuilder">
      <FileTextIcon aria-hidden="true" size="1x" />
      PDF
    </button>
  </div>
</template>

<script>
import { inject } from "vue";
import { dataDisplayStore } from '@/store.js'
import { DownloadIcon, FileTextIcon } from '@zhuowenli/vue-feather-icons'
import { saveAs } from 'file-saver'
import {stringify} from 'csv-stringify/browser/esm';

export default {
  name: 'ExportOptions',
  components: {
    DownloadIcon, FileTextIcon
  },

  setup() {
    const store = dataDisplayStore()
    const xs_screen = inject("xs_screen")

    function generateCSV() {
      var data = []
      store.itemsToDisplay.forEach((row) => {
        var values = {}
        store.field_index.forEach((field) => {
          let field_name = field.name

          if (field.name == 'marked') {
            values[field_name] = store.getFormattedValue({field_name: 'marked', field_value: row.marked})
          }
          else if (field.name == 'created') {
            values[field_name] = store.getFormattedValue({field_name: 'created', field_value: row.created})
          }
          else if (row.data[field.name] !== undefined) {
            var field_value = row.data[field.name]
            if (field.name.endsWith('__html')) {
              field_value = row.data[field.name].value
            }
            let _field = {'field_name': field.name,
                          'field_value': field_value}
            values[field_name] = store.getFormattedValue(_field)
          }
          else {
            values[field_name] = ""
          }
        })
        data.push(values)
      });
      let column_lables = function() {
        let lables = []
        store.field_index.forEach((field) => {
          lables.push({'key': field.name, 'header': field.label})
        })
        return lables
      }();

      stringify(data, {
          header: true,
          columns: column_lables,
        },
        function (err, output) {
          let blob = new Blob([output], {type: "text/csv;charset=utf-8"});
          var title = store.meta.title + ".csv"
          if (store.data_type == 'user') {
            title = "users.csv"
          }
          saveAs(blob, title);
      })
    }

    function generateJSON() {

      var answers = []
      let items = store.itemsToDisplay
      let field_index = store.field_index
      items.forEach((item) => {
        var obj = new Object();
        for(let i = 0; i < field_index.length; i++) {
          var field = field_index[i]
          if (field.name == 'marked') {
            obj.marked = item.marked
          }
          else if (field.name == 'created') {
            obj.created = item.created
          }
          else {
            var field_value = item.data[field.name] === undefined ? "" : item.data[field.name]
            if (field.name.startsWith('checkbox') ||
                field.name.startsWith('radio-group') ||
                field.name.startsWith('select')) {

              var option_values = []
              var values = field_value.split(', ')
              values.forEach((value) => {
                var label = store.getOptionLabel({'field_name': field.name,
                                                  'option_value': value})
                option_values.push(label)
              });
              obj[field.name] = option_values.join(', ')
            } else {
              obj[field.name] = field_value
            }
          }
        }
        answers.push(obj)
      });
      let field_lables = function() {
        let lables = {}
        store.field_index.forEach((field) => {
          lables[field.name] = field.label
        })
        return lables
      }();
      let result = JSON.stringify({
        'answers': answers,
        'labels': field_lables,
      })
      var blob = new Blob([result], {type: "application/json;charset=utf-8"});
      saveAs(blob, store.meta.title + ".json");
    }

    function togglePdfBuilder() {
      if (xs_screen.value) {
        store.ui.slide_options = false
      }
      store.ui.other_options = false
      store.ui.pdf_builder = true
    }

    return {
      xs_screen,
      enabled_exports: store.enabled_exports,
      generateJSON, generateCSV, togglePdfBuilder
    }
  },
}
</script>

<style scoped>
a {
  color: var(--bs-body-color);
  text-decoration: none;
}
.ds-exports > button:not(:first-child) {
  margin-left: .5rem;
}
</style>
