<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div>
    <p class="fw-bold">3. {{ $t("Restore your key from the backup") }}</p>

    <p class="fst-italic">
      {{ $t("Just to make sure your copied data is correct.") }}
    </p>

    <div class="form-group mt-2">
      <label class="form-label" for="pasted-private-key">
        {{ $t("Your private key") }}
      </label>
      <textarea id="pasted-private-key" class="form-control"
                v-model="test_private_key"></textarea>
    </div>

    <div v-if="with_passphrase" class="mt-3">
      <label class="form-label" for="e2ee_pass">
        {{ $t("Your passphrase") }}
      </label>
      <input type="password" class="form-control"
             v-model="test_passphrase" />
    </div>

    <div v-if="restore_error"
        class="ds-error-message mt-1">
     {{ restore_error }}
    </div>

    <div class="mt-3">
      <button class="btn btn-sm btn-primary mr-2"
              @click="unlockKey()">
        {{ $t("Restore") }}
      </button>
    </div>

  </div>
</template>

<script>
import { useI18n } from "vue-i18n";
import { ref, emit, } from "vue";
import { keys, keyStorage } from "@/modules/e2ee-answers.js";

export default {
  name: 'TestRestore',
  props: {
    with_passphrase: Boolean,
    generated_fingerprint: String,
    restoreTestIsComplete: Function,
  },
  setup(props, {emit}) {
    const { t } = useI18n({ useScope: "global" });

    const test_private_key = ref("")
    const test_passphrase = ref("")
    const restore_error = ref("")

    async function unlockKey() {
      try {
        let recoveredKey = await keys.getPrivate(test_private_key.value, test_passphrase.value);
        if (
          keys.isKey(recoveredKey) === true &&
          recoveredKey.isDecrypted() === true &&
          recoveredKey.fingerprint === props.generated_fingerprint
        ) emit('restoreTestIsComplete')
      }
      catch {
        restore_error.value = t("Key recovery failed")
      }
    }

    return {
      unlockKey,
      test_private_key,
      test_passphrase,
      restore_error,
    }
  },
}
</script>

<style scoped>

</style>
