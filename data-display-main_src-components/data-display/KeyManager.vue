<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <template v-if="external_restore">
    <RestoreKey
      v-if="active_external_restore && !canceled_restore"
      @keyIsRestored="keyIsRestored"
      @cancelRestore="cancelRestore"
    />
  </template>
  <UserKeys
    v-else
    :post_endpoint="post_endpoint"
  />
</template>

<script>
import axios from "axios";
import { useI18n } from "vue-i18n";
import { inject, computed, provide, ref } from "vue";
import { keys, keyStorage } from "@/modules/e2ee-answers.js";
import { loadInjections, digestDeviceStorage } from '@/composables/e2eeKeySetup.js';

import UserKeys from "@/components/keyManager/UserKeys.vue";
import RestoreKey from "@/components/keyManager/RestoreKey.vue";

export default {
  name: "KeyManager",
  components: {
    UserKeys,
    RestoreKey,
  },

  setup() {
    const { locale } = useI18n({ useScope: "global" });
    locale.value = inject("ui_language");
    axios.defaults.headers.common["X-CSRF-TOKEN"] = inject("csrf_token");

    const {
      e2ee_status,
    } = loadInjections()

    const active_external_restore = ref(false);
    const canceled_restore = ref(false);

    document.addEventListener("LF_start_restore_e2ee_key_event", function(e) {
      // triggered by a flask template page
      active_external_restore.value = true
      canceled_restore.value = false
    })
    function keyIsRestored() {
      active_external_restore.value = false
      // let the flask template know
      let event = new CustomEvent("LF_e2ee_key_is_restored_event")
      document.dispatchEvent(event);
    }
    function cancelRestore() {
      canceled_restore.value = true
    }

    const o = {
      post_endpoint: inject("endpoint"),
      external_restore: Boolean(inject('external_restore')),
      active_external_restore,
      keyIsRestored,
      cancelRestore,
      canceled_restore,
    };
    return o;
  },
};
</script>

<style scoped>
hr {
  margin: 0;
  margin-left: -1em;
  margin-right: -1em;
}
</style>
<style global>
.modal-title {
  line-height: var(--lf-modal-title-line-height);
  font-size: 1.25rem !important;
  font-weight: lighter !important;
}
.ok-msg {
  color: var(--lf-success);
}
.ko-msg,
.ds-danger {
  color: var(--lf-danger) !important;
}
.ds-link {
  color: var(--lf-link-color);
  cursor: pointer;
}
.ds-e2ee-list {
  list-style-type: none; /* Remove bullets */
  padding: 0; /* Remove padding */
  margin: 0; /* Remove margins */
}
</style>
