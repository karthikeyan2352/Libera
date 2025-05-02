<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div tabindex="-1" aria-labelledby="delete-modal-label" aria-hidden="true"
       class="modal fade d-block show"
       @keydown.esc="closeModal()"
       role="dialog">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="delete-modal-label">
            {{ $t("Delete answer")}}
          </h5>
          <button type="button" aria-label="Close"
                  v-on:click="closeModal()">
            <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1" stroke-linecap="round" stroke-linejoin="round" class="feather feather-x" aria-hidden="true"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>
          </button>
        </div>
        <div class="modal-body">
          {{ $t("This cannot be undone!") }}
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-danger"
                  v-on:click="deleteItem()">
            {{ $t("Delete") }}
          </button>
          <button type="button" class="btn btn-secondary"
                  v-on:click="closeModal()">
            {{ $t("Cancel") }}
          </button>
        </div>
      </div>
    </div>
  </div>
  <div class="modal-backdrop fade show"></div>
</template>

<script>
import axios from 'axios';
import { computed } from "vue";
import { dataDisplayStore } from '@/store.js'

export default {
  name: 'DeleteItemModal',
  components: {

  },
  setup() {
    const store = dataDisplayStore()

    const item_id = store.delete_item_initiated

    function deleteItem() {

      function removeFromStore() {
        let item_pos = store.items.findIndex(item => item.id === item_id)
        store.items.splice(item_pos, 1)
        store.meta.total = store.meta.total - 1
        if (store.searched_items) {
          let item_pos = store.searched_items.findIndex(item => item.id === item_id)
          store.searched_items.splice(item_pos, 1)
        }
      }
      let endpoint = store.item_endpoint + item_id + '/delete'
      axios.delete(endpoint)
        .then(response => {
          if (response.data.deleted == true) {
            removeFromStore()
            if (response.data.remove_alert != false) {
              let alert = document.getElementById(response.data.remove_alert.disk_alert)
              if (alert !== null) {
                alert.remove()
                let alert_menu = document.getElementById(response.data.remove_alert.alerts)
                let alerts = alert_menu.querySelectorAll("li");
                if (alerts.length == 0) {
                  alert_menu.remove()
                }
              }
            }
          }
        })
        .catch(e => {
          console.log(e)
        })
        .finally(() => {
          this.closeModal()
        });
    }

    function closeModal() {
      store.delete_item_initiated = null
    }

    return {
      deleteItem, closeModal,
    }
  },
}
</script>

<style scoped>

</style>
