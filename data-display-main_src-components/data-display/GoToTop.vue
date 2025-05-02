<!--
This file is part of LiberaForms.

# SPDX-FileCopyrightText: 2024 LiberaForms.org
# SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div ref="target" id="go_to_top">
    <div v-if="showButton" class="ds-goto-top" :style="themeColors"
         @click="goToTop()">
      <ChevronUpIcon size="3x"/>
    </div>
  </div>
</template>

<script>
import { computed, ref, watch } from "vue";
import { useElementVisibility } from '@vueuse/core'
import { ChevronUpIcon } from '@zhuowenli/vue-feather-icons'

export default {
  name: 'GoToTop',
  components: {
    ChevronUpIcon,
  },

  setup(props) {
    const target = ref(null)
    const targetIsVisible = useElementVisibility(target)
    const showButton = ref(false)

    function siteThemeColors() {
      // a hack to avoid sending dynamic generated site theme css
      // on the flask app to this component
      let nav_bar = document.getElementsByClassName("ds-main-navbar")[0]
      let nav_bar_style = window.getComputedStyle(nav_bar)
      let backgound_color = nav_bar_style.getPropertyValue('background-color')
      let nav_brand = document.getElementsByClassName("navbar-brand")[0]
      let nav_brand_style = window.getComputedStyle(nav_brand)
      let color = nav_brand_style.getPropertyValue('color')
      return {
        'color': color,
        'background-color': backgound_color
      }
    }

    watch(() => targetIsVisible.value, () => {
      if (targetIsVisible.value) {
        showButton.value = false
      } else {
        let element = document.getElementById("go_to_top")
        if (element) {
          let rect = element.getBoundingClientRect()
          if (rect.top < 0) {
            showButton.value = true
          }
          //console.log(rect)
        }
      }
    }, {immediate: true})

    function goToTop() {
      window.scrollTo(0,0);
    }

    return {
      target,
      showButton, goToTop,
      themeColors: siteThemeColors(),
    }
  },
}
</script>

<style scoped>
.ds-goto-top {
  z-index: 100;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  position: fixed;
  top:auto;
  bottom:2em;
  right:1.5em;
  left:auto;
}
.ds-goto-top > svg {
  margin-left: 4px;
  margin-top: 3px;
}
</style>
