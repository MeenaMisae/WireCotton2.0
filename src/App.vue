<template>
  <div class="h-screen overflow-auto">
    <div class="flex">
      <SideMenu />
      <div class="lg:pr-10 lg:pl-5 lg:ml-8 w-[100%] px-6">
        <NavBar />
        <router-view v-slot="{ Component, route }">
          <Transition name="fade" mode="out-in">
            <div :key="route.path" class="my-4 lg:my-0">
              <component :is="Component"></component>
            </div>
          </Transition>
        </router-view>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import SideMenu from './components/SideMenu.vue'
import NavBar from './components/NavBar.vue'
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
