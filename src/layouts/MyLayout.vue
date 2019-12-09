<template>
  <q-layout view="lHh Lpr lFf">
    <q-header class="text-white" style="background-color: white" align="center">
      <q-toolbar align="center">
        <div style="width: 100vw">
          <q-avatar style="width: auto">
            <img src="statics/FUNCOIN.png">
          </q-avatar>
          </div>
      </q-toolbar>
    </q-header>
        <q-separator inset/>
<!--
        <q-bar dense class="bg-amber text-grey-8">
          <q-icon name="fas fa-signal" />
          <div>mobi-net</div>
          <div>4G</div>
          <q-icon name="fas fa-wifi" />
          <q-space />
          <q-icon name="near_me" />
          <q-icon name="fas fa-battery-full" />
        </q-bar> -->
    <q-page-container>
      <router-view />
      <!-- <expand-ball class="ball-pos">
          <span slot="menu" @click="clicked('M1')">M1</span>
          <span slot="menu" @click="clicked('M2')">M2</span>
      </expand-ball> -->
    </q-page-container>
  </q-layout>
</template>

<script>
import { openURL } from 'quasar'

export default {
  name: 'MyLayout',
  data () {
    return {
      leftDrawerOpen: this.$q.platform.is.desktop
    }
  },
  methods: {
    openURL
    // clicked (str) {
    //   alert(str + ' is just clicked!!')
    // }
  }
}
</script>

<style>
/* .ball-pos {
    bottom: 200px;
    right: 200px;
} */
</style>
