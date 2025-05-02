<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>

  <UnlockAnswers v-if="is_e2ee && e2ee_state !== 'unlocked'" />

  <div v-if="is_loading">
    <div class="grid">
      <div v-if="decrypting_items" class="g-col-12">
          {{ $t("Encrypted data") }}
      </div>
      <div v-else class="g-col-12 ds-loading-data">
        {{ $t('Loading...') }}
      </div>
    </div>
  </div>

  <div v-else-if="render_edition">

    <div class="grid">
      <div class="g-col-12">
        <EditionSelector />
      </div>
    </div>
    <hr class="mt-4"/>

    <div class="grid">
      <div class="g-col-12 my-2">
        <EditionInfo />
      </div>
    </div>
    <hr class="mt-1"/>

    <div class="grid mt-1">
      <div id="diffed-answers" class="g-col-12">
        <AnswerDiff />
      </div>
    </div>

  </div>
</template>

<script>
import { useMq } from "vue3-mq";
import { computed, provide, ref, onMounted, inject, } from "vue";
import { dataDisplayStore } from '@/store.js'
import EditionSelector from '@/components/answersDiff/EditionSelector.vue'
import EditionInfo from '@/components/answersDiff/EditionInfo.vue'
import AnswerDiff from '@/components/answersDiff/AnswerDiff.vue'
import UnlockAnswers from "@/components/keyManager/UnlockAnswers.vue";

export default {
  name: 'AnswersDiff',
  components: {
    EditionSelector, EditionInfo, AnswerDiff, UnlockAnswers,
  },

  setup() {
    const store = dataDisplayStore()

    store.setLanguage(inject('ui_language'))
    store.display_items_as = "diff"
    store.endpoint = inject('endpoint')
    store.is_e2ee = inject('is_e2ee')
    store.e2ee_form_key_id = inject('e2ee_form_key_id')
    store.e2ee_editor_key_id = inject('e2ee_editor_key_id')

    onMounted(async () => {
      store.downloadItems()
    })

    return {
      is_loading: computed(() => !store.itemsReadyToDisplay),
      decrypting_items: computed(() => store.decrypting_items),
      render_edition: computed(() => store.selected_answer_edition !== undefined),
      is_e2ee: computed(() => store.is_e2ee),
      e2ee_state: computed(() => store.e2ee_state),
    }
  },
}
</script>

<style scoped>
hr {
  margin: 0;
  margin-left: -1em;
  margin-right: -1em;
}
</style>
