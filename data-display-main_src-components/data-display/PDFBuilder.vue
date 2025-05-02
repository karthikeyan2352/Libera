<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div v-if="show_builder" class="mt-3">
    <div class="grid">
      <div class="g-col-12 g-col-lg-3">
        <div class="card">
          <div class="card-header">
            <h4>{{$t("PDF options")}}</h4>
          </div>

          <div class="card-body">

            <div class="">
              <label for="fontSize" class="form-label">{{ $t("Font size") }}</label>
              <select @change="prefs.font_size = $event.target.value"
                      class="form-select"
                      id="fontSize"
                      aria-label="Select font size">
                <option v-for="size in font_sizes" :key="size"
                        :selected="prefs.font_size === size"
                        v-bind:value="size">
                  {{ size }}
                </option>
              </select>
            </div>

            <div class="mt-4">
              <label for="pageSize" class="form-label">{{ $t("Page") }}</label>
              <select @change="prefs.page_size = $event.target.value"
                      class="form-select"
                      id="pageSize"
                      aria-label="Select page size">
                <option v-for="(size, val, index) in page_sizes"
                        :selected="prefs.page_size === val"
                        v-bind:value="val">
                  {{ size }}
                </option>
              </select>
            </div>

            <div class="mt-4">
              <label class="form-label">{{ $t("Page orientation") }}</label><br />
              <div class="form-check form-check-inline">
                <input class="form-check-input" type="radio" id="inlineRadio1"
                       @change="prefs.portrait = !prefs.portrait"
                       :checked="prefs.portrait==false">
                <label class="form-check-label" for="inlineRadio1">{{ $t("Landscape") }}</label>
              </div>
              <div class="form-check form-check-inline">
                <input class="form-check-input" type="radio" id="inlineRadio2"
                       @change="prefs.portrait = !prefs.portrait"
                       :checked="prefs.portrait==true">
                <label class="form-check-label" for="inlineRadio2">{{ $t("Portrait") }}</label>
              </div>
            </div>

            <div class="form-check mt-4">
              <input class="form-check-input"
                     type="checkbox"
                     id="exportMarkedRegisters"
                     v-model="only_marked_items"
                     v-on:click="only_marked_items = !only_marked_items">
              <label class="form-check-label" for="exportMarkedRegisters">
                {{ $t("Only export bookmarked") }}
                <BookmarkIcon size="1.1x" />
              </label>
            </div>

          </div>
        </div>
      </div>

      <div class="g-col-12 g-col-lg-9">
        <div class="card">
          <div class="card-header">
            <h4>{{ $t('Export these fields') }}</h4>
          </div>
          <div class="card-body">
            <template v-for="(field, fieldX) in field_index">
              <div class="form-check mb-1">
                <input :id="fieldX"
                      :key="field.name"
                      type="checkbox"
                      class="form-check-input"
                      :value="field.name"
                      :checked="is_selected(field.name) == true"
                      v-on:click="toggleSelectedField(field.name)">
                <label :for="fieldX" class="form-check-label"
                       :class="{'ds-deleted-field-label': isFieldDeletedField(field.name)}">
                  {{ field.label }}
                </label>
              </div>
            </template>
          </div>
        </div>
      </div>
    </div>

    <div class="grid">
      <div class="g-col-12 ds-button-container">
        <button class="btn btn-primary"
                :disabled="prefs.selected_fields.length == 0"
                v-on:click="generatePDF">
          {{ $t('Download PDF') }}
        </button>
        <button class="btn btn-outline-primary"
                v-on:click="togglePdfBuilder">
          {{ $t('Cancel') }}
        </button>
      </div>
    </div>
  </div>

</template>

<script>
import axios from 'axios';
import { useI18n } from "vue-i18n"
import jsPDF from 'jspdf'
import 'jspdf-autotable'
import { useMq } from "vue3-mq";
import { computed, provide, ref, reactive, watch } from "vue";
import { dataDisplayStore } from '@/store.js'
import { BookmarkIcon } from '@zhuowenli/vue-feather-icons'

