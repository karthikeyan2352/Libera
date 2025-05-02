<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>

  <div>
    <RestoreKey
      v-if="is_restoring_key"
      @keyIsRestored="keyIsRestored"
      @cancelRestore="cancelRestore"
     />

    <E2EEStatus
      v-if="show_e2ee_status"
      @initiateRestoreKey="initiateRestoreKey"
    />

    <EncryptionDemo v-if="show_demo" class="mt-4" />

    <div v-if="is_generating_keys"
         class="card">
      <div class="card-header d-flex flex-row justify-content-between align-items-center">
        <template v-if="is_overwriting">
          <h2 v-if="is_overwriting"
              class="text-danger">
            {{ $t("Overwrite your key pair") }}
          </h2>
          <button class="btn btn-sm btn-outline-secondary"
                  @click="cancelGeneration()">
            {{ $t("Cancel") }}
          </button>
        </template>
        <h2 v-else>
          {{ $t("Your personal key pair") }}
        </h2>
      </div>

      <div class="card-body">
        <div class="mb-3">
          <span>
            {{ $t("Generate") }}
          </span>
          <span :class="{'ds-text-muted' : (current_step < 2) }">
            <ChevronRightIcon />{{ $t("Backup") }}
          </span>
          <span :class="{'ds-text-muted' : (current_step < 3) }">
            <ChevronRightIcon />{{ $t("Restore") }}
          </span>
          <span :class="{'ds-text-muted' : (current_step < 4) }">
            <ChevronRightIcon />{{ $t("Save") }}
          </span>
        </div>

        <GenerateKey
          v-if="current_step==1"
          :pgp_passphrase_error="pgp_passphrase_error"
          @generateUserKeys="generateUserKeys"
          :warn_delete_device_keys="Boolean(device_keys.has_key && public_key_on_server)"
        />

        <BackupKey
          v-if="current_step==2"
          :generated_raw_key="generated_raw_key"
          @keyIsBackedUp="keyIsBackedUp"
        />

        <TestRestore
          v-if="current_step==3"
          :with_passphrase="Boolean(pgp_effective_passphrase)"
          :generated_fingerprint="generated_fingerprint"
          @restoreTestIsComplete="restoreTestIsComplete"
        />

        <SaveKey
          v-if="current_step==4"
          :post_errors="post_errors"
          :would_lose_data="device_keys.would_lose_data"
          :key_has_passphrase="Boolean(pgp_effective_passphrase)"
          :is_overwriting="is_overwriting"
          @post_key="postUserPublicKey"
        />
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { useI18n } from "vue-i18n";
import { computed, inject, provide, ref, emit, onMounted, } from "vue";
import { keys, keyStorage, b64, deleteDeviceKeys } from "@/modules/e2ee-answers.js";
import { loadInjections, digestDeviceStorage } from '@/composables/e2eeKeySetup.js';

import E2EEStatus from "@/components/keyManager/user/E2EEStatus.vue";
import BackupKey from "@/components/keyManager/user/BackupKey.vue";
import PassPhrase from "@/components/keyManager/user/PassPhrase.vue";
import GenerateKey from "@/components/keyManager/user/GenerateKey.vue";
import SaveKey from "@/components/keyManager/user/SaveKey.vue";
import TestRestore from "@/components/keyManager/user/TestRestore.vue";
import RestoreKey from "@/components/keyManager/RestoreKey.vue";
import EncryptionDemo from "@/components/keyManager/user/EncryptionDemo.vue";
import { ChevronRightIcon } from "@zhuowenli/vue-feather-icons";

