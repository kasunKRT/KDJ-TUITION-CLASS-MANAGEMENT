<template>
  <q-header class="bg-white text-dark" style="border-bottom: 1px solid #f0f0f0; height: 70px; display: flex; align-items: center;">
    <q-toolbar class="q-py-sm">
      <q-btn
        flat
        dense
        round
        icon="menu"
        aria-label="Menu"
        @click="$emit('toggle-sidebar')"
        class="q-mr-sm text-grey-8"
      />

      <!-- Search Bar -->
      <q-input
        dense
        outlined
        rounded
        v-model="search"
        placeholder="Search students, classes, or ID..."
        class="q-ml-md"
        style="width: 350px; max-width: 100%"
        color="primary"
      >
        <template v-slot:prepend>
          <q-icon name="search" color="grey-6" />
        </template>
      </q-input>

      <q-space />

      <!-- Action Buttons -->
      <div class="row items-center q-gutter-sm">
        <q-btn flat round dense icon="notifications_none" color="grey-7">
          <q-badge color="negative" text-color="white" floating rounded v-if="notificationsCount > 0">{{ notificationsCount }}</q-badge>
        </q-btn>
        
        <q-btn flat round dense icon="brightness_2" color="grey-7">
          <q-tooltip>Dark Mode (Coming Soon)</q-tooltip>
        </q-btn>

        <q-separator vertical inset class="q-mx-sm bg-grey-3" />

        <!-- User Profile -->
        <q-btn flat no-caps class="q-px-sm" style="border-radius: 8px;">
          <div class="row items-center">
            <q-avatar size="38px" color="primary" text-color="white" class="shadow-2">
              <span class="text-weight-bold text-subtitle1">{{ userInitials }}</span>
            </q-avatar>
            <div class="column q-ml-sm text-left">
              <span class="text-weight-bold text-dark" style="font-size: 0.9rem;">{{ userName || 'User' }}</span>
              <span class="text-grey-6" style="font-size: 0.75rem; line-height: 1;">{{ userRole || 'Loading...' }}</span>
            </div>
            <q-icon name="keyboard_arrow_down" size="xs" color="grey-6" class="q-ml-xs"/>
          </div>
          
          <q-menu fit anchor="bottom right" self="top right" :offset="[0, 8]" class="shadow-10" style="border-radius: 12px; min-width: 220px;">
            <q-list class="q-py-sm">
              <q-item-label header class="text-weight-bold text-grey-8">My Account</q-item-label>
              
              <q-item clickable v-close-popup class="menu-action-item" to="/dashboard/profile">
                <q-item-section avatar>
                  <q-icon name="account_circle" color="grey-7" />
                </q-item-section>
                <q-item-section class="text-weight-medium">Profile Settings</q-item-section>
              </q-item>
              
              <q-separator class="q-my-sm" />
              
              <q-item clickable v-close-popup @click="handleLogout" class="menu-action-item error-item text-negative">
                <q-item-section avatar>
                  <q-icon name="logout" color="negative" />
                </q-item-section>
                <q-item-section class="text-weight-bold">Logout</q-item-section>
              </q-item>
            </q-list>
          </q-menu>
        </q-btn>
      </div>
    </q-toolbar>
  </q-header>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'
import { useQuasar } from 'quasar'
import { supabase } from 'boot/supabase'

defineOptions({
  name: 'DashboardTopbar'
})

defineEmits(['toggle-sidebar'])
const search = ref('')

const router = useRouter()
const $q = useQuasar()

// State for topbar
const notificationsCount = ref(0)
const userName = ref('')
const userRole = ref('')

const userInitials = computed(() => {
  if (userName.value) {
    const parts = userName.value.split(' ')
    if (parts.length > 1) {
      return (parts[0][0] + parts[1][0]).toUpperCase()
    }
    return userName.value.substring(0, 2).toUpperCase()
  }
  return 'U'
})

const handleLogout = async () => {
  const { error } = await supabase.auth.signOut()
  if (error) {
    $q.notify({
      type: 'negative',
      message: error.message,
      position: 'top-right'
    })
  } else {
    $q.notify({
      type: 'positive',
      message: 'Successfully logged out.',
      position: 'top-right'
    })
    router.push('/login')
  }
}
</script>

<style scoped>
:deep(.q-field--outlined .q-field__control) {
  border-radius: 50px;
  background: #f4f6f8;
  border: none;
  transition: all 0.3s ease;
}
:deep(.q-field--outlined:not(.q-field--highlighted) .q-field__control:before) {
  border-color: transparent;
}
:deep(.q-field--outlined.q-field--highlighted .q-field__control) {
  background: #fff;
  box-shadow: 0 0 0 2px rgba(25, 118, 210, 0.2);
}

.menu-action-item {
  transition: all 0.2s;
  color: #444;
}
.menu-action-item:hover {
  background: #f4f6f8;
}
.error-item:hover {
  background: #fff5f5;
}
</style>
