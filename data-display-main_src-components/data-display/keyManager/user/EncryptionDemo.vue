<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div class="card mt-4">
    <div class="card-header">
      <h2>
        {{ $t("Encryption demo") }}
      </h2>
    </div>
    <div class="card-body">
      <div class="form-group">
        <label class="form-label" for="pasted-private-key">
          {{ $t("This is a fictitious form field.") }}
        </label>
        <textarea id="pasted-private-key" class="form-control"
                  v-model="test_input"
                  :placeholder="$t('Add comments here please..')"></textarea>
      </div>
      <div class="mt-3">
        {{ $t("The form encrypts the answer with the public key.") }}
      </div>

      <input
        class="btn btn-sm btn-primary mt-3"
        type="button"
        :value="$t('Encrypt')"
        :disabled="!test_input"
        @click="testEncryption()"
      />

      <div v-if="ciphered_data" class="mt-3">
        <div>{{ $t("The encrypted answer is sent to the server and saved.") }}</div>
        <div class="ds-silence-text mt-2 mb-3">{{ciphered_data}}</div>
        <input
          class="btn btn-sm btn-primary"
          type="button"
          :value="$t('Decrypt')"
          @click="testDecryption()"
        />
      </div>

      <div v-if="test_output" class="mt-3">
        <div>{{ $t("The data is sent to your browser and decrypted with the private key.") }}</div>
        <div class="ds-silence-text mt-2 py-1">{{test_output}}</div>
        <div class="mt-3 fw-bold">{{ $t("Do not lose your private key!") }}</div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";
import { loadInjections, digestDeviceStorage } from '@/composables/e2eeKeySetup.js';
import { keys, keyStorage } from "@/modules/e2ee-answers.js";


export default {
  name: 'TestKey',
  setup() {
    const test_input = ref()
    const test_output = ref()
    const ciphered_data = ref()
    const { editor_key_id } = loadInjections()

    const device_keys = ref({})
    onMounted(async () => {
      device_keys.value = await digestDeviceStorage()
    })

    async function testEncryption() {
      if (test_input.value) {
        // get the public key from sessionStorage
        let pubKey = await keys.getPublic(
          keyStorage.readPublic(editor_key_id, true)
        )
        ciphered_data.value = await pubKey.encrypt(test_input.value)
        test_output.value = ""
      }
    }
    async function testDecryption() {
      if (ciphered_data.value) {
        // get the private key from sessionStorage
        let privKey = await keyStorage.readUnlocked(editor_key_id, null, null)
        test_output.value = await privKey.decrypt(ciphered_data.value)
      }
    }
    return {
      testEncryption,
      testDecryption,
      test_input,
      test_output,
      ciphered_data,
    }
  },
}
</script>

<style scoped>

</style>
