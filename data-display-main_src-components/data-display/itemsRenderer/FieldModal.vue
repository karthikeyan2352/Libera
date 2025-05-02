<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div >
    <div class="modal fade show d-block"
         @keydown.esc="closeModal()"
         tabindex="-1"
         role="dialog">
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">
              {{ $t("Answer") }}
            </h5>
            <button type="button"
                    class="close"
                    data-dismiss="modal"
                    aria-label="Close"
                    v-on:click="closeModal()">
              <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1" stroke-linecap="round" stroke-linejoin="round" class="feather feather-x" aria-hidden="true"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>
            </button>
          </div>
          <div class="modal-body">
            <p class="mb-2 fw-bold">{{ $t(data.field_label) }}</p>
            <template v-if="edit_mode">
              <div class="form-group">
                <input v-if="field_type == 'date'"
                       :type="date_field_type(data.field_name)"
                       class="form-control"
                       @keyup.enter="saveField()"
                       v-model="data.field_value" />
                <input v-else-if="field_type == 'text' || field_type == 'number'"
                       :type="field_type"
                       class="form-control"
                       @keyup.enter="saveField()"
                       v-model="data.field_value" />

                <select v-else-if="field_type == 'select'"
                        v-model="data.field_value"
                        class="form-control form-select">
                  <option v-for="option in field_options"
                          :value="option.value">
                      {{ option.label }}
                  </option>
                </select>
                <template v-else-if="field_type == 'checkbox-group'"
                          v-for="option in field_options">
                  <div class="form-check">
                    <input type="checkbox"
                           class="form-check-input"
                           :id="option.value"
                           :value="option.value"
                           v-model="multi_choice" />
                    <label :for="option.value" class="form-check-label">
                      {{ option.label }}
                    </label>
                  </div>
                </template>
                <template v-else-if="field_type == 'radio-group'"
                          v-for="option in field_options">
                  <div class="form-check">
                    <input type="radio"
                           class="form-check-input"
                           name="RadioGroup"
                           :id="option.value"
                           :value="option.value"
                           v-model="data.field_value" />
                    <label :for="option.value" class="form-check-label">
                     {{ option.label }}
                    </label>
                  </div>
                </template>
                <textarea v-else-if="field_type == 'textarea'"
                          class="form-control"
                          rows="6"
                          @keyup.enter="saveField()"
                          v-model="data.field_value">
                </textarea>
                <template v-else-if="field_type == 'map'">
                  <div id="edit_map" class="map" />
                  <input type="text" class="mt-3 form-control"
                         v-model="data.field_value"
                         placeholder='39.54099078086269, 3.3349138617888223' />
                  <div v-if="invalid_field_value" class="mt-1 lf-error">
                    {{ $t("Not a valid value.") }}
                  </div>
                </template>
              </div>
            </template>

            <template v-else>
              <p v-if="!data.field_value && !data.field_name.startsWith('map')"
                      class="mb-0 ds-empty-field">
                {{ $t("Empty") }}
              </p>

              <div v-if="data.field_name.startsWith('checkbox')"
                   v-html="formatted_field_value()">
              </div>

              <div v-else-if="data.field_name.startsWith('map')" id="map" class="map" />

              <div v-else-if="data.field_value">{{ formatted_field_value() }}</div>

            </template>
          </div>
          <div class="modal-footer">
            <button v-if="!edit_mode && data.field_value && (
                          field_type == 'text' ||
                          field_type == 'date' ||
                          field_type == 'number' ||
                          field_type == 'map' ||
                          field_type == 'textarea')"
                    v-on:click="copyToClipboard(data.field_value)"
                    type="button"
                    class="btn btn-outline-primary">
              {{ $t("Copy") }}
            </button>
            <button v-if="can_edit && !edit_mode && !is_deleted_field"
                    type="button"
                    class="btn btn-outline-primary"
                    data-dismiss="modal"
                    v-on:click="edit_mode = true">
              {{ $t("Edit") }}
            </button>
            <button v-if="edit_mode"
                      type="button"
                      class="btn btn-primary"
                      v-on:click="saveField()">
                {{ $t("Save") }}
            </button>
            <button type="button"
                    class="btn btn-secondary"
                    v-on:click="closeModal()">
                {{ edit_mode ? $t("Cancel") : $t("Close") }}
            </button>
          </div>
        </div>
      </div>
    </div>
    <div class="modal-backdrop fade show"></div>
  </div>
