<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div class="grid ds-container-filters">

    <!-- filter -->
    <div v-if="!xs_screen" class="g-col-12 g-col-md-4">
      <SearchText />
    </div>

    <!-- order by -->
    <div class="g-col-12 g-col-md-4">
      <div class="input-group flex-nowrap">
        <span class="input-group-text">
          {{ $t("Order by") }}
        </span>
        <select @change="setOrderBy($event.target.value)"
                class="custom-select form-select"
                id="fieldSelector"
                aria-label="Order data by this field">
          <option v-for="field in default_field_index" :key="field.name"
                  :selected="order_by === field.name"
                  v-bind:value="field.name">
            {{ getFieldLabel(field.name, field.label) }}
          </option>
        </select>
        <button type="button"
                v-on:click="toggleAscending"
                aria-label="Toggle ascending order"
                class="btn btn-outline-secondary appended-button">
          <ArrowDownIcon v-if="ascending" size="1.1x" aria-label="Descending" />
          <ArrowUpIcon v-else size="1.1x" aria-label="Ascending" />
        </button>
      </div>
    </div>

    <!-- first field -->
    <div class="g-col-12 g-col-md-4">
      <div class="input-group flex-nowrap">
        <span class="input-group-text">
          {{ $t("First field") }}
        </span>
        <select @change="changeIndex($event)"
                class="custom-select form-select"
                id="fieldSelector"
                aria-label="Select first field to display">
          <template v-for="field in default_field_index" :key="field.name">
            <option v-if="field.name != 'marked'"
                    :selected="first_field.name === field.name"
                    v-bind:value="field.name">
              {{ getFieldLabel(field.name, field.label) }}
            </option>
          </template>
        </select>
        <button class="btn btn-outline-secondary appended-button"
                type="button"
                v-on:click="resetFieldIndex()">
          <RotateCcwIcon size="1.1x" aria-label="Reset columns" />
        </button>
      </div>
    </div>

  </div>
</template>

<script>
import { useI18n } from "vue-i18n"
import axios from 'axios';
import { computed, ref, inject } from "vue";
import { ArrowUpIcon, ArrowDownIcon, RotateCcwIcon } from '@zhuowenli/vue-feather-icons'
import { dataDisplayStore } from '@/store.js'
import SearchText from '@/components/panel/filters/SearchText.vue'


export default {
  name: 'ItemControls',
  components: {
    SearchText,
    ArrowUpIcon, ArrowDownIcon, RotateCcwIcon
  },
  setup() {
    const store = dataDisplayStore()
    const { t } = useI18n({ useScope: "global" })
    const ascending = ref(store.user_prefs.ascending)

    const first_field = computed(() => {
      if (store.user_prefs.field_index[0].name != 'marked') {
        return store.user_prefs.field_index[0]
      } else {
        return store.user_prefs.field_index[1]
      }
    })

    function toggleAscending() {  // We could use the event argument
      ascending.value = !ascending.value
      store.setAscending(ascending.value)
    }

    function changeIndex(event) {
      let first_item_name = event.target.value
      var new_index = JSON.parse(JSON.stringify(store.user_prefs.field_index));
      var marked_field = null
      if (store.has_row_controls) {
        marked_field = new_index.shift()
      }
      let item_pos = new_index.findIndex(i => i.name === first_item_name)
      var field = new_index[item_pos];
      new_index.splice(item_pos, 1)
      new_index.splice(0, 0, field);
      if (marked_field) {
        new_index.splice(0, 0, marked_field);
      }
      store.user_prefs.field_index = new_index
      axios.post(store.endpoint + '/change-index', {
        field_index: new_index
      })
      .then(function (response) {
        store.user_prefs.field_index = response.data.field_index
      })
      .catch(function (error) {
        console.log(error);
      });
    }

    function resetFieldIndex() {
      store.user_prefs.field_index = store.default_field_index
      axios.post(store.endpoint + '/reset-index')
      .then(function (response) {
        store.user_prefs.field_index = response.data.field_index
      })
      .catch(function (error) {
        console.log(error);
      })
      .finally(function (){

      });
    }

    function getFieldLabel(field_name, field_label) {
      if (store.data_type == 'answer') {
        if (field_name=='marked' || field_name=='created') {
          return t(field_label)
        }
        return field_label
      }
      return t(field_label)
    }

    return {
      xs_screen: inject('xs_screen'),
      default_field_index: store.default_field_index,
      order_by: computed(() => store.user_prefs.order_by),
      first_field,
      setOrderBy: store.setOrderBy,
      changeIndex, resetFieldIndex,
      toggleAscending, ascending,
      getFieldLabel,
    }
  },
}
</script>

<style scoped>


</style>
