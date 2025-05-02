<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
<div>

  <div v-if="!public_key_on_server">
    <div tabindex="-1" aria-labelledby="e2ee-redirect-modal-label" aria-hidden="true"
         class="modal fade d-block show"
         role="dialog">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="e2ee-redirect-modal-label">
              {{ $t("Encryption keys are required") }}
            </h5>
            <button
                  type="button"
                  class="close"
                  data-dismiss="modal"
                  aria-label="Close"
                  v-on:click="cancel()">
              <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1" stroke-linecap="round" stroke-linejoin="round" class="feather feather-x" aria-hidden="true"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>
            </button>
          </div>
          <div class="modal-body">
            <p class="mt-3">
            <a class="ds-link-contain-icon" href="/user/e2ee-public-key">
              {{ $t("Please configure your personal keys to continue") }}
              <ChevronRightIcon width="20" height="20" aria-hidden="true"/></a>
            </p>
          </div>
          <div class="modal-footer">
            <button
                  type="button"
                  class="btn btn-secondary"
                  data-dismiss="modal"
                  aria-label="{{ $t('Cancel')}}"
                  v-on:click="cancel()">
              {{ $t("Cancel")}}
            </button>
          </div>
        </div>
      </div>
    </div>
    <div class="modal-backdrop fade show"></div>
  </div>

  <div v-else-if="form_key_id && need_ciphered_key">
    <div tabindex="-1" aria-labelledby="e2ee-redirect-modal-label" aria-hidden="true"
         class="modal fade d-block show"
         role="dialog">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="e2ee-redirect-modal-label">
              {{ $t("Encryption") }}
            </h5>
            <button
                  type="button"
                  class="close"
                  data-dismiss="modal"
                  aria-label="Close"
                  v-on:click="cancel()">
              <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1" stroke-linecap="round" stroke-linejoin="round" class="feather feather-x" aria-hidden="true"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>
            </button>
          </div>
          <div class="modal-body">
            <p class="mt-2 mb-4">
              {{ $t("Your personal keys are ready but access to these encrypted answers is not enabled.") }}
            </p>
            <p class="mb-2">
              {{ $t("Please speak with %email%").replace(/%email%/, need_ciphered_key) }}
            </p>
          </div>
          <div class="modal-footer">
            <a @click="cancel()"
               class="btn btn-secondary">
              {{ $t("Close") }}
            </a>
          </div>
        </div>
      </div>
    </div>
    <div class="modal-backdrop fade show"></div>
  </div>

  <div v-else-if="need_private_key || need_passphrase">
    <div tabindex="-1" aria-labelledby="e2ee-restore-modal-label" aria-hidden="true"
         class="modal fade d-block show"
         role="dialog">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="e2ee-restore-modal-label">
              {{ $t("Your key is required") }}
            </h5>
            <button
                  type="button"
                  class="close"
                  data-dismiss="modal"
                  aria-label="Close"
                  v-on:click="cancel()">
              <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1" stroke-linecap="round" stroke-linejoin="round" class="feather feather-x" aria-hidden="true"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>
            </button>
          </div>
          <div class="modal-body">
                <div class="form-group">
                  <div v-if="need_private_key">
                    <p>
                      {{ $t("Copy the key to the clipboard and then restore.") }}
                    </p>
                    <div class="form-group">
                      <label class="form-label" for="pasted-private-key">
                        {{ $t("Optionally, paste here to see your key.") }}
                      </label>
                      <textarea id="pasted-private-key" class="form-control"
                                @focus="restore_error=''"></textarea>
                    </div>
                    <div v-if="restore_error"
                         class="ds-error-message mt-1">
                      {{ restore_error }}
                    </div>
                  </div>

                  <div v-if="need_passphrase">
                    <label class="form-label" for="e2ee_pass">
                      {{ $t("Your key's passphrase is required") }}
                    </label>
                    <input id="e2ee_pass" type="password" class="form-control"
                           v-model="pgp_passphrase"
                           @keydown.enter="local_restore()"
                    />
                    <div v-if="restore_error"
                        class="ds-error-message mt-1">
                     {{ restore_error }}
                    </div>
                  </div>
                </div>
          </div>
          <div class="modal-footer justify-content-between">
            <div v-if="need_private_key" class="form-check">
             <label class="form-label form-check-label">
               <input type="checkbox" class="form-check-input"
                      v-model="remember_private_key"  />
               <span>
                 {{ $t("Remember the key") }}
               </span>
             </label>
            </div>
            <div v-if="need_passphrase && (has_local_locked_key || remember_private_key)"
                 class="form-check">
             <label class="form-label form-check-label">
               <input type="checkbox" class="form-check-input"
                      v-model="remember_passphrase" />
               <span>
                 {{ $t("Remember the passphrase") }}
               </span>
             </label>
            </div>
            <div v-else />

            <input
              class="btn btn-primary"
              type="button"
              :value="need_private_key ? $t('Restore') : $t('Unlock')"
              @click="local_restore()"
            />
          </div>
        </div>
      </div>
    </div>
    <div class="modal-backdrop fade show"></div>
  </div>
