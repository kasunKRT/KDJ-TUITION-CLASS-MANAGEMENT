<template>
  <q-page class="q-pa-md bg-dark text-secondary flex flex-center" style="min-height: 100vh;">
    <div class="text-center">
      <q-icon name="dashboard" size="100px" color="primary" class="q-mb-md" />
      <div class="text-h3 text-weight-bold q-mb-sm text-secondary">Dashboard</div>
      <div class="text-h6 text-accent q-mb-lg">
        Welcome to your new ClassMaster dashboard!
      </div>
      <q-btn
        unelevated
        color="primary"
        text-color="secondary"
        label="Logout"
        icon="logout"
        class="text-weight-bold q-py-sm q-px-md"
        style="border-radius: 8px;"
        @click="handleLogout"
      />
    </div>
  </q-page>
</template>

<script setup>
import { useRouter } from 'vue-router'
import { useQuasar } from 'quasar'
import { supabase } from 'boot/supabase'

const router = useRouter()
const $q = useQuasar()

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
</style>
