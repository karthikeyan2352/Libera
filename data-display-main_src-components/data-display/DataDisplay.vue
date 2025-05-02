<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div v-if="show_display_panel">
    <div v-if="is_loading===true" class="ds-loading-data">
      {{ $t('Loading...') }}
    </div>
    <div v-else>
      <DisplayPanel />
    </div>

    <UnlockAnswers v-if="is_e2ee && e2ee_state !== 'unlocked'" />

  </div>
</template>

<script>
import { useMq } from "vue3-mq";
import axios from 'axios';
import { inject, computed, provide, ref, onMounted } from "vue";
import { dataDisplayStore } from '@/store.js'
import DisplayPanel from '@/components/panel/DisplayPanel.vue'
import UnlockAnswers from "@/components/keyManager/UnlockAnswers.vue";

export default {
  name: 'DataDisplay',
  components: {
    DisplayPanel,
    UnlockAnswers,
  },
  setup() {
    const store = dataDisplayStore()
    store.setLanguage(inject('ui_language'))
    axios.defaults.headers.common['X-CSRF-TOKEN'] = inject('csrf_token')
    store.endpoint = inject('endpoint')
    store.is_e2ee = inject('is_e2ee')

    store.e2ee_form_key_id = inject('e2ee_form_key_id')
    store.e2ee_editor_key_id = inject('e2ee_editor_key_id')

    store.enable_exports = inject('enable_exports')
    let display_as = inject('display_as')
    store.display_items_as = display_as ? display_as : "grid"

    onMounted(async () => {
      store.downloadItems()
    })

    const mq = useMq();
    const xs_screen = computed(() => mq.current == 'xs')
    provide("xs_screen", xs_screen)

    return {
      is_loading: computed(() => store.downloading_items),
      show_display_panel: computed(() => store.ui.display_panel),
      is_e2ee: computed(() => store.is_e2ee),
      e2ee_state: computed(() => store.e2ee_state),
    }
  },
}
</script>

<style>
.modal-title {
  line-height: var(--lf-modal-title-line-height);
  font-size: 1.25rem !important;
  font-weight: lighter !important;
}

</style>
