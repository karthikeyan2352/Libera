<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div>
    <ul class="ds-e2ee-list">
      <li>
        <SanityCheck
          :checked="true"
          :ok_label="$t('You restored your keys correctly.')"
        />
      </li>
    </ul>

    <p class="fw-bold mt-2">4. {{ $t("Save the configuration") }}</p>

    <div id="disclaimers" class="mt-3 checkbox-group">
      <Disclaimer
        v-if="is_overwriting && would_lose_data"
        :label="$t('I know that I am going to lose all my encrypted answers, forever.')"
        v-model:checked="disclaimer_overwrite_key"
      />
      <Disclaimer
        v-if="key_has_passphrase"
        :label="$t('I will not forget the passphrase, ever.')"
        v-model:checked="disclaimer_remember_passphrase"
      />
      <Disclaimer
        :label="$t('I have backed up my private key somewhere very safe.')"
        v-model:checked="disclaimer_have_backed_up"
      />
      <Disclaimer
        v-if="key_has_passphrase"
        :label="$t('I know I will permanently lose the answers to my encrypted forms if I forget the passphrase or lose my private key.')"
        v-model:checked="disclaimer_dataloss"
      />
      <Disclaimer
        v-else
        :label="$t('I know I will permanently lose the answers to my encrypted forms if I lose my private key.')"
        v-model:checked="disclaimer_dataloss"
      />
    </div>

    <button
      type="button"
      class="btn btn-primary mt-3"
      :disabled="!is_ready_to_post"
      @click="emit('post_key')"
    >
      {{ $t("Save") }}
    </button>

    <div v-if="post_errors" class="ds-error-message">
      {{ post_errors }}
    </div>
  </div>
</template>

<script>
import { useI18n } from "vue-i18n";
import { ref, watch, inject } from "vue";
import Disclaimer from "@/components/keyManager/user/Disclaimer.vue";
import SanityCheck from "@/components/keyManager/user/SanityCheck.vue";


export default {
  name: "SaveKey",
  components: {
    Disclaimer,
    SanityCheck,
  },
  props: {
    post_errors: String,
    would_lose_data: Boolean,
    key_has_passphrase: Boolean,
    is_overwriting: Boolean,
    post_key: Function,
  },

  setup(props, {emit}) {
    //const { t } = useI18n({ useScope: "global" });

    // Disclaimers
    const disclaimer_overwrite_key = ref(
      !(props.is_overwriting && Boolean(props.would_lose_data))
    );
    const disclaimer_remember_passphrase = ref(!props.key_has_passphrase);
    const disclaimer_have_backed_up = ref(false);
    const disclaimer_dataloss = ref(false);
    const is_ready_to_post = ref(false)

    watch(
      [
        disclaimer_overwrite_key,
        disclaimer_remember_passphrase,
        disclaimer_have_backed_up,
        disclaimer_dataloss,
      ],
      () => {
        is_ready_to_post.value =
          disclaimer_overwrite_key.value &&
          disclaimer_remember_passphrase.value &&
          disclaimer_have_backed_up.value &&
          disclaimer_dataloss.value;
      },
    );

    return {
      disclaimer_overwrite_key,
      disclaimer_remember_passphrase,
      disclaimer_have_backed_up,
      disclaimer_dataloss,
      is_ready_to_post,
      emit,
    }
  }
}
</script>
