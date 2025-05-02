<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
    <RestoreKey
      v-if="need_to_restore && !canceled_restore"
      @keyIsRestored="keyIsRestored"
      @cancelRestore="cancelRestore"
    />
</template>


<script>
import { computed, ref } from "vue";
import { dataDisplayStore } from '@/store.js'
import { loadInjections } from '@/composables/e2eeKeySetup.js';
import RestoreKey from "@/components/keyManager/RestoreKey.vue";

export default {
  name: 'UnlockAnswers',
  components: {
    RestoreKey,
  },
  props: {

  },
  setup() {
    const store = dataDisplayStore()

    const {
      form_key_id,
      editor_key_id,
      public_key_on_server,
    } = loadInjections()

    const canceled_restore = ref(false);

    //store.user_local_public = public_key_on_server
    store.e2ee_form_key_id = form_key_id
    store.e2ee_editor_key_id = editor_key_id

    function keyIsRestored(with_passphrase) {
      //console.log("E2EE answers keyIsRestored()", with_passphrase)
      store.decryptItems(with_passphrase)
    }
    function cancelRestore() {
      canceled_restore.value = true
    }

    return {
      keyIsRestored,
      need_to_restore: computed(() => store.e2ee_state != 'unlocked'),
      canceled_restore,
      cancelRestore,
    }
  },
}
</script>

<style scoped>

</style>
