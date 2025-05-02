<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div class="input-group">
    <input type="text"
           v-model="search_text"
           class="form-control"
           @keyup.enter="searchText()"
           :placeholder="$t('Search')"
           aria-describedby="button-to-search">
    <button class="btn btn-outline-secondary appended-button"
            id="button-to-clear-search"
            type="button"
            :aria-label="$t('Clear')"
            v-on:click="clearText()">
      <SkipBackIcon size="1.1x" :aria-label="$t('Clear')" />
    </button>
    <button class="btn btn-outline-secondary appended-button"
            id="button-to-search"
            type="button"
            :aria-label="$t('Search')"
            v-on:click="searchText()">
     <SearchIcon size="1.1x" :aria-label="$t('Search')" />
    </button>
  </div>
</template>

<script>
import _ from 'underscore';
import { SearchIcon, SkipBackIcon } from '@zhuowenli/vue-feather-icons'
import { computed, ref, watch } from "vue";
import { dataDisplayStore } from '@/store.js'

export default {
  name: 'SearchText',
  components: {
    SearchIcon, SkipBackIcon,
  },

  setup() {
    const store = dataDisplayStore()

    const search_text = ref("")

    watch(computed(() => search_text.value), () => {
      if(search_text.value=="") {
        store.search_text = search_text.value
        store.searched_items = []
      }
    })
    function clearText() {
      search_text.value = ""
    }
    function searchText() {
      let text = search_text.value.toLowerCase()
      var result = _.filter(store.items,
                        function(item) {
                          if (item.created.indexOf(text) !== -1) {
                            return item
                          }
                          for (var field_name in item.data){
                            var field_value = item.data[field_name]
                            if (field_value.value !== undefined) {
                              field_value = field_value.value
                            }
                            field_value = store.getFormattedValue({
                                  'field_name': field_name,
                                  'field_value': field_value
                            })
                            field_value = String(field_value).toLowerCase()
                            if (field_value.indexOf(text) !== -1) {
                              return item
                            }
                          }
                        });
      store.search_text = search_text.value
      store.searched_items = result
    }

    return {
      search_text, searchText, clearText,
    }
  },
}
</script>

<style scoped>

</style>
