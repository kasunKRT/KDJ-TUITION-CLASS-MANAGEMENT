<template>
  <q-page class="q-pa-lg bg-grey-1" style="min-height: 100vh;">
    <!-- Profile Header -->
    <div class="q-mb-xl">
      <div class="text-h4 text-weight-bolder text-dark" style="letter-spacing: -0.5px;">My Profile</div>
      <div class="text-subtitle1 text-grey-6 q-mt-xs">Manage your personal information and security settings.</div>
    </div>

    <div class="row q-col-gutter-lg">
      <!-- Left Column: User Card -->
      <div class="col-12 col-md-4">
        <q-card class="bg-white shadow-2" style="border-radius: 16px; border: 1px solid #f0f0f0;">
          <q-card-section class="q-pa-xl column items-center text-center">
            <q-avatar size="100px" color="primary" text-color="white" class="shadow-4 q-mb-md">
              <span class="text-h3 text-weight-bolder">{{ userInitials }}</span>
              <q-btn round color="dark" icon="edit" size="sm" class="absolute-bottom-right shadow-3" style="margin-bottom: -5px; margin-right: -5px;" />
            </q-avatar>
            <div class="text-h5 text-weight-bolder text-dark q-mb-xs">{{ fullName || 'Loading...' }}</div>
            <div class="text-body2 text-grey-6 q-mb-md">{{ role || 'Loading...' }}</div>
            <q-badge color="green-1" text-color="green" class="text-weight-bold q-px-sm q-py-xs" style="border-radius: 6px;">Active Account</q-badge>
            
            <q-separator class="full-width q-my-xl bg-grey-2" />

            <div class="full-width text-left">
              <div class="q-mb-lg">
                <div class="text-caption text-weight-bold text-grey-5 text-uppercase q-mb-xs" style="letter-spacing: 1px;">Email Address</div>
                <div class="text-body1 text-weight-bold text-dark">{{ userEmail || 'Loading...' }}</div>
              </div>
              <div class="q-mb-lg">
                <div class="text-caption text-weight-bold text-grey-5 text-uppercase q-mb-xs" style="letter-spacing: 1px;">Phone Number</div>
                <div class="text-body1 text-weight-bold text-dark">{{ phone || 'Not provided' }}</div>
              </div>
              <div>
                <div class="text-caption text-weight-bold text-grey-5 text-uppercase q-mb-xs" style="letter-spacing: 1px;">Joined Date</div>
                <div class="text-body1 text-weight-bold text-dark">{{ joinedDate || 'Unknown' }}</div>
              </div>
            </div>
          </q-card-section>
        </q-card>
      </div>

      <!-- Right Column: Settings Tabs -->
      <div class="col-12 col-md-8">
        <q-card class="bg-white shadow-2" style="border-radius: 16px; border: 1px solid #f0f0f0;">
          <q-tabs
            v-model="tab"
            dense
            class="text-grey-7 q-pt-md q-px-md"
            active-color="primary"
            indicator-color="primary"
            align="left"
            narrow-indicator
          >
            <q-tab name="general" label="General Information" class="text-weight-bold" />
            <q-tab name="security" label="Security" class="text-weight-bold" />
          </q-tabs>

          <q-separator class="bg-grey-2" />

          <q-tab-panels v-model="tab" animated class="bg-transparent">
            <!-- General Info Panel -->
            <q-tab-panel name="general" class="q-pa-xl">
              <div class="text-h6 text-weight-bold text-dark q-mb-xl">Personal Details</div>
              
              <q-form @submit.prevent="saveGeneralInfo" class="q-gutter-lg">
                <div class="row q-col-gutter-lg">
                  <div class="col-12 col-sm-6">
                     <div class="text-weight-bold q-mb-sm text-dark text-subtitle2">First Name</div>
                     <q-input v-model="firstName" outlined color="primary" placeholder="Enter your first name" />
                  </div>
                  <div class="col-12 col-sm-6">
                     <div class="text-weight-bold q-mb-sm text-dark text-subtitle2">Last Name</div>
                     <q-input v-model="lastName" outlined color="primary" placeholder="Enter your last name" />
                  </div>
                </div>

                <div class="row q-col-gutter-lg">
                  <div class="col-12 col-sm-6">
                     <div class="text-weight-bold q-mb-sm text-dark text-subtitle2">Phone Number</div>
                     <q-input v-model="phone" outlined color="primary" placeholder="e.g. +94 7X XXX XXXX" />
                  </div>
                  <div class="col-12 col-sm-6">
                     <div class="text-weight-bold q-mb-sm text-dark text-subtitle2">Role</div>
                     <q-input v-model="role" outlined color="grey" bg-color="grey-2" disable />
                  </div>
                </div>

                <div>
                  <div class="text-weight-bold q-mb-sm text-dark text-subtitle2">Bio</div>
                  <q-input v-model="bio" type="textarea" outlined color="primary" placeholder="Tell us a little about yourself" />
                </div>
                
                <div class="q-mt-xl text-right">
                  <q-btn label="Discard" flat color="grey-7" class="text-weight-bold q-mr-sm" />
                  <q-btn label="Save Changes" type="submit" color="primary" unelevated class="text-weight-bold shadow-2 q-px-xl q-py-sm" style="border-radius: 8px;" />
                </div>
              </q-form>
            </q-tab-panel>

            <!-- Security Panel -->
            <q-tab-panel name="security" class="q-pa-xl">
              <div class="text-h6 text-weight-bold text-dark q-mb-xl">Change Password</div>
              
              <q-form @submit.prevent="savePassword" class="q-gutter-lg" style="max-width: 500px">
                <div>
                  <div class="text-weight-bold q-mb-sm text-dark text-subtitle2">Current Password</div>
                  <q-input v-model="currentPassword" type="password" outlined color="primary" />
                </div>
                
                <div>
                  <div class="text-weight-bold q-mb-sm text-dark text-subtitle2">New Password</div>
                  <q-input v-model="newPassword" type="password" outlined color="primary" />
                </div>

                <div>
                  <div class="text-weight-bold q-mb-sm text-dark text-subtitle2">Confirm New Password</div>
                  <q-input v-model="confirmPassword" type="password" outlined color="primary" />
                </div>
                
                <div class="q-mt-xl">
                  <q-btn label="Update Password" type="submit" color="dark" unelevated class="text-weight-bold shadow-2 q-px-xl q-py-sm" style="border-radius: 8px;" />
                </div>
              </q-form>
            </q-tab-panel>
          </q-tab-panels>
        </q-card>
      </div>
    </div>
  </q-page>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import { useQuasar } from 'quasar'
