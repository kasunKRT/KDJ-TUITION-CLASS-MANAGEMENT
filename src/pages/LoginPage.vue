<template>
  <q-page class="bg-primary text-secondary flex flex-center" style="min-height: 100vh;">
    <!-- Background overlay if needed, similar to hero section -->
    <div class="absolute-full" style="background: radial-gradient(circle at 50% 50%, rgba(255,255,255,0.02) 0%, transparent 60%); pointer-events: none;"></div>

    <!-- Glassmorphic Home Button -->
    <q-btn
      to="/"
      class="absolute-top-left q-mt-lg q-ml-lg"
      style="z-index: 10; border-radius: 12px; border: 1px solid rgba(255,255,255,0.1); background: rgba(0,0,0,0.4); backdrop-filter: blur(10px); padding: 8px 16px; transition: all 0.3s ease;"
      flat
      no-caps
    >
      <div class="row items-center q-gutter-sm">
        <q-icon name="arrow_back" color="secondary" size="xs" />
        <div class="text-secondary text-weight-medium" style="letter-spacing: 0.5px;">Return Home</div>
      </div>
    </q-btn>


    <div class="row justify-center w-full q-pa-md" style="max-width: 450px; z-index: 1;">
      <div class="text-center full-width q-mb-xl">
        <router-link to="/" class="text-decoration-none text-secondary">
          <div class="text-weight-bold text-h4 row items-center justify-center cursor-pointer" style="letter-spacing: 2px;">
            <q-icon name="school" class="q-mr-sm" /> Class<span class="text-accent">Master</span>
          </div>
        </router-link>
        <div class="text-accent q-mt-md text-subtitle1">Welcome back, please login to your account.</div>
      </div>

      <q-card class="bg-primary text-secondary full-width q-pa-lg q-pa-md-xl shadow-10" style="border: 1px solid rgba(255,255,255,0.08); border-radius: 20px;">
        <q-form @submit.prevent="handleLogin" class="q-gutter-md">
          <div class="q-mb-md">
            <div class="text-weight-medium q-mb-xs text-accent text-body2">Email Address</div>
            <q-input 
              v-model="email" 
              outlined 
              dark 
              color="secondary"
              placeholder="you@example.com"
              lazy-rules
              :rules="[val => !!val || 'Email is required', val => /.+@.+\..+/.test(val) || 'Invalid email address']"
            >
              <template v-slot:prepend>
                <q-icon name="mail" color="accent" />
              </template>
            </q-input>
          </div>

          <div class="q-mb-lg">
            <div class="row justify-between items-center q-mb-xs">
              <div class="text-weight-medium text-accent text-body2">Password</div>
              <q-btn flat dense no-caps label="Forgot password?" color="accent" class="text-caption" />
            </div>
            <q-input 
              v-model="password" 
              outlined 
              dark 
              :type="showPassword ? 'text' : 'password'"
              color="secondary"
              placeholder="Enter your password"
              lazy-rules
              :rules="[val => !!val || 'Password is required']"
            >
              <template v-slot:prepend>
                <q-icon name="lock" color="accent" />
              </template>
              <template v-slot:append>
                <q-icon
                  :name="showPassword ? 'visibility' : 'visibility_off'"
                  class="cursor-pointer"
                  color="accent"
                  @click="showPassword = !showPassword"
                />
              </template>
            </q-input>
          </div>

          <q-btn 
            :loading="loading"
            type="submit" 
            unelevated 
            label="Log In" 
            color="secondary" 
            text-color="primary" 
            class="full-width q-py-md text-weight-bold text-capitalize shadow-10" 
            style="border-radius: 10px; font-size: 1.1rem;" 
          />

          <q-btn 
            outline 
            label="Login with Google" 
            icon="img:https://www.gstatic.com/firebasejs/ui/2.0.0/images/auth/google.svg"
            color="secondary" 
            class="full-width q-py-md text-weight-bold text-capitalize q-mt-md" 
            style="border-radius: 10px; font-size: 1rem;"
            @click="handleGoogleLogin"
          />
        </q-form>
      </q-card>

      <div class="text-center full-width q-mt-xl text-accent">
        Don't have an account? 
        <router-link to="/register" class="text-secondary text-weight-bold" style="text-decoration: underline; text-underline-offset: 4px;">
          Create an account
        </router-link>
      </div>
    </div>
  </q-page>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { useQuasar } from 'quasar'
import { supabase } from 'boot/supabase'

const router = useRouter()
const $q = useQuasar()

const email = ref('')
const password = ref('')
const showPassword = ref(false)
const loading = ref(false)

const handleLogin = async () => {
  try {
    loading.value = true
    const { error } = await supabase.auth.signInWithPassword({
      email: email.value,
      password: password.value
    })

    if (error) throw error

    $q.notify({
      type: 'positive',
      message: 'Successfully logged in!',
      position: 'top-right'
    })
    
    // Redirect to dashboard or home
    router.push('/dashboard') // Update to proper route once dashboard exists
  } catch (error) {
    let message = error.message
    if (message === 'Invalid login credentials') {
      message = 'Invalid email or password.'
    }
    
    $q.notify({
      type: 'negative',
      message: message,
      position: 'top-right'
    })
  } finally {
    loading.value = false
  }
}

const handleGoogleLogin = async () => {
  try {
    const { error } = await supabase.auth.signInWithOAuth({
      provider: 'google',
    })
    if (error) throw error
  } catch (error) {
    $q.notify({
      type: 'negative',
      message: error.message,
      position: 'top-right'
    })
  }
}
</script>

<style scoped>
.w-full {
  width: 100%;
}
.text-decoration-none {
  text-decoration: none;
}
:deep(.q-field--outlined .q-field__control) {
  border-radius: 10px;
}
:deep(.q-field--outlined:not(.q-field--highlighted) .q-field__control:before) {
  border-color: rgba(255,255,255,0.15);
}
:deep(.q-field--outlined.q-field--highlighted .q-field__control:after) {
  border-width: 2px;
}
</style>
