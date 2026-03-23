<template>
  <q-page class="q-pa-lg bg-grey-1" style="min-height: 100vh;">
    <!-- Page Header -->
    <div class="row items-center justify-between q-mb-xl">
      <div>
        <div class="text-h4 text-weight-bolder text-dark" style="letter-spacing: -0.5px;">User Management</div>
        <div class="text-subtitle1 text-grey-6 q-mt-xs">View, invite and manage user accounts.</div>
      </div>
      <q-btn
        unelevated color="primary" icon="person_add" label="Add User"
        class="text-weight-bold q-px-md shadow-2"
        style="border-radius: 8px; text-transform: none;"
        @click="addUserDialog = true"
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
                  <q-item clickable v-close-popup class="action-menu-item" @click="deleteUser(props.row)">
                    <q-item-section avatar><q-icon name="delete" color="negative" size="sm"/></q-item-section>
                    <q-item-section class="text-negative text-weight-bold">Delete</q-item-section>
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
            <div class="text-caption q-mt-xs">Try adjusting your filters or add a new user</div>
          </div>
        </template>
      </q-table>
    </q-card>

    <!-- Add User Dialog -->
    <q-dialog v-model="addUserDialog" backdrop-filter="blur(4px)">
      <q-card style="min-width: 440px; border-radius: 16px;">
        <q-card-section class="q-pa-xl">
          <div class="text-h6 text-weight-bolder text-dark q-mb-xs">Add New User</div>
          <div class="text-caption text-grey-6 q-mb-xl">Create a new user account manually.</div>
          <q-form @submit.prevent="submitAddUser" class="q-gutter-md">
            <div>
              <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Full Name</div>
              <q-input v-model="addUserForm.name" outlined dense color="primary" placeholder="e.g. Kamal Perera" />
            </div>
            <div>
              <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Email Address</div>
              <q-input v-model="addUserForm.email" outlined dense color="primary" placeholder="user@example.com" />
            </div>
            <div>
              <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Password</div>
              <q-input v-model="addUserForm.password" type="password" outlined dense color="primary" placeholder="••••••••" />
            </div>
            <div>
              <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Assign Role</div>
              <q-select v-model="addUserForm.role" :options="roleOptions.filter(r => r.value !== null)" emit-value map-options outlined dense color="primary" />
            </div>
            <div class="row justify-end q-gutter-sm q-pt-md">
              <q-btn flat label="Cancel" color="grey-7" class="text-weight-bold" v-close-popup />
              <q-btn unelevated label="Add User" color="primary" type="submit" class="text-weight-bold q-px-lg shadow-2" style="border-radius: 8px;" />
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

    <!-- Edit User Dialog -->
    <q-dialog v-model="editUserDialog" backdrop-filter="blur(4px)">
      <q-card style="min-width: 440px; border-radius: 16px;">
        <q-card-section class="q-pa-xl">
          <div class="text-h6 text-weight-bolder text-dark q-mb-xs">Edit User</div>
          <div class="text-caption text-grey-6 q-mb-xl">Update details for the user.</div>
          <q-form @submit.prevent="saveUserDetails" class="q-gutter-md">
            <div>
              <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Full Name</div>
              <q-input v-model="editUserForm.name" outlined dense color="primary" placeholder="e.g. Kamal Perera" />
            </div>
            <div>
              <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Email Address</div>
              <q-input v-model="editUserForm.email" outlined dense color="primary" placeholder="user@example.com" />
            </div>
            <div class="row justify-end q-gutter-sm q-pt-md">
              <q-btn flat label="Cancel" color="grey-7" class="text-weight-bold" v-close-popup />
              <q-btn unelevated label="Save Changes" color="primary" type="submit" class="text-weight-bold q-px-lg shadow-2" style="border-radius: 8px;" />
            </div>
          </q-form>
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
const addUserDialog = ref(false)
const changeRoleDialog = ref(false)
const editUserDialog = ref(false)
const selectedUser = ref(null)
const newRole = ref(null)

const editUserForm = ref({ id: '', name: '', email: '' })
const addUserForm = ref({ name: '', email: '', password: '', role: 'Student' })

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

    const { data, error } = await supabase.functions.invoke('list-users', {
      method: 'GET',
    })

    if (error) {
      throw new Error(error.message || 'Failed to fetch users')
    }

    // Map to UI format
    users.value = data.users.map((u, idx) => ({
      ...u,
      initials: u.name
        ? u.name.split(' ').map(w => w[0]).join('').substring(0, 2).toUpperCase()
        : u.email.substring(0, 2).toUpperCase(),
      avatarColor: avatarColors[idx % avatarColors.length],
    }))
  } catch (err) {
    console.error(err)
    $q.notify({ type: 'negative', message: 'Failed to load users: ' + (err.message || ''), position: 'top-right' })
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
  editUserForm.value = { id: user.id, name: user.name, email: user.email }
  editUserDialog.value = true
}