export default {
  name: 'PDFBuilder',
  components: {
    BookmarkIcon,
  },

  setup() {
    const store = dataDisplayStore()

    const { t } = useI18n({ useScope: "global" })
    const mq = useMq();
    const xs_screen = computed(() => mq.current == 'xs')
    provide("xs_screen", xs_screen)
    const only_marked_items = ref(false)
    const font_sizes = ["10", "9", "8", "7", "6", "5"]
    const page_sizes = {'a4': 'A4', 'a3': 'A3', 'letter': 'Letter', 'ledger':'Ledger'}
    const prefs = reactive(JSON.parse(JSON.stringify(store.pdf_prefs)))

    watch(() => store.pdf_prefs.default, (current_value) => {
      prefs.selected_fields = JSON.parse(JSON.stringify(store.pdf_prefs.selected_fields))
      prefs.font_size = store.pdf_prefs.font_size
      prefs.page_size = store.pdf_prefs.page_size
      prefs.portrait = store.pdf_prefs.portrait
    })

    const field_index = computed(() => {
      var _field_index = store.field_index
      if (prefs.selected_fields.length == 0) {
        // set field selection to default values
        for (let field in _field_index) {
          if (_field_index[field].name == 'marked') {
            continue
          }
          toggleSelectedField(_field_index[field].name)
        }
      }
      return _field_index
    })

    function generatePDF() {
      const doc = new jsPDF({
                      orientation: prefs.portrait ? 'p' : 'l',
                      format: prefs.page_size
                  })
      var totalPagesExp = '{total_pages_count_string}'
      var pdf_name = store.meta.title
      var page_string = t("Page")
      doc.autoTable({
        theme: 'grid',
        headStyles: {fontWeight: 'normal',
                     overflow: 'ellipsize',
                     fontSize: prefs.font_size,
                     fillColor: '#666'},
        bodyStyles: {fontSize: prefs.font_size},
        head: getHeader(),
        body: getRows(),
        didDrawPage: function (data) {
          // Header
          doc.setFontSize(14)
          doc.setTextColor(32, 32, 32)
          // (x,y)
          doc.text(pdf_name, data.settings.margin.left + 0, 12)

          // Footer
          var str = page_string + ' ' + doc.internal.getNumberOfPages()
          if (typeof doc.putTotalPages === 'function') {
            str = str + ' of ' + totalPagesExp
          }
          doc.setFontSize(10)

          // jsPDF 1.4+ uses getWidth, <1.4 uses .width
          let pageSize = doc.internal.pageSize
          let pageHeight = pageSize.height ? pageSize.height : pageSize.getHeight()
          doc.text(str, data.settings.margin.left, pageHeight - 10)
        }
      })
      if (typeof doc.putTotalPages === 'function') {
        doc.putTotalPages(totalPagesExp)
      }
      doc.save(pdf_name + '.pdf')
      saveUserPreferences()
    }

    function getHeader() {
      var columns = []
      field_index.value.forEach((field, i) => {
        if (prefs.selected_fields.includes(field.name)) {
          columns.push(field.label)
        }
      });
      return [columns]
    }
    function getRows() {
      //let url_regex = /<a[^>]+href=\"(.*?)\"[^>]*>/
      let url_regex = /<a\s+(?:[^>]*?\s+)?href=(["'])(.*?)\1/
      var rows = []
      for (let i in store.itemsToDisplay) {
        var item = store.itemsToDisplay[i]
        if (only_marked_items.value && item.marked!=true) {
          continue
        }
        var row = []
        for (let f in field_index.value) {
          var field = field_index.value[f]
          if (prefs.selected_fields.includes(field.name)) {
            if (field.name == 'marked' || field.name == 'created') {
              let field_data = {'field_name': field.name,
                                'field_value': item[field.name]}
              row.push(store.getFormattedValue(field_data))
              continue
            } else if (field.name.startsWith('file-')) {
              if (item.data[field.name] !== undefined) {
                let url = item.data[field.name].match(url_regex);
                url != null ? row.push(url[2]) : row.push("")
              } else {
                row.push("")
              }
              continue
            } else if (!item.data[field.name]) {
              row.push("")
              continue
            }
            let field_data = {'field_name': field.name,
                              'field_value': item.data[field.name]}
            row.push(store.getFormattedValue(field_data))
          }
        }
        rows.push(row)
      }
      return rows
    }

    const is_selected = computed(() => (field_name) => {
      return prefs.selected_fields.includes(field_name)
    })
    function togglePdfBuilder() {
      store.ui.pdf_builder = !store.ui.pdf_builder
    }
    function toggleSelectedField(field_name) {
      if (prefs.selected_fields.includes(field_name)) {
        prefs.selected_fields.splice(prefs.selected_fields.indexOf(field_name), 1)
      } else {
        prefs.selected_fields.push(field_name)
      }
    }
    function saveUserPreferences() {
      prefs.default = false
      store.pdf_prefs = prefs
      axios.patch(store.endpoint + '/set-pdf-preferences',
                    JSON.stringify({prefs: store.pdf_prefs}),
                    {headers: { 'Content-Type': 'application/json' }})
      .then(function (response) {
        //
      })
      .catch(function (error) {
        console.log(error);
      });
    }

    return {
      xs_screen, field_index, prefs,
      show_builder: computed(() => store.ui.pdf_builder),
      togglePdfBuilder, toggleSelectedField,
      is_selected, only_marked_items, page_sizes, font_sizes,
      isFieldDeletedField: store.isFieldDeletedField,
      generatePDF,
    }
  },
}
</script>

<style scoped>
.card {
  border-top: 1px solid var(--lf-gray-300);
}
</style>
