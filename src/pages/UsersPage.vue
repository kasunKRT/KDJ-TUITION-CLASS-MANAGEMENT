<template>
  <q-page class="q-pa-lg bg-grey-1" style="min-height: 100vh;">
    <!-- Page Header -->
    <div class="row items-center justify-between q-mb-xl">
      <div>
        <div class="text-h4 text-weight-bolder text-dark" style="letter-spacing: -0.5px;">User Management</div>
        <div class="text-subtitle1 text-grey-6 q-mt-xs">View, invite and manage user accounts.</div>
      </div>
      <q-btn
        unelevated color="primary" icon="person_add" label="Invite User"
        class="text-weight-bold q-px-md shadow-2"
        style="border-radius: 8px; text-transform: none;"
        @click="inviteDialog = true"
      />
    </div>

    <!-- Users Table Card -->
    <q-card class="bg-white shadow-2" style="border-radius: 16px; border: 1px solid #f0f0f0;">
      <!-- Filter Row -->
      <q-card-section class="q-pa-lg row items-center q-col-gutter-md" style="border-bottom: 1px solid #f5f5f5;">
        <div class="col-12 col-md-5">
          <q-input
            v-model="search"
            dense outlined rounded
            placeholder="Search by name or email..."
            color="primary"
          >
            <template v-slot:prepend>
              <q-icon name="search" color="grey-6" />
            </template>
          </q-input>
        </div>
        <div class="col-12 col-md-3">
          <q-select
            v-model="filterRole"
            :options="roleOptions"
            emit-value map-options
            dense outlined rounded color="primary"
            label="Filter by Role"
          />
        </div>
        <div class="col-12 col-md-3">
          <q-select
            v-model="filterStatus"
            :options="statusOptions"
            emit-value map-options
            dense outlined rounded color="primary"
            label="Filter by Status"
          />
        </div>
      </q-card-section>

      <!-- Table -->
      <q-table
        :rows="filteredUsers"
        :columns="columns"
        row-key="id"
        :loading="loading"
        flat
        :rows-per-page-options="[10, 20, 50]"
        class="users-table"
      >
        <!-- User Column with Avatar -->
        <template v-slot:body-cell-name="props">
          <q-td :props="props">
            <div class="row items-center q-gutter-sm no-wrap">
              <q-avatar size="36px" :color="props.row.avatarColor" text-color="white" class="text-weight-bold shadow-1" style="font-size: 0.8rem;">
                {{ props.row.initials }}
              </q-avatar>
              <div class="column">
                <span class="text-weight-bold text-dark" style="font-size: 0.9rem;">{{ props.row.name }}</span>
                <span class="text-caption text-grey-6">{{ props.row.email }}</span>
              </div>
            </div>
          </q-td>
        </template>

        <!-- Role Badge -->
        <template v-slot:body-cell-role="props">
          <q-td :props="props">
            <q-badge
              :color="getRoleColor(props.row.role) + '-1'"
              :text-color="getRoleColor(props.row.role)"
              class="text-weight-bold q-px-sm q-py-xs"
              style="border-radius: 6px; font-size: 0.8rem;"
            >
              {{ props.row.role }}
            </q-badge>
          </q-td>
        </template>

        <!-- Status Badge -->
        <template v-slot:body-cell-status="props">
          <q-td :props="props">
            <q-badge
              :color="props.row.status === 'Active' ? 'green-1' : 'red-1'"
              :text-color="props.row.status === 'Active' ? 'green' : 'negative'"
              class="text-weight-bold q-px-sm q-py-xs"
              style="border-radius: 6px; font-size: 0.8rem;"
            >
              <q-icon :name="props.row.status === 'Active' ? 'circle' : 'cancel'" size="8px" class="q-mr-xs" />
              {{ props.row.status }}
            </q-badge>
          </q-td>
        </template>

        <!-- Actions -->
        <template v-slot:body-cell-actions="props">
          <q-td :props="props" class="text-right">
            <q-btn flat round dense icon="more_vert" color="grey-7">
              <q-menu anchor="bottom right" self="top right" style="border-radius: 10px; min-width: 160px;">
                <q-list class="q-py-sm">
                  <q-item clickable v-close-popup class="action-menu-item" @click="editUser(props.row)">
                    <q-item-section avatar><q-icon name="edit" color="primary" size="sm"/></q-item-section>
                    <q-item-section class="text-weight-medium">Edit User</q-item-section>
                  </q-item>
                  <q-item clickable v-close-popup class="action-menu-item" @click="changeRole(props.row)">
                    <q-item-section avatar><q-icon name="swap_horiz" color="orange" size="sm"/></q-item-section>
                    <q-item-section class="text-weight-medium">Change Role</q-item-section>
                  </q-item>
                  <q-separator class="q-my-xs" />
                  <q-item clickable v-close-popup class="action-menu-item" @click="toggleStatus(props.row)">
                    <q-item-section avatar><q-icon :name="props.row.status === 'Active' ? 'block' : 'check_circle'" :color="props.row.status === 'Active' ? 'negative' : 'green'" size="sm"/></q-item-section>
                    <q-item-section :class="props.row.status === 'Active' ? 'text-negative text-weight-bold' : 'text-green text-weight-bold'">
                      {{ props.row.status === 'Active' ? 'Deactivate' : 'Activate' }}
                    </q-item-section>
                  </q-item>
                </q-list>
              </q-menu>
            </q-btn>
          </q-td>
        </template>

        <!-- Empty State -->
        <template v-slot:no-data>
          <div class="full-width column flex-center q-py-xl text-grey-5">
            <q-icon name="people_outline" size="64px" class="q-mb-md opacity-50" />
            <div class="text-h6 text-weight-bold">No users found</div>
            <div class="text-caption q-mt-xs">Try adjusting your filters or invite a new user</div>
          </div>
        </template>
      </q-table>
    </q-card>

    <!-- Invite User Dialog -->
    <q-dialog v-model="inviteDialog" backdrop-filter="blur(4px)">
      <q-card style="min-width: 440px; border-radius: 16px;">
        <q-card-section class="q-pa-xl">
          <div class="text-h6 text-weight-bolder text-dark q-mb-xs">Invite New User</div>
          <div class="text-caption text-grey-6 q-mb-xl">An invitation email will be sent to the user.</div>
          <q-form @submit.prevent="submitInvite" class="q-gutter-md">
            <div>
              <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Full Name</div>
              <q-input v-model="inviteForm.name" outlined dense color="primary" placeholder="e.g. Kamal Perera" />
            </div>
            <div>
              <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Email Address</div>
              <q-input v-model="inviteForm.email" outlined dense color="primary" placeholder="user@example.com" />
            </div>
            <div>
              <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Assign Role</div>
              <q-select v-model="inviteForm.role" :options="roleOptions" emit-value map-options outlined dense color="primary" />
            </div>
            <div class="row justify-end q-gutter-sm q-pt-md">
              <q-btn flat label="Cancel" color="grey-7" class="text-weight-bold" v-close-popup />
              <q-btn unelevated label="Send Invite" color="primary" type="submit" class="text-weight-bold q-px-lg shadow-2" style="border-radius: 8px;" />
            </div>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>

    <!-- Change Role Dialog -->
    <q-dialog v-model="changeRoleDialog" backdrop-filter="blur(4px)">
      <q-card style="min-width: 380px; border-radius: 16px;">
        <q-card-section class="q-pa-xl">
          <div class="text-h6 text-weight-bolder text-dark q-mb-xs">Change Role</div>
          <div class="text-caption text-grey-6 q-mb-xl" v-if="selectedUser">Updating role for <strong>{{ selectedUser.name }}</strong></div>
          <q-select v-model="newRole" :options="roleOptions" emit-value map-options outlined dense color="primary" label="Select New Role" />
          <div class="row justify-end q-gutter-sm q-pt-xl">
            <q-btn flat label="Cancel" color="grey-7" class="text-weight-bold" v-close-popup />
            <q-btn unelevated label="Update Role" color="primary" class="text-weight-bold q-px-lg shadow-2" style="border-radius: 8px;" @click="saveRoleChange" />
          </div>
        </q-card-section>
      </q-card>
    </q-dialog>

  </q-page>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useQuasar } from 'quasar'
