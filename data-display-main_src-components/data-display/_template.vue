<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
Hello world
</template>

<script>
import { computed } from "vue";
import { dataDisplayStore } from '@/store.js'

export default {
  name: 'Template',
  components: {

  },

  setup() {
    const store = dataDisplayStore()


    return {

    }
  },
}
</script>

<style scoped>

</style>
