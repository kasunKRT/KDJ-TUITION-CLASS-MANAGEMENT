<template>
  <q-drawer
    show-if-above
    v-model="drawerOpen"
    side="left"
    bordered
    class="bg-white"
    :width="280"
  >
    <!-- Logo Section -->
    <div class="q-pa-md row items-center justify-center cursor-pointer" style="height: 70px; border-bottom: 1px solid #f0f0f0;">
      <div class="text-weight-bolder text-h5 text-dark" style="letter-spacing: 0.5px;">
        <q-icon name="school" color="primary" class="q-mr-sm" size="md" /> 
        Class<span class="text-primary">Master</span>
      </div>
    </div>

    <!-- Navigation Menu -->
    <q-scroll-area style="height: calc(100% - 70px);">
      <q-list class="q-px-md q-py-md">
        
        <div class="text-caption text-weight-bold text-grey-5 q-ml-sm q-mb-sm text-uppercase" style="letter-spacing: 1px;">Main Menu</div>
        
        <q-item 
          v-for="link in mainLinks" 
          :key="link.title"
          clickable 
          v-ripple
          :to="link.route"
          exact
          class="q-mb-sm menu-item"
          active-class="menu-active"
        >
          <q-item-section avatar style="min-width: 40px;">
            <q-icon :name="link.icon" size="sm" class="menu-icon" />
          </q-item-section>
          <q-item-section class="text-weight-bold menu-text">
            {{ link.title }}
          </q-item-section>
        </q-item>

        <q-separator class="q-my-md bg-grey-2" />
        
        <div class="text-caption text-weight-bold text-grey-5 q-ml-sm q-mb-sm text-uppercase" style="letter-spacing: 1px;">Administration</div>

        <q-item 
          v-for="link in adminLinks" 
          :key="link.title"
          clickable 
          v-ripple
          :to="link.route"
          class="q-mb-sm menu-item"
          active-class="menu-active"
        >
          <q-item-section avatar style="min-width: 40px;">
            <q-icon :name="link.icon" size="sm" class="menu-icon" />
          </q-item-section>
          <q-item-section class="text-weight-bold menu-text">
            {{ link.title }}
          </q-item-section>
        </q-item>

      </q-list>
    </q-scroll-area>
  </q-drawer>
</template>

<script setup>
import { computed } from 'vue'

defineOptions({
  name: 'DashboardSidebar'
})

const props = defineProps({
  modelValue: {
    type: Boolean,
    default: true
  }
})

const emit = defineEmits(['update:modelValue'])

const drawerOpen = computed({
  get: () => props.modelValue,
  set: (val) => emit('update:modelValue', val)
})

const mainLinks = [
  { title: 'Dashboard', icon: 'grid_view', route: '/dashboard' },
]

const adminLinks = [
  { title: 'User Management', icon: 'manage_accounts', route: '/dashboard/users' },
  { title: 'Roles & Permissions', icon: 'admin_panel_settings', route: '/dashboard/roles' },
]
</script>

<style scoped>
.menu-item {
  border-radius: 12px;
  color: #637381;
  transition: all 0.3s ease;
  padding: 10px 16px;
}

.menu-item:hover {
  background: #f4f6f8;
  color: #212b36;
}

.menu-active {
  background: rgba(25, 118, 210, 0.08) !important;
  color: var(--q-primary) !important;
}

.menu-active .menu-icon {
  color: var(--q-primary) !important;
}

.menu-text {
  font-size: 0.95rem;
}
</style>
