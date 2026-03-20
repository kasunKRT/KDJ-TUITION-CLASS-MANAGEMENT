<template>
  <q-page class="q-pa-lg bg-grey-1" style="min-height: 100vh;">
    <!-- Page Header -->
    <div class="row items-center justify-between q-mb-xl">
      <div>
        <div class="text-h4 text-weight-bolder text-dark" style="letter-spacing: -0.5px;">Roles & Permissions</div>
        <div class="text-subtitle1 text-grey-6 q-mt-xs">Define what each role can access and do within the system.</div>
      </div>
      <q-btn
        unelevated color="primary" icon="add" label="New Role"
        class="text-weight-bold q-px-md shadow-2"
        style="border-radius: 8px; text-transform: none;"
        @click="openNewRoleDialog"
      />
    </div>

    <!-- Roles Cards -->
    <div class="row q-col-gutter-lg q-mb-xl">
      <div class="col-12 col-md-4" v-for="role in roles" :key="role.name">
        <q-card class="bg-white shadow-2 role-card" style="border-radius: 16px; border: 1px solid #f0f0f0;">
          <q-card-section class="q-pa-lg">
            <div class="row items-start justify-between no-wrap q-mb-md">
              <div class="row items-center">
                <q-avatar size="48px" :color="role.color + '-1'" :text-color="role.color" style="border-radius: 12px;" class="q-mr-md">
                  <q-icon :name="role.icon" size="26px" />
                </q-avatar>
                <div>
                  <div class="text-h6 text-weight-bolder text-dark">{{ role.name }}</div>
                  <div class="text-caption text-grey-6">{{ role.userCount }} users assigned</div>
                </div>
              </div>
              <q-btn flat round dense icon="more_vert" color="grey-7">
                <q-menu anchor="bottom right" self="top right" style="border-radius: 10px; min-width: 160px;">
                  <q-list class="q-py-sm">
                    <q-item clickable v-close-popup class="action-menu-item" @click="editRole(role)">
                      <q-item-section avatar><q-icon name="edit" color="primary" size="sm"/></q-item-section>
                      <q-item-section class="text-weight-medium">Edit Role</q-item-section>
                    </q-item>
                    <q-item clickable v-close-popup class="action-menu-item text-negative" v-if="!role.system" @click="deleteRole(role)">
                      <q-item-section avatar><q-icon name="delete" color="negative" size="sm"/></q-item-section>
                      <q-item-section class="text-weight-bold text-negative">Delete Role</q-item-section>
                    </q-item>
                  </q-list>
                </q-menu>
              </q-btn>
            </div>
            
            <div class="text-body2 text-grey-7 q-mb-lg">{{ role.description }}</div>

            <!-- Permissions List -->
            <div class="q-gutter-sm">
              <q-badge
                v-for="perm in role.permissions"
                :key="perm"
                color="grey-2"
                text-color="grey-8"
                class="text-caption text-weight-bold q-pa-sm"
                style="border-radius: 6px; font-size: 0.78rem;"
              >
                <q-icon name="check_circle" color="green" size="12px" class="q-mr-xs" />
                {{ perm }}
              </q-badge>
            </div>

            <q-separator class="q-my-lg bg-grey-2" />
            
            <div class="row items-center justify-between">
              <q-badge :color="role.color + '-1'" :text-color="role.color" class="text-weight-bold q-px-sm" style="border-radius: 6px;">
                {{ role.name }}
              </q-badge>
              <q-btn v-if="role.system" flat dense no-caps color="grey-5" icon="lock" label="System Role" class="text-caption" />
              <q-btn v-else flat dense no-caps color="primary" icon="edit" label="Edit Permissions" class="text-caption text-weight-bold" @click="editRole(role)" />
            </div>
          </q-card-section>
        </q-card>
      </div>
    </div>

    <!-- Permissions Matrix Card -->
    <q-card class="bg-white shadow-2" style="border-radius: 16px; border: 1px solid #f0f0f0;">
      <q-card-section class="q-pa-lg">
        <div class="text-h6 text-weight-bolder text-dark q-mb-xs">Permissions Matrix</div>
        <div class="text-caption text-grey-6 q-mb-lg">Overview of all permissions across roles</div>

        <q-table
          :rows="permissionMatrix"
          :columns="matrixColumns"
          row-key="permission"
          flat hide-bottom
          class="matrix-table"
        >
          <template v-slot:body-cell-admin="props">
            <q-td :props="props" class="text-center">
              <q-icon :name="props.row.admin ? 'check_circle' : 'cancel'" :color="props.row.admin ? 'green' : 'red-3'" size="sm" />
            </q-td>
          </template>
          <template v-slot:body-cell-teacher="props">
            <q-td :props="props" class="text-center">
              <q-icon :name="props.row.teacher ? 'check_circle' : 'cancel'" :color="props.row.teacher ? 'green' : 'red-3'" size="sm" />
            </q-td>
          </template>
          <template v-slot:body-cell-student="props">
            <q-td :props="props" class="text-center">
              <q-icon :name="props.row.student ? 'check_circle' : 'cancel'" :color="props.row.student ? 'green' : 'red-3'" size="sm" />
            </q-td>
          </template>
        </q-table>
      </q-card-section>
    </q-card>

    <!-- New/Edit Role Dialog -->
    <q-dialog v-model="roleDialog" backdrop-filter="blur(4px)">
      <q-card style="min-width: 500px; border-radius: 16px;">
        <q-card-section class="q-pa-xl">
          <div class="text-h6 text-weight-bolder text-dark q-mb-xs">{{ isEditingRole ? 'Edit Role' : 'Create New Role' }}</div>
          <div class="text-caption text-grey-6 q-mb-xl">Configure the name and permissions for this role.</div>
          <q-form @submit.prevent="saveRole" class="q-gutter-md">
            <div>
              <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Role Name</div>
              <q-input v-model="roleForm.name" outlined dense color="primary" placeholder="e.g. Supervisor" />
            </div>
            <div>
              <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Description</div>
              <q-input v-model="roleForm.description" outlined dense color="primary" placeholder="Brief description of this role..." />
            </div>
            <div>
              <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Permissions</div>
              <div class="row q-col-gutter-sm">
                <div class="col-6" v-for="perm in allPermissions" :key="perm">
                  <q-checkbox v-model="roleForm.permissions" :val="perm" :label="perm" color="primary" />
                </div>
              </div>
            </div>
            <div class="row justify-end q-gutter-sm q-pt-md">
              <q-btn flat label="Cancel" color="grey-7" class="text-weight-bold" v-close-popup />
              <q-btn unelevated :label="isEditingRole ? 'Save Changes' : 'Create Role'" color="primary" type="submit" class="text-weight-bold q-px-lg shadow-2" style="border-radius: 8px;" />
            </div>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>

  </q-page>
