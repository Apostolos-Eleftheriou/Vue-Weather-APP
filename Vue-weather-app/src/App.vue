<template>
  <div class="flex flex-col min-h-screen font-Roboto bg-gradient-to-r from-weather-primary/95 to-weather-secondary">
    <SiteNavigation />
    <RouterView v-slot="{ Component }">
      <Transition name="page" mode="out-in">
        <component :is="Component" />
      </Transition>
    </RouterView>
    <!-- <router-view v-slot="{ Component }">
      <keep-alive includes="CityList">
        <component :is="Component" />
      </keep-alive>
    </router-view> -->
  </div>
</template>

<script setup>
import { RouterView } from 'vue-router';
import SiteNavigation from './components/SiteNavigation.vue';
</script>

<style>
/* .page-leave-active, */
.page-enter-active {
  transition: 600ms ease all;
}

/* .page-leave-to, */
.page-enter-from {
  opacity: 0;
}
</style>
