<template>
  <ion-app>
    <HomePage />
  </ion-app>
</template>

<script setup>
import { IonApp } from '@ionic/vue';
import HomePage from '@/views/HomePage.vue';
</script>

<style>
* {
  font-family: var(--ion-font-family);
  box-sizing: border-box;
  -webkit-font-smoothing: antialiased;
  -webkit-tap-highlight-color: transparent;
}

/* Custom Scrollbar for Monochrome Aesthetic */
::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}
::-webkit-scrollbar-track {
  background: #050505;
}
::-webkit-scrollbar-thumb {
  background: #2a2a2a;
  border-radius: 3px;
}
::-webkit-scrollbar-thumb:hover {
  background: #444444;
}
</style>