export default {
  name: 'UserKeys',
  components: {
    E2EEStatus,
    BackupKey,
    PassPhrase,
    GenerateKey,
    SaveKey,
    TestRestore,
    RestoreKey,
    EncryptionDemo,
    ChevronRightIcon,
  },
  props: {
    post_endpoint: String,
  },
  setup(props, {emit}) {
    const { t } = useI18n({ useScope: "global" });

    const {
      editor_key_id,
      public_key_on_server,
    } = loadInjections()

    // key specific
    const generated_raw_key = ref(undefined)
    const generated_fingerprint = ref(undefined)
    const generated_pubKey = ref(undefined)
    const pgp_effective_passphrase = ref('')
    const pgp_passphrase_error = ref('')
    provide('pgp_effective_passphrase', pgp_effective_passphrase)
    provide('pgp_passphrase_error', pgp_passphrase_error)

    // UI state
    const is_overwriting = ref(false)
    const is_generating_keys = ref(false)
    const is_backed_up = ref(false)
    const is_restore_test_complete = ref(false)
    const is_restoring_key = ref(false)
    const can_overwrite = ref(false)
    const show_e2ee_status = ref(false)
    const post_errors = ref("")

    const device_keys = ref({})
    onMounted(async () => {
      device_keys.value = await digestDeviceStorage()
      is_generating_keys.value = !device_keys.value.should_restore_key && !public_key_on_server
      can_overwrite.value = public_key_on_server
      if (!public_key_on_server)
        await deleteDeviceKeys()
    })

    const show_demo = computed(() =>
      public_key_on_server &&
      device_keys.value.e2ee_state == 'unlocked' &&
      !is_generating_keys.value
    )

    const current_step = computed(() => {
      if (!is_generating_keys.value)
        return null
      if (!generated_pubKey.value)
        return 1 // generate
      if (!is_backed_up.value)
        return 2 // backup
      if (!is_restore_test_complete.value)
        return 3 // restore
      return 4 // save
    })

    function initiateRestoreKey() {
      is_restoring_key.value = true
    }
    function keyIsRestored(with_passphrase='') {
      window.location.reload();
    }
    function cancelRestore() {
      is_restoring_key.value = false
    }

    async function generateUserKeys() {
      const privKey = await keys.generate(
        editor_key_id,
        pgp_effective_passphrase.value, // <-- added
      );
      generated_pubKey.value = privKey.toPublic()
      generated_fingerprint.value = privKey.fingerprint
      let b64Binary = await privKey.export()
      generated_raw_key.value = await b64.ensureString(b64Binary)
      let unlockedKey = await keys.getPrivate(b64Binary, pgp_effective_passphrase.value)
      // save unlocked to sessionStorage
      await unlockedKey.persist(null, true)
    }
    async function cancelGeneration() {
      await deleteDeviceKeys()
      window.location.reload();
    }

    async function postUserPublicKey() {
      await axios
        .post(props.post_endpoint, {
          e2ee_public_key: {
            fingerprint: generated_pubKey.value.fingerprint,
            public_key: generated_pubKey.value.export(),
          },
        })
        .then(function () {
          window.location.reload();
        })
        .catch(function (error) {
          console.log("post user key error", error)
          let msg =
              error.response.data && error.response.data.msg
                ? error.response.data.msg
                : error.message;
          post_errors.value = msg;
          console.log(post_errors.value)
        });
    }

    function initiateOverwrite() {
      console.log("overwrite_keys_mode")
      is_overwriting.value = true
      is_generating_keys.value = true
    }
    provide('initiateOverwrite', initiateOverwrite)

    // Key creation steps
    function keyIsBackedUp() {
      is_backed_up.value = true;
      navigator.clipboard.writeText("");
    }
    function restoreTestIsComplete() {
      is_restore_test_complete.value = true;
    }

    return {
      show_e2ee_status: computed(() => public_key_on_server && !is_generating_keys.value),
      is_generating_keys,
      is_overwriting,
      is_restoring_key,
      initiateRestoreKey,
      keyIsRestored,
      device_keys,
      show_demo,
      can_overwrite,
      generateUserKeys,
      postUserPublicKey,
      post_errors,
      current_step,
      pgp_passphrase_error,
      generated_raw_key,
      keyIsBackedUp,
      restoreTestIsComplete,
      generated_fingerprint,
      pgp_effective_passphrase,
      cancelGeneration,
      cancelRestore,
    }
  },
}
</script>

<style scoped>
.step-list {
  list-style-type: none; /* Remove bullets */
  padding: 0; /* Remove padding */
  margin: 0 0 0 0; /* Remove margins */
}
.ds-text-muted {
  color: var(--lf-gray-400);
}
.ds-breadcrumbs {

}
</style>
