<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div>
    <ul class="ds-e2ee-list">
      <li>
        <a href="javascript:void(0);"
           class="ds-danger"
           v-on:click="emit('collapseOptions'); initiateOverwrite()">
          {{ $t("Overwrite your key pair") }}</a
        >
      </li>
    </ul>
  </div>
</template>

<script>
import { inject, computed } from "vue";

import { ChevronUpIcon, ChevronDownIcon } from "@zhuowenli/vue-feather-icons";

export default {
  name: 'KeyOptions',
  components: {
    ChevronUpIcon,
    ChevronDownIcon,
  },
  props: {
    collapseOptions: Function,
  },
  setup(props, {emit}) {

    return {
      initiateOverwrite: inject('initiateOverwrite'),
      emit,
    }
  },
}
</script>

<style scoped>
ul > li:not(:last-child) {
  margin-bottom: 0.5em;
}
.ds-danger {
  color: var(--lf-danger) !important;
}
</style>
