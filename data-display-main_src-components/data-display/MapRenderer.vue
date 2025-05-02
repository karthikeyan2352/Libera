<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div v-if="show_component" class="mt-3">
    <div v-if="map_selection_options.length == 2" class="fw-bold">
      {{map_selection_options[1].label}}
    </div>
    <select v-else
            v-model="map_selection"
            class="form-control form-select">
      <option v-for="option in map_selection_options" :key="option.name"
              :value="option.name">
          {{ option.label }}
      </option>
    </select>
  </div>
  <div id="map" :style="toggle_map_css" class="mt-3 map" />
</template>

<script>
import axios from 'axios';
import { useI18n } from "vue-i18n"
import { computed, watch, nextTick, ref } from "vue";
import { dataDisplayStore } from '@/store.js'
import * as L from 'leaflet';
import { iconDefault } from '@/modules/leaflet.js'
import 'leaflet/dist/leaflet.css'

/*
This Map compontent setup is somewhat different to the other sibling renderes
because the map's html element needs to be first present in the DOM before
we can create the map.
*/

export default {
  name: 'MapRenderer',
  components: {

  },
  setup() {
    const store = dataDisplayStore()
    const { t } = useI18n({ useScope: "global" })

    const map_selection = ref("default")
    var map_object = null

    L.Marker.prototype.options.icon = iconDefault;

    watch(() => store.display_items_as, () => {
      if (can_display() && !map_object) {
        nextTick(() => {
          initializeMaps()
        })
      }
    }, { immediate: true })
    watch(() => store.itemsToDisplay, () => {
      if (can_display() && !map_object) {
          nextTick(() => {
            initializeMaps()
          })
      } else if (map_object) {
        setMarkers()
      }
    })
    watch(() => store.edition_cnt, () => {
      if (map_object) {
        setMarkers()
      }
    })
    watch(() => store.field_index, () => {
      if (map_object) {
        setMarkers()
      }
    })
    watch(() => map_selection.value, () => {
      setTiles()
      setMarkers()
    })

    function until_map_preference() {
      let poll = resolve => {
        if (store.user_prefs.map !== undefined) resolve();
        else setTimeout(_ => poll(resolve), 400);
      }
      return new Promise(poll);
    }
    async function initializeMaps() {
      await until_map_preference().then(() => {
        map_object = new L.map("map", {
            center: store.user_prefs.map.center,
            zoom: store.user_prefs.map.zoom,
            renderer: new L.canvas(),
            attributionControl: true,
        });
        let tiles = store.all_map_fields.length == 1 ?
                    getMapTiles() :
                    'https://tile.openstreetmap.org/{z}/{x}/{y}.png'
        L.tileLayer(tiles, {
          maxZoom: 19,
          attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
        }).addTo(map_object)

        map_object.setView(store.user_prefs.map.center, store.user_prefs.map.zoom)
        map_object.on('moveend', (e) => {
          saveMapPreferences(e)
        });
        setMarkers()
      })
    }
    function getMapTiles() {
      if (map_selection.value=="default" && store.all_map_fields) {
        return store.getMapTiles(store.all_map_fields[0].name)
      }
      return store.getMapTiles(map_selection.value)
    }
    function saveMapPreferences(e) {
      if (map_selection.value != "default") {
        return
      }
      axios.patch(store.endpoint + '/set-map-preference',
                  JSON.stringify({
                    preference: {
                      center: [
                        map_object.getCenter().lat,
                        map_object.getCenter().lng
                      ],
                      zoom: e.target._zoom
                    },
                  }),
                  {headers: { 'Content-Type': 'application/json' }})
        .catch(function (error) {
          console.log(error);
        });
    }
    function setTiles() {
      map_object.eachLayer((layer) => {
        if (layer instanceof L.tileLayer) {
           layer.remove();
        }
      });
      L.tileLayer(getMapTiles(), {
        maxZoom: 19,
        attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
      }).addTo(map_object)

    }
    function setMarkers() {
      map_object.eachLayer((layer) => {
        if (layer instanceof L.Marker) {
           layer.remove();
        }
      });
      let map_meta = store.meta.map
      store.itemsToDisplay.forEach((item) => {
        for (let [field_name, answer] of Object.entries(item.data)) {
          if (field_name.startsWith("map-")) {
            let field_structure = store.getFieldStructure(field_name)
            if (!(map_selection.value == "default" || map_selection.value == field_name) ||
                answer == "" ||
                (Boolean(field_structure.removed) && !store.include_deleted_fields)
               ) {
              continue
            }
            try {
              let coords = answer.split(',').map(Number)
              let marker = new L.marker(coords)

              if (map_meta.markers[field_name] !== undefined) {
                var popup_html = ""
                let answer_field_names = Object.keys(item.data)
                map_meta.markers[field_name].popup_fields.forEach((popup_field_name) => {
                  try {
                    if (answer_field_names.includes(popup_field_name)) {
                      let label = store.getFieldLabel(popup_field_name)
                      let value = store.getFormattedValue({
                        field_name: popup_field_name,
                        field_value: item.data[popup_field_name]
                      })
                      popup_html = popup_html + '<b>' + label + '</b>: ' + value + '</br>'
                    }
                  }
                  catch {

                  }
                })
                marker.bindPopup(popup_html)
              }
              marker.addTo(map_object);
            }
            catch (e) {
              console.log(e)
            }
          }
        }
      });
    }
    function can_display() {
      if (!store.downloading_items &&
          store.display_items_as=='map' &&
          store.items.length &&
          !store.ui.pdf_builder) {
        return true
      }
      return false
    }
    const toggle_map_css = computed(() => {
      if (can_display()) {
        return "display: block; position: relative; outline-style: none;"
      }
      return "display: none;"
    })

    return {
      show_component: computed(() => can_display()),
      toggle_map_css,
      map_selection,
      map_selection_options: computed(() => [{name:"default", label: t("All fields")},
                                             ...store.all_map_fields])
    }
  }
}
</script>

<style scoped>
.map {
  height: 60vh;
}
</style>
