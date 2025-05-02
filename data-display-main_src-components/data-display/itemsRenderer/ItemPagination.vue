<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div v-if="is_loading === false"
       class="d-flex align-items-center justify-content-end mb-2">
    <small class="page_count_display me-2">
      {{ getPaginationText() }}
    </small>
    <nav aria-label="Paginate items">
      <ul class="pagination pagination-sm mb-0">
        <li class="page-item"
            :class="{disabled: page_number==1}">
          <a class="page-link"
             v-on:click="decrementPage">
            {{ $t("Previous") }}
          </a>
        </li>
        <template v-for="page in total_pages">
          <li v-if="showPageButton(page) === true"
              class="page-item"
              :class="{active: page==page_number}">
            <a class="page-link"
               v-on:click="setPage(page)">
              {{ page }}
            </a>
          </li>
          <li v-else-if="showPageButton(page) === false"
              class="page-item disabled"
              :class="{active: page==page_number}">
            <a class="page-link">
              ...
            </a>
          </li>
        </template>
        <li class="page-item"
            :class="{disabled: page_number==total_pages}">
          <a class="page-link"
             v-on:click="incrementPage">
             {{ $t("Next") }}
          </a>
        </li>
      </ul>
    </nav>
  </div>
</template>

<script>
import { useI18n } from "vue-i18n"
import { computed } from "vue";
import { dataDisplayStore } from '@/store.js'

export default {
  name: 'ItemPagination',
  components: {

  },
  setup() {
    const store = dataDisplayStore()
    const { t } = useI18n({ useScope: "global" })

    const total_pages = computed(() => {
      let total_items_count = store.itemsToDisplay.length
      return Math.ceil(total_items_count / store.page_length)
    })

    function getPaginationText() {
      var slice_start = (store.page - 1) * store.page
      var total_items = store.itemsToDisplay.length
      if (total_items < store.page_length) {
        var displayed_items = total_items
      } else {
        var last_item = store.page * store.page_length
        last_item = last_item > total_items ? total_items : last_item
        var first_item = ((store.page -1) * store.page_length) + 1
        var displayed_items = first_item + " - " + last_item
      }
      return t("Displaying %number% of %total%").replace(/%number%/, displayed_items)
                                                .replace(/%total%/, total_items)
    }


    function showPageButton(page) {
      if (this.total_pages <= 7) {
        //console.log("show all")
        return true
      }
      if (page == store.page || page == 1 || page == this.total_pages) {
        return true
      }
      if (page <= 5 && store.page < 4) {
        return true
      }
      if (page == 6 && store.page < 5) {
        return false
      }
      if (page >= this.total_pages -4 && store.page > this.total_pages -3 ) {
        return true
      }
      if (page == this.total_pages -5 && store.page > this.total_pages -4) {
        return false
      }
      if (store.page >= 4) {
        if (page == store.page -1 || page == store.page +1) {
          return true
        } else if (page == store.page -2 || page == store.page +2){
          return false
        }
      }
      return null
    }
    function decrementPage() {
      if (store.page > 1) {
        store.page = store.page-1
      }
    }
    function incrementPage() {
      if (store.page < this.total_pages) {
        store.page = store.page+1
      }
    }
    function setPage(page) {
      store.page = page
    }

    return {
      is_loading: computed(() => store.downloading_items),
      page_number: computed(() => store.page),
      total_pages,
      getPaginationText,
      showPageButton,
      decrementPage, incrementPage, setPage
    }
  },
}
</script>

<style scoped>
.pagination {
  cursor: pointer;
}
</style>
