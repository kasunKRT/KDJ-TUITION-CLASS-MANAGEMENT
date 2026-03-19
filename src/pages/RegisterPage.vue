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
        <div class="text-accent q-mt-md text-subtitle1">Create a new account and get started!</div>
      </div>

      <q-card class="bg-primary text-secondary full-width q-pa-lg q-pa-md-xl shadow-10" style="border: 1px solid rgba(255,255,255,0.08); border-radius: 20px;">
        <q-form @submit.prevent="handleRegister" class="q-gutter-md">
          <div class="row q-col-gutter-sm q-mb-md">
            <!-- Full Name -->
            <div class="col-12">
              <div class="text-weight-medium q-mb-xs text-accent text-body2">Full Name</div>
              <q-input 
                v-model="fullName" 
                outlined 
                dark 
                color="secondary"
                placeholder="John Doe"
                lazy-rules
                :rules="[val => !!val || 'Name is required']"
              >
                <template v-slot:prepend>
                  <q-icon name="person" color="accent" />
                </template>
              </q-input>
            </div>

            <!-- Email Address -->
            <div class="col-12 q-mt-sm">
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

            <!-- Password -->
            <div class="col-12 q-mt-sm">
              <div class="text-weight-medium q-mb-xs text-accent text-body2">Password</div>
              <q-input 
                v-model="password" 
                outlined 
                dark 
                :type="showPassword ? 'text' : 'password'"
                color="secondary"
                placeholder="Must be at least 6 characters"
                lazy-rules
                :rules="[val => val && val.length >= 6 || 'Password must be at least 6 characters']"
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
            
            <!-- Default check -->
            <div class="col-12 q-mt-sm">
              <q-checkbox v-model="agreeTerms" color="secondary" class="text-caption text-accent" keep-color>
                I agree to the <a href="#" class="text-secondary">Terms</a> and <a href="#" class="text-secondary">Privacy Policy</a>
              </q-checkbox>
            </div>
          </div>

          <q-btn 
            :loading="loading"
            type="submit" 
            unelevated 
            label="Sign Up" 
            color="secondary" 
            text-color="primary" 
            class="full-width q-py-md text-weight-bold text-capitalize shadow-10 q-mt-md" 
            style="border-radius: 10px; font-size: 1.1rem;" 
          />
          
          <q-btn 
            outline 
            label="Sign up with Google" 
            icon="img:https://www.gstatic.com/firebasejs/ui/2.0.0/images/auth/google.svg"
            color="secondary" 
            class="full-width q-py-md text-weight-bold text-capitalize q-mt-md" 
            style="border-radius: 10px; font-size: 1rem;"
            @click="handleGoogleRegister"
          />
        </q-form>
      </q-card>

      <div class="text-center full-width q-mt-xl text-accent">
        Already have an account? 
        <router-link to="/login" class="text-secondary text-weight-bold" style="text-decoration: underline; text-underline-offset: 4px;">
          Log In
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

const fullName = ref('')
const email = ref('')
const password = ref('')
const showPassword = ref(false)
const loading = ref(false)
const agreeTerms = ref(false)

const handleRegister = async () => {
  if (!agreeTerms.value) {
    $q.notify({
      type: 'warning',
      message: 'Please agree to the Terms and Privacy Policy.',
      position: 'top-right'
    })
    return
  }
  
  try {
    loading.value = true
    const { error } = await supabase.auth.signUp({
      email: email.value,
      password: password.value,
      options: {
        data: {
          full_name: fullName.value,
        }
      }
    })

    if (error) throw error

    $q.notify({
      type: 'positive',
      message: 'Successfully registered! Please check your email.',
      position: 'top-right'
    })
    
    // Redirect to login or auto login depending on app configuration
    router.push('/login')
  } catch (error) {
    let message = error.message
    if (error.status === 400 && message.includes('already registered')) {
      message = 'This email is already registered.'
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

const handleGoogleRegister = async () => {
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
a:not(.q-btn) {
  text-decoration: underline;
  text-underline-offset: 4px;
}
</style>
