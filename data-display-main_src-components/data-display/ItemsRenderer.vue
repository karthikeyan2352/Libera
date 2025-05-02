<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
<div class="lf-answer-container">
  <div v-if="!is_loading && total_items && !pdf_builder_mode &&
             (display_items_as=='grid' || display_items_as=='cards')">

    <GoToTop v-if="xs_screen" />

    <div v-if="xs_screen" class="mt-3">
      <ItemsCards />
    </div>
    <div v-else-if="display_items_as=='cards'" class="mt-3">
      <ItemsCards />
    </div>
    <div v-else-if="display_items_as=='grid'"  class="mt-3">
      <ItemsGrid />
    </div>

    <FieldModal v-if="show_field_modal"
                :data="field_modal_data"
                @closeFieldModal="closeFieldModal" />

    <DeleteItemModal v-if="delete_item_initiated" />

  </div>
</div>
</template>

<script>
import { useMq } from "vue3-mq";
import { computed, provide, ref, reactive } from "vue";
import { dataDisplayStore } from '@/store.js'
import ItemsGrid from '@/components/itemsRenderer/ItemsGrid.vue'
import ItemsCards from '@/components/itemsRenderer/ItemsCards.vue'
import FieldModal from '@/components/itemsRenderer/FieldModal.vue'
import DeleteItemModal from '@/components/itemsRenderer/DeleteItemModal.vue'
import GoToTop from '@/components/GoToTop.vue'

export default {
  name: 'ItemsRenderer',
  components: {
    ItemsGrid, ItemsCards, FieldModal, DeleteItemModal, GoToTop
  },

  setup() {
    const store = dataDisplayStore()

    const mq = useMq();
    const xs_screen = computed(() => mq.current == 'xs')
    provide("xs_screen", xs_screen)

    const show_field_modal = ref(false)
    const field_modal_data = reactive({
      field_name: "",
      field_value: "",
      field_label: "",
      item_id: null
    })

    function showFieldEditor(data) {
      if (data !== undefined) {
        field_modal_data.item_id = data.item_id
        field_modal_data.field_name = data.field_name
        field_modal_data.field_value = data.field_value
        field_modal_data.field_label = store.getFieldLabel(data.field_name)
      }
      show_field_modal.value = true
    }
    provide('showFieldEditor', showFieldEditor)

    function closeFieldModal(data) {
      show_field_modal.value = false
    }

    return {
      xs_screen,
      total_items: computed(() => store.items.length),
      field_modal_data, show_field_modal, closeFieldModal,
      is_loading: computed(() => store.downloading_items),
      display_items_as: computed(() => store.display_items_as),
      delete_item_initiated: computed(() => { return store.delete_item_initiated }),
      pdf_builder_mode: computed(() => store.ui.pdf_builder),
    }
  },
}
</script>

<style scoped>

</style>
