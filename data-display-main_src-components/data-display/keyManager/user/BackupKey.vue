<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div>

    <p class="fw-bold">
      2. {{ $t("Backup the key") }}
    </p>

    <p>
      {{ $t("Copy your new and unique private key to the clipboard and save it somewhere safe.") }}
    </p>

    <input
      class="btn btn-sm btn-primary mt-2"
      type="button"
      @click="copyKey()"
      :value="$t('Copy key to clipboard')"
    />

    <div v-if="is_copied" class="mt-4">
      <p>
        {{ $t("Now is the time to save it somewhere safe.") }}
      </p>

      <div class="form-group" >
        <a href="javascript:void(0);"
           @click="show_key = !show_key">
          <span>{{
            $t("Your private key")
          }}</span>
          <ChevronDownIcon
            v-if="show_key==false"
            size="1.5x"
            aria-hidden="true"
          />
          <ChevronUpIcon
            v-else
            class="expand_panel_icon"
            size="1.5x"
            aria-hidden="true"
          />
        </a>

        <textarea
          v-if="show_key"
          id="key-textarea"
          class="form-control mt-2 mb-2"
          disabled
        >{{generated_raw_key}}</textarea>
      </div>

      <Disclaimer
        class="mt-4"
        :label="$t('I have backed up my private key somewhere very safe.')"
        :checked="false"
        @click="emit('keyIsBackedUp')"
      />
    </div>
  </div>
</template>


<script>
import { computed, ref, inject } from "vue";
import Disclaimer from "@/components/keyManager/user/Disclaimer.vue";
import { CheckSquareIcon, ChevronUpIcon, ChevronDownIcon } from "@zhuowenli/vue-feather-icons";

export default {
  name: 'BackupKey',
  components: {
    Disclaimer,
    CheckSquareIcon,
    ChevronUpIcon,
    ChevronDownIcon,
  },
  props: {
    generated_raw_key: String,
    keyIsBackedUp: Function,
  },
  setup(props, {emit}) {

    const is_copied = ref(false)
    const show_key = ref(false)

    function copyKey() {
      navigator.clipboard.writeText(props.generated_raw_key);
      is_copied.value = true;
    }

    return {
      emit,
      show_key,
      copyKey,
      is_copied,
      show_key,
    }
  },
}
</script>
