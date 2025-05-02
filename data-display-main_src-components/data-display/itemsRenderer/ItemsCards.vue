<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div v-if="display_items_as=='cards' || xs_screen" class="my-0">

    <div class="ds-card-list"
         :class="xs_screen ? 'mt-4' : 'mt-3'">
      <ItemCard v-for="item in items" :key="item.id"
                :item="item"
                :field_index="field_index"
                :expand_all="false" />
    </div>

    <div v-if="more_cards_to_load" class="mt-3 ds-more-cards"
         @click="LoadMoreCards()">
        {{ $t('Show more') }}
    </div>

  </div>
</template>

<script>
import { computed, ref, inject } from "vue";
import { ChevronUpIcon } from '@zhuowenli/vue-feather-icons'
import ItemCard from '@/components/itemsRenderer/ItemCard.vue'
import { dataDisplayStore } from '@/store.js'
import { orderItems } from '@/modules/items.js'

export default {
  name: 'ItemsCards',
  components: {
    ItemCard, ChevronUpIcon,
  },

  setup() {
    const store = dataDisplayStore()
    const xs_screen = inject('xs_screen')

    const show_goto_top = ref(false)
    const goto_top = ref(false)
    const field_index = computed(() => store.field_index)

    const siteThemeColors = computed(() => {
      // a hack just to avoid sending dynamic site theme css
      // from the flask app down to the component
      let nav_bar = document.getElementsByClassName("ds-main-navbar")[0]
      let nav_bar_style = window.getComputedStyle(nav_bar)
      let backgound_color = nav_bar_style.getPropertyValue('background-color')
      let nav_brand = document.getElementsByClassName("navbar-brand")[0]
      let nav_brand_style = window.getComputedStyle(nav_brand)
      let color = nav_brand_style.getPropertyValue('color')
      return {
        'color': color,
        'background-color': backgound_color
      }
    })

    const more_cards_to_load = computed(() => store.paginatedCards.length < store.itemsToDisplay.length)
    function LoadMoreCards() {
      store.card_batch = store.card_batch+1
    }

    return {
      xs_screen,
      show_goto_top, goto_top,
      siteThemeColors,
      field_index,
      more_cards_to_load, LoadMoreCards,
      items: computed(() => store.paginatedCards),
      total_items: computed(() => store.itemsToDisplay.length),
      display_items_as: computed(() => store.display_items_as),
    }
  },
}
</script>

<style>
.ds-more-cards {
  color: var(--lf-link-color);
  text-decoration: underline;
  cursor: pointer;
}
.ds-card-list .card {
  border-top: 1px solid var(--lf-gray-300);
}
.ds-card-list li > span:first-child {
  margin-right: 0.5em;
}
.ds-card-list .card-header {
  padding: 0;
  border-bottom: 0;
}
.ds-card-list .card-header .ds-avatar > img {
  height: 22px !important;
  width: 22px !important;
}
.ds-card-list .card.active {
  /* border-top: 1px solid var(--lf-gray-400); */
}
.ds-card-list .card.active .card-header {
  background-color: var(--lf-gray-200);
  border-bottom: var(--lf-card-border-width) solid var(--lf-card-border-color);
}
.ds-card-list .card:not(:first-child) {
  margin-top: 0.5em;
}
.ds-card-list .card-body {
  padding: 0; /*var(--lf-card-spacer-y) calc(var(--lf-card-spacer-x) / 2);*/
}
</style>
