<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
<div>
  <div class="card">
    <div class="card-header d-flex flex-row justify-content-between align-items-center">
      <h2>
        {{ $t("Key status") }}
      </h2>
      <a v-if="is_options_available"
         href="javascript:void(0);"
         v-on:click="expanded_options = !expanded_options">
        <span>{{ $t("Options") }}</span></a
      >
    </div>

    <KeyOptions
      v-if="is_options_available && expanded_options"
      class="card-header"
      @collapseOptions="collapseOptions"
    />

    <div class="card-body">
      <ul class="ds-e2ee-list">
        <li>
          <SanityCheck
            :checked="true"
            :ok_label="$t('The server has a copy of your public key')"
          />
        </li>
        <li>
          <SanityCheck
            :checked="device_keys.has_key"
            :ok_label="$t('A private key is available on this browser')"
            :ko_label="$t('A private key is NOT available on this browser')"
          />
        </li>
        <li>
          <SanityCheck
            v-if="device_keys.has_key"
            :checked="device_keys.user_key_remote_match"
            :ok_label="$t('The private key matches your public key')"
            :ko_label="$t('The private key does NOT match your public key')"
          />
        </li>
        <li v-if="device_keys.e2ee_state=='need_passphrase'">
          <SanityCheck
            :checked="e2ee_state=='need_passphrase'"
            :ko_label="$t('Your passphrase is required to unlock your private key')"
          />
        </li>
        <li v-else>
          <SanityCheck
            :checked="device_keys.user_key_remote_match && device_keys.e2ee_state=='unlocked'"
            :ok_label="$t('You can decrypt answers on this browser')"
            :ko_label="$t('You cannot decrypt answers on this browser')"
          />
        </li>
      </ul>

      <div v-if="device_keys.should_restore_key"
           class="my-3">
        <a href="javascript:void(0);"
           v-on:click="emit('initiateRestoreKey')">
          {{ $t("Restore your key on this browser") }}</a
        >
      </div>
      <div v-else-if="device_keys.has_LF_E2EE_Keys"
           class="my-3">
        <a href="javascript:void(0);"
           v-on:click="deleteKeys()">
          {{ $t("Delete the keys on this browser") }}</a
        >
      </div>

      <div v-if="e2ee_status.forms.length" class="mt-4">
        <span>{{ encrypted_stats }}</span>
      </div>

      <label class="form-label mt-4">
        {{ $t("For your information, your key's fingerprint.") }}
      </label>
      <input
        type="text"
        class="form-control"
        disabled="true"
        :value="public_key_on_server.fingerprint"
      />
    </div>
  </div>
  <CloseBrowser v-if="deleted_keys"/>
</div>
</template>


<script>
import axios from "axios";
import { useI18n } from "vue-i18n";
import { computed, ref, onMounted } from "vue";
import { loadInjections,
         digestDeviceStorage } from '@/composables/e2eeKeySetup.js';
import { deleteDeviceKeys } from '@/modules/e2ee-answers.js';
import { ChevronRightIcon, } from "@zhuowenli/vue-feather-icons";
import KeyOptions from "@/components/keyManager/user/KeyOptions.vue";
import SanityCheck from "@/components/keyManager/user/SanityCheck.vue";
import CloseBrowser from "@/components/keyManager/user/CloseBrowser.vue";

export default {
  name: "E2EEStatus",
  components: {
    KeyOptions,
    SanityCheck,
    ChevronRightIcon,
    CloseBrowser,
  },
  props: {
    initiateRestoreKey: Function,
  },

  setup(props, {emit}) {
    const { t } = useI18n({ useScope: "global" });

    // UI state
    const is_options_available = ref(false)
    const expanded_options = ref(false)
    function collapseOptions() {
      expanded_options.value = false
    }
    const deleted_keys = ref(false)

    // provided by the Loader
    const {
      public_key_on_server,
      e2ee_status,
    } = loadInjections()

    const device_keys = ref({})
    onMounted(async () => {
      device_keys.value = await digestDeviceStorage()
      is_options_available.value = public_key_on_server
    })

    async function deleteKeys() {
      await deleteDeviceKeys()
      deleted_keys.value = true
    }

    // Local-only data. Retreived from storage
    const encrypted_stats = computed(() => {
      let stats = t(
        "You have enabled encryption on %total_forms% forms with a total of %total_answers% answers.",
      );
      let e2ee_status_forms = e2ee_status.forms!==undefined ? e2ee_status.forms : []
      let total_answers = e2ee_status_forms.
                          map((x) => x.answer_count).reduce((x, y) => x + y, 0)
      return stats
        .replace(/%total_forms%/, e2ee_status.forms.length)
        .replace(/%total_answers%/, total_answers);
    })
    return {
      device_keys,
      public_key_on_server,
      encrypted_stats,
      expanded_options,
      collapseOptions,
      emit,
      e2ee_status,
      deleteKeys,
      is_options_available,
      deleted_keys,
    }
  },
}
</script>

<style scoped>
label .feather {
  margin-right: 0.5em;
}
</style>
