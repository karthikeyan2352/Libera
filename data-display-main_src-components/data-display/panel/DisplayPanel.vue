<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>

  <template v-if="data_type=='answer'">

    <template v-if="xs_screen">

      <div class="mt-2">
        <SearchText />
      </div>

      <div class="mt-3"
           :class="{'d-flex align-items-start justify-content-between': enable_exports && show_options}">
        <a class="toggle-panel-btn" v-on:click="toggleOptions()">
          <span>{{ $t("Options") }}</span>
          <ChevronDownIcon v-if="show_options==false" size="1.5x" aria-hidden="true" />
          <ChevronUpIcon v-else class="expand_panel_icon" size="1.5x" aria-hidden="true" />
        </a>
        <div v-if="enable_exports && show_options" class="ds-controls-height">
          <ExportOptions />
        </div>
      </div>

      <!-- options start -->
      <slide-up-down :active="show_options"
                     :duration="slide_duration"
                     @close-end="slideFinished()">
        <div class="ds-button-container">
          <DisplayModes />
        </div>
        <div class="mt-4">
          <ItemControls />
          <div class="mt-2 ds-button-container">
            <OtherOptions />
          </div>
          <div v-if="show_delete_all" class="mt-3">
            <DeleteAllItems />
          </div>
          <div v-if="extended_filters" class="mt-3">
            <ItemFilters />
          </div>
          <div v-if="show_numeric_totals" class="mt-3">
            <NumericTotals />
          </div>
        </div>
      </slide-up-down >
      <!-- options end -->
      <div class="mt-3">
        {{ displaying_total_of }}
      </div>

    </template>

    <template v-if="!xs_screen">
      <div :class="controlGroupClasses" class="ds-mt-n1">
        <div class="ds-controls-height ds-button-container">
          <DisplayModes />
        </div>
        <div v-if="enable_exports">
          <ExportOptions />
        </div>
      </div>

      <div class="mt-3">
        <ItemControls />
      </div>

      <div class="d-flex flex-wrap flex-md-nowrap align-items-center justify-content-between">
        <div class="mt-3">
          {{ displaying_total_of }}
        </div>
        <div class="ds-controls-height ds-button-container">
          <OtherOptions />
        </div>
      </div>
      <div v-if="show_delete_all"
           class="mt-2 ds-controls-height ds-button-container">
        <DeleteAllItems />
      </div>
      <div v-if="extended_filters" class="mt-2">
        <ItemFilters />
      </div>
      <div v-if="show_numeric_totals" class="mt-2">
        <NumericTotals />
      </div>

    </template>
  </template>
  <!-- end render data_type = answers -->

  <template v-else-if="xs_screen">

    <div class="mt-2">
      <SearchText />
    </div>

    <div class="mt-3"
         :class="{'d-flex align-items-start justify-content-between': enable_exports && show_options}">
      <a class="toggle-panel-btn" v-on:click="toggleOptions()">
        <ChevronDownIcon v-if="show_options==false" size="1.5x" aria-hidden="true" />
        <ChevronUpIcon v-else class="expand_panel_icon" size="1.5x" aria-hidden="true" />
        <span>{{ $t("Options") }}</span>
      </a>
      <div v-if="enable_exports && show_options" class="ds-controls-height">
        <ExportOptions />
      </div>
    </div>

    <slide-up-down :active="show_options" :duration="200">
      <div>
        <div class="mt-3">
          <ItemControls />
        </div>
        <div class="mt-3">
          <DisplayModes />
        </div>
      </div>
    </slide-up-down>

    <div class="mt-3">
      {{ displaying_total_of }}
    </div>
  </template>

  <template v-else>
    <div v-if="enable_exports"
         :class="controlGroupClasses"
         class="mb-3">
      <ExportOptions />
    </div>
    <div class="mt-2">
      <ItemControls />
    </div>
    <div class="mt-3">
      {{ displaying_total_of }}
    </div>
  </template>

</template>


