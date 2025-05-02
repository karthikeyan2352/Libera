<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div v-if="display_items_as=='grid'">
    <div class="table-responsive" ref="gridRef">
      <table class="table">
        <thead>
          <tr>
            <template v-for="(field, index) in field_index">
              <th v-if="field.name == 'marked'" :key="index"
                  v-on:click="setOrder(field.name)"
                  class="stickyColumn widthZero first-sticky-col"
                  scope="col">
                <div class="ds-marked-col-header" :aria-label="$t('Marked')">
                  <BookmarkIcon size="1.25x" aria-hidden="true" />
                </div>
              </th>
              <th v-else :key="index + 100"
                  :class="[index == 1 && make_sticky ? sticky_column_classes : '']"
                   scope="col">
                <div :style="[index == 1 && make_sticky ? sticky_col_width : grid_column_width]"
                     v-on:click="setOrder(field.name)"
                     :class="{'ds-deleted-field-label': isFieldDeletedField(field.name)}"
                     :title="getFieldLabel(field.name, field.label)">
                  {{ getFieldLabel(field.name, field.label) }}
                </div>
              </th>
            </template>
          </tr>
        </thead>
        <tbody>
          <GridRow v-for="item in items" :key="item.id"
                   :item="item"
                   :field_index="field_index"
                   :highlighted_row="highlighted_row"
                   @click="highlighted_row = item.id"
                   :column_width="grid_column_width"
                   :make_sticky="make_sticky"
                   :sticky_column_classes="sticky_column_classes"
                   :sticky_col_width="sticky_col_width">
          </GridRow>
        </tbody>
      </table>
    </div>
    <ItemPagination />
  </div>
</template>

<script>
import { useI18n } from "vue-i18n"
import { useElementSize } from '@vueuse/core'
import { computed, ref, onMounted } from "vue";
import { dataDisplayStore } from '@/store.js'
import { orderItems } from '@/modules/items.js'
import { ArrowDownIcon, BookmarkIcon } from '@zhuowenli/vue-feather-icons'
import ItemPagination from '@/components/itemsRenderer/ItemPagination.vue'
import GridRow from '@/components/itemsRenderer/GridRow.vue'

export default {
  name: 'ItemsGrid',
  components: {
    ItemPagination, GridRow,
    ArrowDownIcon, BookmarkIcon,
  },

  setup() {
    const store = dataDisplayStore()
    const data_type = store.data_type
    const highlighted_row = ref(0)
    const { t } = useI18n({ useScope: "global" })
    const field_index = computed(() => store.field_index)
    const gridRef = ref(null)
    const { width } = useElementSize(gridRef)
    const grid_column_width = ref({
      'width': '',
      'min-width': '',
      'max-width': ''
    })

    var set_data_column_width = false
    var make_sticky = ref(false)

    function calculateColumnWidth() {
      let grid_width = width.value
      let fields_count = store.field_index.length
      var occupied_width = 0
      if (store.has_row_controls){
        occupied_width = 38
        fields_count = fields_count - 1
      }
      let cel_padding = fields_count * 18

      var col_width = ((grid_width - occupied_width - cel_padding) / (fields_count))
      if (data_type=='answer') {
        set_data_column_width = true
        if (col_width < 130 ) {
          col_width = 130
          make_sticky.value = true
        } else {
          col_width = 240
        }
      }

      //console.log("grid_width", grid_width)
      //console.log("occupied_width", occupied_width)
      //console.log("cel_padding", cel_padding)
      //console.log("fields_count", fields_count)
      //console.log("col_width", col_width)
      //console.log("set_data_column_width", set_data_column_width)
      //console.log("make_sticky", make_sticky.value)

      if (data_type == 'answer' && set_data_column_width) {
        grid_column_width.value = {
          //'width': col_width + 'px',
          //'min-width': col_width + 'px',
          'max-width': col_width + 'px'
        }
      } else {
        grid_column_width.value = {}
      }
    }

    onMounted(() => {
      calculateColumnWidth()
    })

    function getFieldLabel(field_name, field_label) {
      if (store.data_type == 'answer') {
        if (field_name=='marked' || field_name=='created') {
          return t(field_label)
        }
        return field_label
      }
      return t(field_label)
    }

    const sticky_column_classes = computed(() => {
      if (store.ui.delete_mode) {
        return "stickyColumn sticky_data_col sticky-data-col-with-controls-expanded";
      }
      if (store.has_row_controls) {
        return "stickyColumn sticky_data_col sticky-data-col-with-controls";
      }
      return ""
    })

    function setOrder(field_name) {
      // called when column header is clicked
      if (store.user_prefs.order_by != field_name) {
        store.setOrderBy(field_name)
      } else {
        store.setAscending(!store.user_prefs.ascending)
      }
    }

    return {
      gridRef,
      grid_column_width,
      items: computed(() => store.paginatedItems),
      field_index,
      setOrder,
      isFieldDeletedField: store.isFieldDeletedField,
      getFieldLabel,
      make_sticky,
      sticky_column_classes,
      sticky_col_width: {
        "width": "168px",
        "min-width": "168px",
        "max-width": "168px",
      },
      data_type,
      highlighted_row,
      display_items_as: computed(() => store.display_items_as),
    }
  },
}
</script>

<style>
table, caption, tbody, tfoot, thead, tr, th, td {
  margin: 0;
  padding: 0;
  font-size: 100%;
  font: inherit;
  vertical-align: baseline;
}
table {
  border: 1px solid var(--lf-gray-300);
  border-collapse: collapse;
}
table th {
  cursor: pointer;
}
th {
  font-weight: bold;
}
th:not(.stickyColumn), td:not(.stickyColumn) {
  border-right: 1px solid var(--lf-gray-300);
}
th > div, td > div {
  white-space: nowrap;
  overflow: hidden;
	text-overflow: ellipsis;
}
.widthZero {
  width: 0;
}
.stickyColumn {
  background-color: var(--lf-gray-100) !important;
  position: -webkit-sticky;
  position: sticky;
  outline: 1px solid var(--lf-gray-400);
  z-index: 2;
}
.first-sticky-col {
  left: 0px;
  /* border-style: none !important; */
  /*border: 1px solid var(--lf-gray-300);*/
}
@-moz-document url-prefix()  {
  .first-sticky-col {
    left: 1px;
  }
}
.sticky-data-col-with-controls {
  left: 40px;
}
@-moz-document url-prefix()  {
  .sticky-data-col-with-controls {
    left: 41px;
  }
}
.sticky-data-col-with-controls-expanded {
  left: 82px;
}
.ds-marked-col-header {
  margin-left: 2px;
}
</style>
