<template>
  <q-layout view="lHh Lpr lFf" class="bg-grey-1" style="font-family: 'Inter', sans-serif;">
    
    <!-- Topbar Component -->
    <Topbar @toggle-sidebar="leftDrawerOpen = !leftDrawerOpen" />

    <!-- Sidebar Component -->
    <Sidebar v-model="leftDrawerOpen" />

    <!-- Page Content -->
    <q-page-container>
      <router-view />
    </q-page-container>

  </q-layout>
</template>

<script setup>
import { ref } from 'vue'
import Topbar from 'src/components/dashboard/Topbar.vue'
import Sidebar from 'src/components/dashboard/Sidebar.vue'

const leftDrawerOpen = ref(true)
</script>

<style>
/* Base styling for the dashboard theme */
body {
  background-color: #f4f6f8;
}
</style>