<script>
import { useI18n } from "vue-i18n"
import { computed, provide, inject } from "vue";
import { dataDisplayStore } from '@/store.js'
import { ChevronDownIcon, ChevronUpIcon } from '@zhuowenli/vue-feather-icons'
import ItemControls from '@/components/panel/ItemControls.vue'
import ExportOptions from '@/components/panel/ExportOptions.vue'
import DisplayModes from '@/components/panel/DisplayModes.vue'
import OtherOptions from '@/components/panel/OtherOptions.vue'
import DeleteAllItems from '@/components/panel/DeleteAllItems.vue'
import SearchText from '@/components/panel/filters/SearchText.vue'
import ItemFilters from '@/components/panel/filters/ItemFilters.vue'
import NumericTotals from '@/components/itemsRenderer/NumericTotals.vue'

export default {
  name: 'DisplayPanel',
  components: {
    ItemControls, ExportOptions, DisplayModes, OtherOptions, DeleteAllItems, ItemFilters,
    SearchText, NumericTotals,
    ChevronDownIcon, ChevronUpIcon,
  },

  setup() {
    const store = dataDisplayStore()
    const xs_screen = inject('xs_screen')
    const { t } = useI18n({ useScope: "global" })

    provide('has_items', computed(() => {return store.itemsToDisplay.length}))

    function toggleOptions() {
      store.ui.other_options = !store.ui.other_options
    }
    function slideFinished() {
      store.ui.slide_options = true
    }

    const controlGroupClasses = computed(() => {
      if (xs_screen.value) {
        return "mb-3"
      }
      if (store.data_type=='answer') {
        return 'd-flex align-items-center justify-content-between'
      }
      else if (store.enabled_exports.length) {
        return 'd-flex align-items-center justify-content-end'
      }
      return "mt-1";
    })

    const show_displaying_of = computed(() => {
      if (store.display_items_as != 'grid' || xs_screen.value) {
        return true
      }
      if (store.items.length != store.itemsToDisplay.length ||
          Object.keys(store.extended_filters).length) {
        return true
      }
      return false
    })

    const displaying_total_of = computed(() => {
      let total = store.items.length
      if (total == 0) {
        return store.is_e2ee && store.decrypting_items ? t('Encrypted data') : t('No data')
      }
      let filtered = store.itemsToDisplay.length
      if (Object.keys(store.extended_filters).length ||
          store.searched_items.length ||
          store.search_text) {
        return t("Filtered %number% of %total%").replace(/%number%/, filtered)
                                                .replace(/%total%/, total)
      }
      if (filtered==total) {
        if (store.data_type == "form") {
          return t("Total forms") + ": " + total
        }
        if (store.data_type == "answer") {
          return t("Total answers") + ": " + total
        }
        if (store.data_type == "user") {
          return t("Total users") + ": " + total
        }
        return t("Total") + ": " + total
      }
      return t("Displaying %number% of %total%").replace(/%number%/, filtered)
                                                .replace(/%total%/, total)
    })

    const show_delete_all = computed(() => {
      if (store.ui.delete_mode && store.can_edit &&
          !(store.ui.extended_filters || store.ui.numeric_totals || store.ui.pdf_builder)) {
        return true
      }
      return false
    })

    return {
      xs_screen,
      controlGroupClasses,
      data_type: store.data_type,
      can_edit: store.can_edit,
      toggleOptions, slideFinished, show_delete_all,
      show_options: computed(() => store.ui.other_options),
      delete_mode: computed(() => store.ui.delete_mode),
      enable_exports: store.enabled_exports.length,
      extended_filters: computed(() => store.ui.extended_filters),
      show_numeric_totals: computed(() => store.ui.numeric_totals),
      show_displaying_of, displaying_total_of,
      slide_duration : computed(() => store.ui.slide_options ? 200 : 1),
    }
  },
}
</script>

<style scoped>
.ds-mt-n1 {
  margin-top: -1em;
}
.ds-controls-height {
  min-height: 31px !important;
}

</style>
<style>
.ds-controls-title svg {
  color: var(--lf-link-color);
  cursor: pointer;
}
.ds-button-container {
  display: flex;
  flex-wrap: wrap;
  gap: 0.25rem;
}
.ds-button-container > button {
  margin-top: 1em;
}
</style>
