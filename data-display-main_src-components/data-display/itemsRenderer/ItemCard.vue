<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>

  <div class="card" :class="expanded ? 'active' : ''">

    <div class="card-header d-flex justify-content-between align-items-baseline">

      <RowControls v-if="data_type=='answer'"
                   :item_id="item.id"
                   :marked="item.marked" />

      <CardTitle v-on:click="expanded = !expanded"
          :key="`title_${item.id}`"
          :field_name="title_field.field_name"
          :field_value="title_field.field_value"
          class="ds-card-title px-2 flex-grow-1 text-truncate" />

      <button v-if="expanded == false" class="card-controls ds-expand-button"
              v-on:click="expanded = !expanded"
              v-bind:aria-label="$t('Expand')">
        <ChevronDownIcon size="1.5x" aria-hidden="true" />
      </button>

      <button v-else class="card-controls ds-expand-button"
              v-on:click="expanded = !expanded"
              v-bind:aria-label="$t('Collapse')">
        <ChevronUpIcon size="1.5x" aria-hidden="true" />
      </button>

    </div>

    <div v-if="expanded == true"
         :class="{'card-data-flex-container': expand_all == true}"
         class="card-body">
      <ul class="list-group list-group-flush">
        <template v-for="(field, index) in field_index" :key="'field'+index">
          <li v-if="field.name != 'marked'"
              :class="{'card-data-flex': expand_all == true}"
              class="list-group-item px-2">

            <span class="fw-bold"
                  :class="{deletedFieldLabel: isFieldDeletedField(field.name)}">
              {{ $t(field.label) }}
            </span>
            <FieldRender :field_name="field.name"
                         :field_value="field.name == 'created' ? item.created : item.data[field.name]"
                         :item_id="item.id" />
          </li>
        </template>
      </ul>
    </div>

  </div>
</template>

<script>
import { useI18n } from "vue-i18n"
import { computed, ref } from "vue";
import { ChevronUpIcon, ChevronDownIcon } from '@zhuowenli/vue-feather-icons'
import { dataDisplayStore } from '@/store.js'
import FieldRender from '@/components/itemsRenderer/FieldRender.vue'
import RowControls from '@/components/itemsRenderer/RowControls.vue'
import CardTitle from '@/components/itemsRenderer/CardTitle.vue'
import { fieldProperties } from '@/composables/fieldProperties.js'

export default {
  name: 'ItemCard',
  components: {
    RowControls,
    FieldRender,
    CardTitle,
    ChevronUpIcon, ChevronDownIcon,
  },
  props: {
    item: Object,
    field_index: Object,
    expand_all: Boolean,
  },

  setup(props) {
    const store = dataDisplayStore()
    const expanded = ref(false)

    const title_value = computed(
      () => (
        store.first_field.name=='created' ?
          props.item.created :
          props.item.data[store.first_field.name]
      )
    );
    const title_field = computed(() => {
      return {
        field_name: store.first_field.name,
        field_value: title_value.value,
      }
    });

    return {
      expanded,
      expand_all: computed(() => props.expand_all),
      isFieldDeletedField: store.isFieldDeletedField,
      data_type: store.data_type,
      title_field,
    }
  },
}
</script>

<style scoped>
.ds-card-title {
  cursor: pointer;
}
.card-header:hover {
  background-color: var(--lf-gray-100);
}
button.ds-expand-button {
  min-height: 36px;
  max-height: 36px;
  min-width: 30px;
}
.deletedFieldLabel {
  color: var(--lf-danger);
}


</style>
