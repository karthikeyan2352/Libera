<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <tr :class="[is_highlighted ? 'ds-highlighted' : '']">
      <template v-for="(field, index) in field_index">
        <td v-if="field.name == 'marked'"
            class="p-0 stickyColumn first-sticky-col">
          <RowControls :item_id="item.id"
                       :marked="item.marked" />
        </td>
        <td v-else
            :class="[index == 1 && make_sticky ? sticky_column_classes : '',
                     is_highlighted ? 'highlighted' : '']">
          <div :style="[index == 1 && make_sticky ? sticky_col_width : column_width]">
            <FieldRender :field_name="field.name"
                         :field_value="field.name == 'created' ? item.created : item.data[field.name]"
                         :item_id="item.id" />
          </div>
        </td>
      </template>
    </tr>
</template>

<script>
import { computed, watch, ref } from "vue";
import { dataDisplayStore } from '@/store.js'
import FieldRender from '@/components/itemsRenderer/FieldRender.vue'
import RowControls from '@/components/itemsRenderer/RowControls.vue'

export default {
  name: 'GridRow',
  components: {
    RowControls, FieldRender
  },
  props: {
    item: Object,
    field_index: Object,
    make_sticky: Boolean,
    sticky_column_classes: String,
    sticky_col_width: Object,
    column_width: Object,
    highlighted_row: Number,
  },

  setup(props) {
    const store = dataDisplayStore()

    var is_highlighted = ref(false)
    watch(() => props.highlighted_row, () => {
      is_highlighted.value = props.item.id == props.highlighted_row
    })

    return {
      make_sticky: computed(() => props.make_sticky),
      sticky_column_classes: computed(() => props.sticky_column_classes),
      sticky_col_width: props.sticky_col_width,
      column_width: computed(() => props.column_width),
      is_highlighted,
      data_type: computed(() => store.data_type)

    }
  },
}
</script>

<style scoped>

</style>
