<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div class="rowControls" :class="controlClass">
    <button :class="{'bg-success text-white':marked, 'ds-card-button':is_card}"
            v-on:click="toggleBookmarked"
            :aria-label="$t('Mark for reference')">
      <BookmarkIcon size="1.3x" />
    </button>
    <button v-if="delete_mode"
            class="ds-delete-button"
            :class="{'ds-card-button':is_card==true}"
            :aria-label="$t('Delete answer')"
            v-on:click="initiateDelete()">
      <TrashIcon size="1.3x" />
    </button>
  </div>
</template>

<script>
import axios from 'axios';
import { computed } from "vue";
import { dataDisplayStore } from '@/store.js'
import { BookmarkIcon, TrashIcon } from '@zhuowenli/vue-feather-icons'

export default {
  name: 'RowControls',
  components: {
    BookmarkIcon, TrashIcon,
  },
  props: {
    item_id: Number,
    marked: Boolean,
    is_card: Boolean,
  },

  setup(props) {
    const store = dataDisplayStore()

    const controlClass = computed(() => {
      if (props.is_card==true) {
        return store.ui.delete_mode ? 'card-controls expanded' : 'card-controls notExpanded'
      } else {
        return store.ui.delete_mode ? 'grid-controls expanded' : 'grid-controls notExpanded'
      }
    })

    function toggleBookmarked(event) {
      let item = store.items.find(i => i.id == props.item_id);
      if (item) {
        item.marked = item.marked ? false : true
        axios.post(store.item_endpoint + props.item_id +'/mark')
          .then(function(response) {
            item.marked = response.data.marked
          })
          .catch(e => {
            item.marked = item.marked ? false : true
            console.log(e)
          })
      }
    }

    function initiateDelete() {
      store.delete_item_initiated = props.item_id
    }

    return {
      controlClass,
      toggleBookmarked,
      delete_mode: computed(() => store.ui.delete_mode),
      initiateDelete

    }
  },
}
</script>

<style scoped>
.grid-controls.notExpanded {
  min-width: 38px;
  max-width: 38px;
}
.grid-controls.expanded {
  min-width: 80px;
  max-width: 80px;
}
.card-controls {
  min-height: 30px;
  max-height: 30px;
}
.card-controls.notExpanded {
  min-width: 38px;
  max-width: 38px;
  margin-left: -8px;
}
.card-controls.expanded {
  min-width: 84px;
  max-width: 84px;
  margin-left: -8px;
}
button {
  min-height: 38px;
  max-height: 38px;
  min-width: 38px;
  text-align: center;
}
button.ds-card-button {
  min-height: 30px;
  max-height: 30px;
  min-width: 30px;
  border: 1px solid var(--lf-gray-400);
}
button.ds-delete-button {
  color: var(--lf-danger);
}
</style>