</div>
</template>

<script>
import { useI18n } from "vue-i18n";
import { ref, computed, emit, onMounted, } from "vue";
import { loadInjections,
         digestDeviceStorage } from '@/composables/e2eeKeySetup.js';
import { keys, deleteDeviceKeys } from "@/modules/e2ee-answers.js";
import { ChevronRightIcon } from "@zhuowenli/vue-feather-icons";

export default {
  name: "RestoreKey",
  components: {
    ChevronRightIcon,
  },
  props: {
    //decrypting_answers: [Boolean, undefined],
    keyIsRestored: Function,
    cancelRestore: Function,
    //is_canceled: Boolean,
  },
  setup(props, {emit}) {
    const { t } = useI18n({ useScope: "global" });

    const {
      public_key_on_server,
      editor_key_id,
      form_key_id,
      e2ee_status,
    } = loadInjections()

    const provided_private_key = ref("");
    const pgp_passphrase = ref("")
    const restore_error = ref("");
    const lockedKey = ref()
    const my_e2ee_state = ref()
    const remember_private_key = ref(false)
    const remember_passphrase = ref(false)
    var privKey;

    const need_ciphered_key = ref()

    //console.log(e2ee_status)
    if (public_key_on_server && form_key_id) {
      // The user has a public key on server
      // The form key may not have been shared with this form_user yet
      // check for the ciphered_key
      function needsCipheredKey() {
        let author = e2ee_status.editors.find(el => el.form_role == 'author');
        need_ciphered_key.value = author.email
      }
      try {
        let e2ee_form_user = e2ee_status.editors.find(
          user => (
            user.e2ee_public_key &&
            user.e2ee_public_key.fingerprint == public_key_on_server.fingerprint
          )
        )
        if (
          e2ee_form_user.e2ee_ciphered_key_backup.form_key_fingerprint !=
          e2ee_status.is_e2ee_enabled.fingerprint
        ) {
          needsCipheredKey()
        }
      }
      catch(e) {
        needsCipheredKey()
        //console.log('caught',e)
      }
    }

    const has_local_locked_key = ref(false)
    onMounted(async () => {
      const device_keys = await digestDeviceStorage();
      if (device_keys.has_key && !device_keys.user_key_remote_match) {
        await deleteDeviceKeys(editor_key_id)
        my_e2ee_state.value = 'error'
      }
      else {
        my_e2ee_state.value = device_keys.e2ee_state;
        if (device_keys.has_local_key) {
          lockedKey.value = device_keys.lockedKey
          has_local_locked_key.value = true
        }
      }

    })
    const need_private_key = computed(
      () => (
        (my_e2ee_state.value == 'locked' || my_e2ee_state.value == 'error') &&
        !lockedKey.value
      )
    )
    const need_passphrase = computed(
      () => my_e2ee_state.value == 'need_passphrase'
    )

    async function local_restore() {
      // TODO: warn when replacing. replacing what??

      if (need_private_key.value) {
        // get the private key from the user
        try {
          provided_private_key.value = await navigator.clipboard.readText()
        }
        catch {
          let private_key = document.getElementById("pasted-private-key").value
          if (private_key === "") {
            restore_error.value = t("Sorry, we cannot read your clipboard. Please paste the key above.")
            return
          }
          provided_private_key.value = private_key;
        }
        try {
          let recoveredKey = await keys.getPrivate(provided_private_key.value);
          lockedKey.value = await recoveredKey.export()
        }
        catch {
          restore_error.value = t("That was not a valid key.")
          return
        }
      }
      if (lockedKey.value) {
        try {
          privKey = await keys.getPrivate(lockedKey.value, pgp_passphrase.value)
        }
        catch {
          restore_error.value = t("Incorrect passphrase.")
          return
        }
        // TODO: match fingerprints?
      }
      if (!keys.isKey(privKey))
        return

      my_e2ee_state.value = privKey.isDecrypted() ? 'unlocked' : 'need_passphrase'

      async function persist_key() {
        // save unlocked to sessionStorage
        await privKey.persist(null, true)

        if (remember_passphrase.value && pgp_passphrase.value) {
          // save unlocked to localStorage
          await privKey.persist(null, false)
        }
        else if (remember_private_key.value && pgp_passphrase.value ) {
          // save locked to localStorage
          await privKey.persist(pgp_passphrase.value, false)
        }
        else if (remember_private_key.value) {
          // save unlocked to localStorage
          await privKey.persist(null, false)
        }
        emit('keyIsRestored', pgp_passphrase.value)
      }
      if (my_e2ee_state.value == 'unlocked')
        persist_key()
    }

    function cancel() {
      emit('cancelRestore')
    }

    return {
      local_restore,
      restore_error,
      provided_private_key,
      pgp_passphrase,
      need_private_key,
      need_passphrase,
      provided_private_key,
      remember_private_key,
      remember_passphrase,
      my_e2ee_state,
      public_key_on_server,
      need_ciphered_key,
      form_key_id,
      cancel,
      has_local_locked_key,
    }
  }
}
</script>