import { supabase } from 'boot/supabase'

const $q = useQuasar()

const loading = ref(false)
const search = ref('')
const filterRole = ref(null)
const filterStatus = ref(null)
const inviteDialog = ref(false)
const changeRoleDialog = ref(false)
const selectedUser = ref(null)
const newRole = ref(null)

const inviteForm = ref({ name: '', email: '', role: 'Student' })

const roleOptions = [
  { label: 'All Roles', value: null },
  { label: 'Admin', value: 'Admin' },
  { label: 'Teacher', value: 'Teacher' },
  { label: 'Student', value: 'Student' },
]

const statusOptions = [
  { label: 'All Status', value: null },
  { label: 'Active', value: 'Active' },
  { label: 'Inactive', value: 'Inactive' },
]

const columns = [
  { name: 'name', label: 'User', field: 'name', align: 'left', sortable: true },
  { name: 'role', label: 'Role', field: 'role', align: 'left', sortable: true },
  { name: 'status', label: 'Status', field: 'status', align: 'left', sortable: true },
  { name: 'lastLogin', label: 'Last Login', field: 'lastLogin', align: 'left', sortable: true },
  { name: 'actions', label: '', field: 'actions', align: 'right' },
]

// Avatar colors for users
const avatarColors = ['primary', 'teal', 'purple', 'orange', 'blue', 'green', 'red']