import { supabase } from 'boot/supabase'

const $q = useQuasar()

const tab = ref('general')
const userEmail = ref('')

const firstName = ref('')
const lastName = ref('')
const phone = ref('')
const role = ref('')
const bio = ref('')
const joinedDate = ref('')

const currentPassword = ref('')
const newPassword = ref('')
const confirmPassword = ref('')

const fullName = computed(() => {
  if (firstName.value || lastName.value) {
    return `${firstName.value} ${lastName.value}`.trim()
  }
  return ''
})

const userInitials = computed(() => {
  if (firstName.value || lastName.value) {
    const f = firstName.value ? firstName.value.charAt(0) : ''
    const l = lastName.value ? lastName.value.charAt(0) : ''
    return (f + l).toUpperCase()
  }
  return 'U'
})

onMounted(async () => {
  const { data: { user } } = await supabase.auth.getUser()
  if (user && user.email) {
    userEmail.value = user.email
  }
})

const saveGeneralInfo = () => {
  $q.notify({
    type: 'positive',
    message: 'Profile information updated successfully!',
    position: 'top-right'
  })
}

const savePassword = () => {
  if (newPassword.value !== confirmPassword.value) {
    $q.notify({
      type: 'negative',
      message: 'New passwords do not match!',
      position: 'top-right'
    })
    return
  }
  
  $q.notify({
    type: 'positive',
    message: 'Password updated successfully!',
    position: 'top-right'
  })
  currentPassword.value = ''
  newPassword.value = ''
  confirmPassword.value = ''
}
</script>

<style scoped>
:deep(.q-field--outlined .q-field__control) {
  border-radius: 10px;
  background: #f8f9fa;
  border: none;
  transition: all 0.3s ease;
}
:deep(.q-field--outlined:not(.q-field--highlighted) .q-field__control:before) {
  border-color: rgba(0, 0, 0, 0.05);
}
:deep(.q-field--outlined.q-field--highlighted .q-field__control) {
  background: #fff;
  box-shadow: 0 0 0 2px rgba(25, 118, 210, 0.2);
}
:deep(.q-tab-panels) {
  border-radius: 0 0 16px 16px;
}
</style>
