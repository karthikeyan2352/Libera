<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>

<div class="lf-answer-container">
  <div v-for="(field, loop) in field_index" :key="loop">
    <div v-if="!(field.name == 'marked' || field.name == 'created')">
      <FieldDiff :field="field"
                 :selected_edition="selected_edition"
                 :compared_with="compared_with" />
    </div>
  </div>
</div>

</template>

<script>
import { computed } from "vue";
import { dataDisplayStore } from '@/store.js'
import FieldDiff from '@/components/answersDiff/FieldDiff.vue'

export default {
  name: 'AnswerDiff',
  components: {
    FieldDiff
  },

  setup() {
    const store = dataDisplayStore()

    return {
      field_index: computed(() => store.field_index),
      selected_edition: computed(() => store.selected_answer_edition),
      compared_with: computed(() => store.previous_answer_edition),
    }
  },
}
</script>

<style scoped>

</style>
