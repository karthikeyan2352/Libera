<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <a v-if="show_button"
     class="btn btn-sm btn-outline-danger modes"
     role="button"
     :href="endpoint+'/delete-all-items'">
    {{ button_text }}
  </a>
</template>

<script>
import { useI18n } from "vue-i18n";
import { computed } from "vue";
import { dataDisplayStore } from '@/store.js'

export default {
  name: 'DeleteAllItems',
  components: {

  },

  setup() {
    const store = dataDisplayStore()
    const { t } = useI18n({ useScope: "global" })

    const button_text = computed(() => {
      let total = store.items.length
      let filtered = store.itemsToDisplay.length
      if (total == filtered) {
        return t("Delete all")
      }
      return t("Delete all") + " (" + total + ")"
    })

    return {
      endpoint: store.endpoint,
      button_text,
      show_button: computed(() => store.items.length)
    }
  },
}
</script>

<style scoped>

</style>
