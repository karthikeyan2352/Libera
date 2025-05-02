<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div class="mt-0">
    {{ submitted_date }}
  </div>

  <div class="mt-2">
    <span class="author" v-html="author" />&nbsp;
    <span class='ds-diff-removed px-2' />&nbsp;{{ $t("removed") }}
    <span class='ds-diff-added px-2' />&nbsp;{{ $t("added") }}
  </div>

</template>

<script>
import { useI18n } from "vue-i18n";
import { computed } from "vue";
import { dataDisplayStore } from '@/store.js'

export default {
  name: 'EditionInfo',
  components: {

  },

  setup() {
    const store = dataDisplayStore()
    const { t } = useI18n({ useScope: "global" });

    const submitted_date = computed(() => {
      let string = t("Data submitted on the %date%")
      return string.replace(/%date%/, store.selected_answer_edition.created)
    })

    return {
      submitted_date,
      author: computed(() => store.selected_answer_edition.author)
    }
  },
}
</script>

<style scoped>
.author :deep(.ds-avatar > img) {
  height: 22px;
  width: 22px;
  margin-right: 0.25em;
  vertical-align: bottom;
}

</style>
