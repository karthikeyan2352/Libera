<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <li>
    <label class="form-label form-check-label">
      <CheckSquareIcon
        v-if="checked"
        class="ok-msg"
        width="16"
        height="16"
        stroke-width="2"
        aria-hidden="true"
      />
      <XSquareIcon
        v-else
        class="ko-msg"
        width="16"
        height="16"
        stroke-width="2"
        aria-hidden="true"
      />
      <a v-if="href"
         :class="text_decorator"
         :href="href">{{ computed_label }}</a>
      <span v-else
            :class="text_decorator">
        {{ computed_label }}
      </span>
    </label>
  </li>
</template>

<script setup>
import { computed } from "vue";
import { CheckSquareIcon, XSquareIcon } from "@zhuowenli/vue-feather-icons";

const props = defineProps({
  ok_label: String,
  ko_label: String,
  href: String,
  text_decorator: Object,
  checked: [Object, String, Boolean, null],
});
const computed_label = computed(() =>
  props.checked ? props.ok_label : props.ko_label || props.ok_label,
);
</script>

<style scoped>
label .feather {
  margin-right: 0.5em;
}
</style>
