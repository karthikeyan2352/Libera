<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div v-if="selected_edition" >
    <label class="form-label">{{ $t("Select a date") }}</label>
    <select @change="loadEdition($event.target.value)"
            class="custom-select form-select"
            id="editionSelector"
            aria-label="Order data by this field">
      <option v-for="(edition, index) in editions" :key="edition.id"
              :selected="is_selected(edition.id)"
              v-bind:value="edition.id">
        {{ index == 0 ? $t("Most recent") : $t(edition.created) }}
      </option>
    </select>
  </div>

</template>

<script>
import { computed } from "vue";
import { dataDisplayStore } from '@/store.js'

export default {
  name: 'DiffSelector',
  components: {

  },

  setup() {
    const store = dataDisplayStore()

    function is_selected(id) {
      return id == store.selected_answer_edition.id ? true : false
    }

    function loadEdition(id) {
      store.meta.selected_edition_id = id
    }

    return {
      editions: computed(() => store.items),
      loadEdition,
      is_selected,
      selected_edition: computed(() => store.selected_answer_edition)
    }
  },
}
</script>

<style scoped>

</style>