</template>

<script setup>
import { ref } from 'vue'
import { useQuasar } from 'quasar'

const $q = useQuasar()

const roleDialog = ref(false)
const isEditingRole = ref(false)
const roleForm = ref({ name: '', description: '', permissions: [] })

const allPermissions = [
  'View Dashboard', 'Manage Users', 'Manage Roles',
  'View Students', 'Manage Students',
  'View Classes', 'Manage Classes',
  'View Payments', 'Manage Payments',
]

const roles = ref([
  {
    name: 'Admin',
    icon: 'shield',
    color: 'red',
    userCount: 0,
    system: true,
    description: 'Full access to all features and settings. Can manage users, roles, classes, and payments.',
    permissions: ['Manage Users', 'Manage Roles', 'Manage Classes', 'Manage Payments', 'View Dashboard'],
  },
  {
    name: 'Teacher',
    icon: 'school',
    color: 'blue',
    userCount: 0,
    system: true,
    description: 'Can manage assigned classes, view student attendance, and enter exam results.',
    permissions: ['View Students', 'View Classes', 'View Dashboard'],
  },
  {
    name: 'Student',
    icon: 'person',
    color: 'teal',
    userCount: 0,
    system: true,
    description: 'Limited access. Can view their own class schedule, attendance and payments.',
    permissions: ['View Dashboard'],
  },
])

const matrixColumns = [
  { name: 'permission', label: 'Permission', field: 'permission', align: 'left' },
  { name: 'admin', label: 'Admin', field: 'admin', align: 'center' },
  { name: 'teacher', label: 'Teacher', field: 'teacher', align: 'center' },
  { name: 'student', label: 'Student', field: 'student', align: 'center' },
]

const permissionMatrix = ref([
  { permission: 'View Dashboard',   admin: true,  teacher: true,  student: true  },
  { permission: 'Manage Users',     admin: true,  teacher: false, student: false },
  { permission: 'Manage Roles',     admin: true,  teacher: false, student: false },
  { permission: 'View Students',    admin: true,  teacher: true,  student: false },
  { permission: 'Manage Students',  admin: true,  teacher: false, student: false },
  { permission: 'View Classes',     admin: true,  teacher: true,  student: true  },
  { permission: 'Manage Classes',   admin: true,  teacher: false, student: false },
  { permission: 'View Payments',    admin: true,  teacher: false, student: true  },
  { permission: 'Manage Payments',  admin: true,  teacher: false, student: false },
])

const openNewRoleDialog = () => {
  isEditingRole.value = false
  roleForm.value = { name: '', description: '', permissions: [] }
  roleDialog.value = true
}

const editRole = (role) => {
  isEditingRole.value = true
  roleForm.value = { name: role.name, description: role.description, permissions: [...role.permissions] }
  roleDialog.value = true
}

const saveRole = () => {
  $q.notify({ type: 'positive', message: `Role "${roleForm.value.name}" saved!`, position: 'top-right' })
  roleDialog.value = false
}

const deleteRole = (role) => {
  $q.dialog({
    title: 'Delete Role',
    message: `Are you sure you want to delete the "${role.name}" role?`,
    cancel: true,
    ok: { label: 'Delete', color: 'negative', flat: false },
  }).onOk(() => {
    const idx = roles.value.findIndex(r => r.name === role.name)
    if (idx !== -1) roles.value.splice(idx, 1)
    $q.notify({ type: 'positive', message: `Role "${role.name}" deleted.`, position: 'top-right' })
  })
}
</script>

<style scoped>
.role-card {
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}
.role-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 24px -10px rgba(0, 0, 0, 0.1) !important;
}
:deep(.matrix-table thead th) {
  font-weight: 700;
  color: #637381;
  font-size: 0.8rem;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  background: #f9fafb;
  border-bottom: 1px solid #f0f0f0;
}
:deep(.matrix-table tbody td) {
  border-bottom: 1px solid #f9fafb;
  padding: 12px 20px;
}
:deep(.matrix-table tbody tr:hover td) {
  background: #f4f6f8;
}
.action-menu-item {
  border-radius: 6px;
  transition: background 0.2s;
}
.action-menu-item:hover {
  background: #f4f6f8;
}
</style>
