<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div class="mt-2">
    <a href="javascript:void(0);" v-on:click="togglePassphrase()">
      <span>{{
        $t("Recommended key protection")
      }}</span>
      <ChevronDownIcon
        v-if="expanded_passphrase==false"
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
    <p v-if="expanded_passphrase" class="mt-3 mb-0">{{
      $t("Optionally, protect your key with a passphrase/password.")
    }}</p>
    <div v-if="expanded_passphrase" class="grid mt-3 mb-1">
      <div class="g-col-12 g-col-md-6">
        <label class="form-label">
          {{ $t("Key passphrase") }}
        </label>
        <input
          type="password"
          class="form-control"
          v-model="pgp_passphrase"
        />
      </div>
      <div class="g-col-12 g-col-md-6">
        <label class="form-label">
          {{ $t("Repeat the passphrase") }}
        </label>
        <input
          type="password"
          class="form-control"
          v-model="pgp_passphrase_repeat"
        />
      </div>
    </div>
    <span v-if="pgp_passphrase_error" class="ds-error-message">
      {{ $t("The passphrases to not match.") }}
    </span>
    <p v-if="expanded_passphrase" class="mt-4 mb-0">{{
      $t("Note that the passphrase is permanent and cannot be changed.")
    }}</p>
  </div>

</template>

<script>
import { ChevronUpIcon, ChevronDownIcon } from "@zhuowenli/vue-feather-icons";
import { computed, ref, inject, watch } from "vue";

export default {
  name: 'PassPhrase',
  components: {
    ChevronUpIcon,
    ChevronDownIcon,
  },
  setup() {
    const expanded_passphrase = ref(false);
    const pgp_passphrase = ref("");
    const pgp_passphrase_repeat = ref("");
    const pgp_effective_passphrase = inject("pgp_effective_passphrase")
    const pgp_passphrase_error = inject("pgp_passphrase_error")

    watch(
      [
        pgp_passphrase,
        pgp_passphrase_repeat
      ], () =>
      {
        pgp_effective_passphrase.value =
          pgp_passphrase.value == pgp_passphrase_repeat.value
          ? pgp_passphrase.value : undefined

        pgp_passphrase_error.value =
          pgp_effective_passphrase.value !== pgp_passphrase_repeat.value
      },
      {immediate: true}
    )

    function togglePassphrase() {
      if (!pgp_passphrase_error.value) {
        expanded_passphrase.value = !expanded_passphrase.value
      }
    }


    return {
      togglePassphrase,
      expanded_passphrase,
      pgp_passphrase,
      pgp_passphrase_repeat,
      pgp_passphrase_error,

    }
  },
}
</script>

<style scoped>

</style>
