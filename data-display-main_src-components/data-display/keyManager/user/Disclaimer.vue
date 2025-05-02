<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div class="form-check">
    <label class="form-label form-check-label">
      <input type="checkbox" class="form-check-input"
             v-model="checked"
             :disabled="disabled" />
      <span>{{ label }}</span>
    </label>
  </div>
</template>

<script setup>
//import { } from "vue";
const checked = defineModel("checked", { default: false });
const disabled = defineModel("disabled", { default: false });
defineProps({ label: String });
</script>