</template>

<script>
import axios from 'axios';
import useClipboard from 'vue-clipboard3'
import { computed, ref, watch, onMounted, nextTick } from "vue";
import { dataDisplayStore } from '@/store.js'
//import { XIcon } from '@zhuowenli/vue-feather-icons'
import * as L from 'leaflet';
import { iconDefault } from '@/modules/leaflet.js'
import 'leaflet/dist/leaflet.css'
import { centerPoint2coords } from '@/modules/map_utils.js'
import { keys, keyStorage } from "@/modules/e2ee-answers.js";

export default {
  name: 'FieldModal',
  components: {
  },
  props: {
    data: Object,
  },
  emits: ['closeFieldModal'],

  setup(props, { emit }) {

    const store = dataDisplayStore()
    const can_edit = store.can_edit
    const { toClipboard } = useClipboard()

    const edit_mode = ref(false)
    const field_options = ref([])
    const multi_choice = ref([])

    let structure = store.getFieldStructure(props.data.field_name)
    let field_structure = structure !== undefined ? structure : {"type": null}
    const field_type = ref(field_structure.type)
    const invalid_field_value = ref(false)

    var map_object = null

    watch(() => edit_mode.value, () => {
      if (edit_mode.value == true) {
        createFormData()
      }
    })
    watch(() => props.data.field_value, () => {
      invalid_field_value.value = false
    })
    onMounted(()=> {
      if (field_type.value == 'map') {
        L.Marker.prototype.options.icon = iconDefault;
        initiateMap()
      }
    });

    function initiateMap() {
      let centerPoint
      var map_coords
      var zoom
      var element_id = "map"

      if (edit_mode.value == true) {
        element_id = "edit_map"
        map_coords = [map_object.getCenter().lat, map_object.getCenter().lng]
        zoom = map_object.getZoom()
      }
      else {
        zoom = field_structure.zoom
        if (props.data.field_value) {
          map_coords = centerPoint2coords(props.data.field_value)
          if (!map_coords.length) {
            map_coords = centerPoint2coords(field_structure.centerPoint)
          }
        }
        else {
          map_coords = centerPoint2coords(field_structure.centerPoint)
        }
      }
      let map_options = {
          center: map_coords,
          zoom: zoom,
          renderer: L.canvas(),
          attributionControl: true,
      }
      map_object = new L.map(element_id, map_options);

      let tiles = store.getMapTiles(props.data.field_name)
      L.tileLayer(tiles, {
        maxZoom: 19,
        attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
      }).addTo(map_object);

      map_object.setView(map_coords, zoom);
      if (props.data.field_value) {
        updateMapMarker()
      }
      if (edit_mode.value == true) {
        map_object.on('click', (e) => {
          props.data.field_value = "{%x%}, {%y%}".replace(/{%x%}/, e.latlng.lat)
                                                 .replace(/{%y%}/, e.latlng.lng)
          updateMapMarker()
        });
      }
    }
    function updateMapMarker() {
      map_object.eachLayer((layer) => {
        if (layer instanceof L.Marker) {
           layer.remove();
        }
      })
      let coords = centerPoint2coords(props.data.field_value)
      if (coords.length) {
        L.marker(coords).addTo(map_object)
      }
    }

    function createFormData() {
      field_options.value = []
      multi_choice.value = []
      if (field_type.value == 'select' ||
          field_type.value == 'checkbox-group' ||
          field_type.value == 'radio-group') {
        if (props.data.field_value !== undefined) { // TODO should be !=="" ?
          multi_choice.value = props.data.field_value.split(', ')
        }
        field_structure.values.forEach((value) => {
          field_options.value.push({
            value: value.value,
            label: value.label
          })
        });
      }
      if (field_type.value == 'map') {
        nextTick(() => {
          initiateMap()
        })
      }
    }
    function date_field_type(field_name) {
      let field_struct = store.getFieldStructure(field_name)
      if (field_struct && field_struct['subtype'] != undefined) {
        return field_struct['subtype']
      }
      return 'date'
    }
    function formatted_field_value() {
      if (!props.data.field_value) {
        return props.data.field_value
      }
      if (props.data.field_name.startsWith('checkbox')) {
        var values = props.data.field_value.split(', ')
        var html = '<ul>'
        values.forEach((value) => {
          var label = store.getOptionLabel({'field_name': props.data.field_name,
                                            'option_value': value})
          html = html + '<li style="margin-left: -1.3em;">' + label + '</li>'
        });
        html = html + '</ul>'
        return html
      }
      if (props.data.field_name.startsWith('radio-group') ||
          props.data.field_name.startsWith('select')){
        var label = store.getOptionLabel({'field_name': props.data.field_name,
                                          'option_value': props.data.field_value})
        if (label) {
          return label
        }
      }
      return props.data.field_value
    }

    async function saveField() {
      let item_id = props.data.item_id
      var item = store.items.find(item => item.id === item_id)
      var field_value = ""
      if (item !== undefined) {
        if (field_type.value == "checkbox-group") {
          if (multi_choice.value.length > 0 && multi_choice.value[0].trim() === "") {
            // multi_choice may contain "" in as the first item of the array. why?
            multi_choice.value = multi_choice.value.slice(1)
          }
          field_value = multi_choice.value.join(', ')
        }
        else if (field_type.value == "number") {
          field_value = props.data.field_value !== undefined ? props.data.field_value : ""
        }
        else if (field_type.value == "map") {
          field_value = props.data.field_value !== undefined ? props.data.field_value : ""
          if (!centerPoint2coords(field_value).length) {
            invalid_field_value.value = true
            return
          }
        }
        else {
          field_value = props.data.field_value !== undefined ? props.data.field_value.trim() : ""
        }
        let field_name = props.data.field_name
        var initial_value = item.data[field_name]
        var save_edition = true
        if (field_name.startsWith('checkbox') && field_value === "") {
          // answer.data dict must not include this key with an empty value
          // database expects dict without this key
          delete item.data[field_name];
        }
        else {
          if (item.data[field_name] == field_value) {
            save_edition = false
          }
          else {
            item.data[field_name] = field_value
          }
        }
        if (save_edition) {
          let json_data;
          if (store.is_e2ee) {
            try {
              // get form's public key from sessionStorage
              let pubKey = await keys.getPublic(
                keyStorage.readPublic(store.e2ee_form_key_id, true)
              )
              let ciphered_data = await pubKey.encrypt(JSON.stringify(item.data))
              json_data = JSON.stringify({
                item_data: {'e2ee-answer': ciphered_data}
              })
            }
            catch(e) {
              console.log(e)
              item.data[field_name] = initial_value
              closeModal()
              return
            }
          }
          else {
            json_data = JSON.stringify({item_data: item.data});
          }
          axios.patch(
              `${store.item_endpoint}${item_id}/save`,
              json_data,
              { headers: {'Content-Type': 'application/json'} }
            )
            .then(response => {
              if (response.data.saved == true) {
                ++store.edition_cnt
              }
            })
            .catch(e => {
              console.log(e)
              item.data[field_name] = initial_value
            });
        }
      }
      closeModal()
    }
    function closeModal() {
      edit_mode.value = false
      field_structure = {}
      emit('closeFieldModal')
    }
    const copyToClipboard = async (text) => {
      try {
        await toClipboard(text)
      } catch (e) {
        console.error(e)
      }
      closeModal()
    }

    return {
      can_edit, edit_mode, formatted_field_value,
      field_type, field_options, multi_choice, date_field_type,
      closeModal, saveField, copyToClipboard,
      invalid_field_value,
      is_deleted_field: computed(() => store.isFieldDeletedField(props.data.field_name)),
    }
  },
}
</script>

<style scoped>
.map {
  height: 40vh;
}
.lf-error {
  color: var(--lf-danger);
}
</style>
