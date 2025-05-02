<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div>
    <p class="fw-bold">1. {{ $t("Generate the key") }}</p>

    <p v-if="!will_overwrite_keys" class="mt-1">
      {{ $t("To begin using encryption, you must first generate a key pair.") }}
    </p>

    <PassPhrase />

    <div v-if="warn_delete_device_keys" class="mt-3">
      <span>{{$t("The key on this browser will be overwritten.")}}  </span>
    </div>

    <div class="mt-4">
      <input
        class="btn btn-sm btn-primary"
        type="button"
        :disabled="pgp_passphrase_error"
        :value="$t('Generate')"
        @click="generateKeys()"
      />
    </div>
  </div>

</template>

<script>
import { ref, computed, inject, emit, provide, } from "vue";
import { loadInjections } from '@/composables/e2eeKeySetup.js';
import { CheckSquareIcon } from "@zhuowenli/vue-feather-icons";
import PassPhrase from "@/components/keyManager/user/PassPhrase.vue";
import Disclaimer from "@/components/keyManager/user/Disclaimer.vue";

export default {
  name: "GenerateKey",
  components: {
    CheckSquareIcon,
    PassPhrase,
    Disclaimer,
  },
  props: {
    generateUserKeys: Function,
    warn_delete_device_keys: Boolean,
    pgp_passphrase_error: Boolean,
  },
  setup(props, {emit}) {

    const {
      public_key_on_server,
    } = loadInjections()

    const disclaimer_overwrite_keys = ref(!public_key_on_server)
    const will_overwrite_keys = Boolean(public_key_on_server)

    function generateKeys() {
      emit('generateUserKeys')
      //await props.generateUserKeys()
    }

    return {
      disclaimer_overwrite_keys,
      will_overwrite_keys,
      generateKeys,
    }
  }
}
</script>
