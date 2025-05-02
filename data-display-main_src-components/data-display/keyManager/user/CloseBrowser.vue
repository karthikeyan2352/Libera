<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2025 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div>
    <div tabindex="-1" aria-labelledby="close-browser-modal-label" aria-hidden="true"
         class="modal fade d-block show"
         role="dialog">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-body  text-center">
            <p class="mt-4">
              <CheckSquareIcon
                stroke="var(--lf-success)"
                width="16"
                height="16"
                stroke-width="2"
                aria-hidden="true"
              />
              {{ $t("All local and session keys have been deleted.") }}
            </p>
            <p class="mt-1 mb-3">
              {{ $t("To ensure any remaining keys are deleted...") }}
            </p>
            <AlertTriangleIcon
              size="2.5x"
              stroke-width="1.75"
              stroke="var(--lf-danger)"
              aria-hidden="true"/>
            <p class="mt-3 mb-4">
              {{ $t("Close your browser window now!") }}
            </p>
          </div>
        </div>
      </div>
    </div>
    <div class="modal-backdrop fade show"></div>
  </div>
</template>

<script>
import { AlertTriangleIcon, CheckSquareIcon } from "@zhuowenli/vue-feather-icons";

export default {
  name: 'CloseBrowser',
  components: {
    AlertTriangleIcon,
    CheckSquareIcon,
  },
  setup() {
    return {

    }
  },
}
</script>

<style scoped>

</style>
