<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
    <button v-if="!xs_screen" class="btn btn-sm"
           :class="button_classes.grid"
           v-on:click="displayItemsAs('grid')">
     {{ $t("Table") }}
    </button>

    <button v-if="data_type=='answer' || !xs_screen"
            class="btn btn-sm"
            :class="button_classes.cards"
            v-on:click="displayItemsAs('cards')">
     {{ $t("Cards") }}
    </button>

    <button v-if="data_type=='answer'"
           class="btn btn-sm"
           :class="button_classes.graphs"
           v-on:click="displayItemsAs('graphs')">
     {{ $t("Graphs") }}
    </button>

    <button v-if="has_map"
           class="btn btn-sm"
           :class="button_classes.map"
           v-on:click="displayItemsAs('map')">
     {{ $t("Map") }}
    </button>

</template>


<script>
import axios from 'axios';
import { computed, inject, ref } from "vue";
import { dataDisplayStore } from '@/store.js'

export default {
  name: 'DisplayModes',
  components: {

  },

  setup() {
    const store = dataDisplayStore()
    const xs_screen = inject('xs_screen')

    const data_type = store.data_type
    const has_map = store.form_has_map
    const display_mode = ref(store.display_items_as)
    if (data_type=="answer" && display_mode.value=='grid' && xs_screen.value) {
      display_mode.value = 'cards'
    }

    function displayItemsAs(mode) {
      display_mode.value = mode
      if (xs_screen.value) {
        store.ui.slide_options = false
      }
      store.ui.pdf_builder = false
      store.ui.other_options = false
      if (mode=='cards' && xs_screen.value) {
        store.display_items_as = 'grid'
      } else {
        store.display_items_as = mode
      }
      axios.patch(store.endpoint + '/set-landing-page',
                    JSON.stringify({page: store.display_items_as}),
                    {headers: { 'Content-Type': 'application/json' }})
      .then(function () { // we could use response argument
        //
      })
      .catch(function (error) {
        console.log(error);
      });
    }

    return {
      xs_screen, displayItemsAs,
      button_classes: computed(() => {
        if (store.ui.pdf_builder) {
          return {
            'cards': 'btn-outline-secondary',
            'grid': 'btn-outline-secondary',
            'graphs': 'btn-outline-secondary',
            'map': 'btn-outline-secondary'
          }
        }
        return {
          'cards': display_mode.value == 'cards' ? 'btn-primary' : 'btn-outline-secondary',
          'grid': display_mode.value == 'grid' ? 'btn-primary' : 'btn-outline-secondary',
          'graphs': display_mode.value == 'graphs' ? 'btn-primary' : 'btn-outline-secondary',
          'map': display_mode.value == 'map' ? 'btn-primary' : 'btn-outline-secondary'
        }
      }),
      data_type,
      has_map,
    }
  },
}
</script>

<style scoped>
.ds-controls-height {
  min-height: 31px !important;
}
.ds-controls-title svg {
  color: var(--lf-link-color);
  cursor: pointer;
}
</style>