const saveUserDetails = async () => {
  if (!editUserForm.value.name || !editUserForm.value.email) {
    $q.notify({ type: 'warning', message: 'Name and email are required.', position: 'top-right' })
    return
  }

  try {
    $q.loading.show({ message: 'Saving details...' })
    const { error } = await supabase.functions.invoke('update-user-details', {
      body: { userId: editUserForm.value.id, name: editUserForm.value.name, email: editUserForm.value.email }
    })

    if (error) throw new Error(error.message || 'Failed to update details')

    const userIndex = users.value.findIndex(u => u.id === editUserForm.value.id)
    if (userIndex !== -1) {
      users.value[userIndex].name = editUserForm.value.name
      users.value[userIndex].email = editUserForm.value.email
    }

    $q.notify({ type: 'positive', message: 'User details updated successfully!', position: 'top-right' })
    editUserDialog.value = false
  } catch (err) {
    console.error(err)
    $q.notify({ type: 'negative', message: 'Failed to update user: ' + (err.message || 'Unknown error'), position: 'top-right' })
  } finally {
    $q.loading.hide()
  }
}

const deleteUser = (user) => {
  $q.dialog({
    title: 'Confirm Deletion',
    message: `Are you sure you want to completely delete ${user.name}? This action cannot be undone.`,
    persistent: true,
    ok: { color: 'negative', label: 'Delete', unelevated: true, style: 'border-radius: 8px;' },
    cancel: { flat: true, color: 'grey-7', style: 'border-radius: 8px;' }
  }).onOk(async () => {
    try {
      $q.loading.show({ message: 'Deleting user...' })
      const { error } = await supabase.functions.invoke('delete-user', {
        body: { userId: user.id }
      })

      if (error) throw new Error(error.message || 'Failed to delete user')

      users.value = users.value.filter(u => u.id !== user.id)
      $q.notify({ type: 'positive', message: `${user.name} has been deleted.`, position: 'top-right' })
    } catch (err) {
      console.error(err)
      $q.notify({ type: 'negative', message: 'Failed to delete user: ' + (err.message || 'Unknown error'), position: 'top-right' })
    } finally {
      $q.loading.hide()
    }
  })
}

const changeRole = (user) => {
  selectedUser.value = user
  newRole.value = user.role
  changeRoleDialog.value = true
}

const toggleStatus = async (user) => {
  const previousStatus = user.status
  const newStatus = previousStatus === 'Active' ? 'Inactive' : 'Active'
  
  try {
    $q.loading.show({ message: `Changing status to ${newStatus}...` })
    const { error } = await supabase.functions.invoke('update-user-status', {
      body: { userId: user.id, status: newStatus }
    })

    if (error) {
      throw new Error(error.message || 'Failed to update status')
    }

    user.status = newStatus
    $q.notify({
      type: newStatus === 'Active' ? 'positive' : 'warning',
      message: `${user.name} has been ${newStatus === 'Active' ? 'activated' : 'deactivated'}.`,
      position: 'top-right'
    })
  } catch (err) {
    console.error(err)
    $q.notify({ type: 'negative', message: 'Failed to update status: ' + (err.message || 'Unknown error'), position: 'top-right' })
  } finally {
    $q.loading.hide()
  }
}

const saveRoleChange = async () => {
  if (selectedUser.value && newRole.value) {
    try {
      $q.loading.show({ message: 'Updating role...' })
      const { error } = await supabase.functions.invoke('update-user-role', {
        body: { userId: selectedUser.value.id, role: newRole.value }
      })

      if (error) {
        throw new Error(error.message || 'Failed to update role')
      }

      selectedUser.value.role = newRole.value
      $q.notify({ type: 'positive', message: `Role updated to ${newRole.value}`, position: 'top-right' })
      changeRoleDialog.value = false
    } catch (err) {
      console.error(err)
      $q.notify({ type: 'negative', message: 'Failed to update user role: ' + (err.message || 'Unknown error'), position: 'top-right' })
    } finally {
      $q.loading.hide()
    }
  }
}

const submitAddUser = async () => {
  if (!addUserForm.value.email || !addUserForm.value.password || !addUserForm.value.name) {
    $q.notify({ type: 'warning', message: 'All fields are required.', position: 'top-right' })
    return
  }

  try {
    $q.loading.show({ message: 'Creating user...' })
    const { error } = await supabase.functions.invoke('create-user', {
      body: addUserForm.value
    })
    
    if (error) throw new Error(error.message || 'Failed to create user')

    $q.notify({ type: 'positive', message: `User ${addUserForm.value.name} created!`, position: 'top-right' })
    addUserForm.value = { name: '', email: '', password: '', role: 'Student' }
    addUserDialog.value = false
    await loadUsers()
  } catch (err) {
    console.error(err)
    $q.notify({ type: 'negative', message: err.message || 'Unknown error', position: 'top-right' })
  } finally {
    $q.loading.hide()
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