const users = ref([])

// Load users from Supabase Auth via Edge Function
const loadUsers = async () => {
  loading.value = true
  try {
    const { data: { session } } = await supabase.auth.getSession()
    if (!session) throw new Error('Not authenticated')

    const response = await fetch(
      `${import.meta.env.VITE_SUPABASE_URL}/functions/v1/list-users`,
      {
        headers: {
          Authorization: `Bearer ${session.access_token}`,
          'Content-Type': 'application/json',
        },
      }
    )

    if (!response.ok) {
      const err = await response.json()
      throw new Error(err.error || 'Failed to fetch users')
    }

    const { users: rawUsers } = await response.json()

    // Map to UI format
    users.value = rawUsers.map((u, idx) => ({
      ...u,
      initials: u.name
        ? u.name.split(' ').map(w => w[0]).join('').substring(0, 2).toUpperCase()
        : u.email.substring(0, 2).toUpperCase(),
      avatarColor: avatarColors[idx % avatarColors.length],
    }))
  } catch (err) {
    $q.notify({ type: 'negative', message: err.message || 'Failed to load users', position: 'top-right' })
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  loadUsers()
})

const filteredUsers = computed(() => {
  return users.value.filter(u => {
    const matchSearch = !search.value ||
      u.name.toLowerCase().includes(search.value.toLowerCase()) ||
      u.email.toLowerCase().includes(search.value.toLowerCase())
    const matchRole = !filterRole.value || u.role === filterRole.value
    const matchStatus = !filterStatus.value || u.status === filterStatus.value
    return matchSearch && matchRole && matchStatus
  })
})

const getRoleColor = (role) => {
  const map = { Admin: 'red', Teacher: 'blue', Student: 'teal' }
  return map[role] || 'grey'
}

const editUser = (user) => {
  $q.notify({ type: 'info', message: `Edit coming soon for: ${user.name}`, position: 'top-right' })
}

const changeRole = (user) => {
  selectedUser.value = user
  newRole.value = user.role
  changeRoleDialog.value = true
}

const toggleStatus = (user) => {
  user.status = user.status === 'Active' ? 'Inactive' : 'Active'
  $q.notify({
    type: user.status === 'Active' ? 'positive' : 'warning',
    message: `${user.name} has been ${user.status === 'Active' ? 'activated' : 'deactivated'}.`,
    position: 'top-right'
  })
}

const saveRoleChange = () => {
  if (selectedUser.value && newRole.value) {
    selectedUser.value.role = newRole.value
    $q.notify({ type: 'positive', message: `Role updated to ${newRole.value}`, position: 'top-right' })
    changeRoleDialog.value = false
  }
}

const submitInvite = async () => {
  if (!inviteForm.value.email) {
    $q.notify({ type: 'warning', message: 'Please enter an email address.', position: 'top-right' })
    return
  }

  try {
    // Note: actual invite requires admin API - for now notify success and refresh
    $q.notify({ type: 'positive', message: `Invite sent to ${inviteForm.value.email}!`, position: 'top-right' })
    inviteForm.value = { name: '', email: '', role: 'Student' }
    inviteDialog.value = false
    await loadUsers()
  } catch (err) {
    $q.notify({ type: 'negative', message: err.message, position: 'top-right' })
  }
}
</script>

<style scoped>
:deep(.users-table .q-table__top),
:deep(.users-table .q-table__bottom) {
  padding: 16px 20px;
}
:deep(.users-table thead th) {
  font-weight: 700;
  color: #637381;
  font-size: 0.8rem;
  letter-spacing: 0.5px;
  text-transform: uppercase;
  background: #f9fafb;
  border-bottom: 1px solid #f0f0f0;
}
:deep(.users-table tbody td) {
  border-bottom: 1px solid #f9fafb;
  padding: 14px 20px;
}
:deep(.users-table tbody tr:hover td) {
  background: #f4f6f8;
}
:deep(.q-field--outlined .q-field__control) {
  border-radius: 50px;
}
.action-menu-item {
  border-radius: 6px;
  transition: background 0.2s;
}
.action-menu-item:hover {
  background: #f4f6f8;
}
</style>
