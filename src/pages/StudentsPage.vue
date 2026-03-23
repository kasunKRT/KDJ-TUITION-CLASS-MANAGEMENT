<template>
  <q-page class="q-pa-lg bg-grey-1" style="min-height: 100vh;">
    <!-- Page Header -->
    <div class="row items-center justify-between q-mb-xl">
      <div>
        <div class="text-h4 text-weight-bolder text-dark" style="letter-spacing: -0.5px;">Students & Enrollments</div>
        <div class="text-subtitle1 text-grey-6 q-mt-xs">Manage student profiles, enrollments, and ID cards.</div>
      </div>
      <q-btn unelevated color="primary" icon="person_add" label="Register Student" class="text-weight-bold shadow-2" style="border-radius: 8px;" @click="openAddStudentDialog" />
    </div>

    <!-- Students DataTable -->
    <q-card class="bg-white shadow-2" style="border-radius: 16px; border: 1px solid #f0f0f0;">
      <div class="row items-center justify-between q-pa-lg" style="border-bottom: 1px solid #f5f5f5;">
        <q-input v-model="search" dense outlined rounded placeholder="Search by name, ID or email..." color="primary" style="width: 320px;">
          <template v-slot:prepend>
            <q-icon name="search" color="grey-6" />
          </template>
        </q-input>
      </div>

      <q-table :rows="filteredStudents" :columns="columns" row-key="id" :loading="loading" flat :rows-per-page-options="[10, 20, 50]" class="custom-table">
        <!-- Student Info -->
        <template v-slot:body-cell-student="props">
          <q-td :props="props">
            <div class="row items-center q-gutter-md no-wrap">
              <q-avatar size="40px" color="blue-1" text-color="primary" class="text-weight-bold shadow-1">
                {{ props.row.name.charAt(0).toUpperCase() }}
              </q-avatar>
              <div class="column">
                <span class="text-weight-bold text-dark" style="font-size: 0.95rem;">{{ props.row.name }}</span>
                <span class="text-caption text-grey-6">{{ props.row.email }}</span>
              </div>
            </div>
          </q-td>
        </template>

        <!-- Student ID -->
        <template v-slot:body-cell-idNumber="props">
          <q-td :props="props">
            <q-badge v-if="props.row.student_id_number" color="grey-2" text-color="dark" class="text-weight-bold q-px-sm q-py-xs" style="border-radius: 6px; border: 1px dashed #ccc;">
              {{ props.row.student_id_number }}
            </q-badge>
            <span v-else class="text-grey-5 italic text-caption">Not set</span>
          </q-td>
        </template>

        <!-- Enrollments -->
        <template v-slot:body-cell-enrollments="props">
          <q-td :props="props">
            <div class="q-gutter-xs">
              <q-badge v-for="e in props.row.enrollments" :key="e.id" color="teal-1" text-color="teal-9" class="q-px-sm" style="border-radius: 6px;">
                {{ e.classes?.name || 'Unknown Class' }}
              </q-badge>
              <span v-if="!props.row.enrollments || props.row.enrollments.length === 0" class="text-grey-5 italic text-caption">Not enrolled</span>
            </div>
          </q-td>
        </template>

        <!-- Phone -->
        <template v-slot:body-cell-phone="props">
          <q-td :props="props">
            <div class="text-dark">{{ props.row.phone || 'N/A' }}</div>
          </q-td>
        </template>

        <!-- Actions -->
        <template v-slot:body-cell-actions="props">
          <q-td :props="props" class="text-right">
            <q-btn flat round dense icon="more_vert" color="grey-7">
              <q-menu anchor="bottom right" self="top right" style="border-radius: 10px; min-width: 180px;">
                <q-list class="q-py-sm">
                  <q-item clickable v-close-popup class="action-menu-item" @click="openViewIdDialog(props.row)">
                    <q-item-section avatar><q-icon name="branding_watermark" color="warning" size="sm"/></q-item-section>
                    <q-item-section class="text-weight-medium">View ID Card</q-item-section>
                  </q-item>
                  <q-item clickable v-close-popup class="action-menu-item" @click="openEditProfileDialog(props.row)">
                    <q-item-section avatar><q-icon name="person" color="primary" size="sm"/></q-item-section>
                    <q-item-section class="text-weight-medium">Edit Profile</q-item-section>
                  </q-item>
                  <q-item clickable v-close-popup class="action-menu-item" @click="openEnrollmentDialog(props.row)">
                    <q-item-section avatar><q-icon name="class" color="secondary" size="sm"/></q-item-section>
                    <q-item-section class="text-weight-medium">Manage Classes</q-item-section>
                  </q-item>
                </q-list>
              </q-menu>
            </q-btn>
          </q-td>
        </template>
        
        <template v-slot:no-data>
          <div class="full-width column flex-center q-py-xl text-grey-5">
            <q-icon name="badge" size="64px" class="q-mb-md opacity-50" />
            <div class="text-h6 text-weight-bold">No students found</div>
            <div class="text-caption q-mt-xs">Register a new student to see them here</div>
          </div>
        </template>
      </q-table>
    </q-card>

    <!-- Dialogs -->

    <!-- Add New Student Dialog -->
    <q-dialog v-model="addStudentDialog" backdrop-filter="blur(4px)">
      <q-card style="min-width: 500px; border-radius: 16px;">
        <q-card-section class="q-pa-xl">
          <div class="text-h6 text-weight-bolder text-dark q-mb-xs">Register Student</div>
          <div class="text-caption text-grey-6 q-mb-xl">Create student account and baseline profile.</div>
          <q-form @submit.prevent="saveNewStudent">
            <div class="row q-col-gutter-y-md q-col-gutter-x-md">
              <div class="col-12">
                <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Full Name</div>
                <q-input v-model="addStudentForm.name" outlined dense color="primary" required />
              </div>
              <div class="col-12 col-md-6">
                <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Email Address</div>
                <q-input v-model="addStudentForm.email" outlined dense color="primary" required />
              </div>
              <div class="col-12 col-md-6">
                <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Login Password</div>
                <q-input v-model="addStudentForm.password" type="password" outlined dense color="primary" required />
              </div>
              <div class="col-12 col-md-6">
                <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Student ID (Custom)</div>
                <q-input v-model="addStudentForm.student_id_number" outlined dense color="primary" placeholder="Leave empty to auto-generate" />
              </div>
              <div class="col-12 col-md-6">
                <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Phone Number</div>
                <q-input v-model="addStudentForm.phone" outlined dense color="primary" />
              </div>
              <div class="col-12 row justify-end q-gutter-sm q-pt-md">
                <q-btn flat label="Cancel" color="grey-7" class="text-weight-bold" v-close-popup />
                <q-btn unelevated label="Register Student" color="primary" type="submit" class="text-weight-bold q-px-lg shadow-2" style="border-radius: 8px;" />
              </div>
            </div>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>

    <!-- Edit Profile Dialog -->
    <q-dialog v-model="editProfileDialog" backdrop-filter="blur(4px)">
      <q-card style="min-width: 550px; border-radius: 16px;">
        <q-card-section class="q-pa-xl">
          <div class="text-h6 text-weight-bolder text-dark q-mb-xs">Edit Student Profile</div>
          <div class="text-caption text-grey-6 q-mb-xl">Update extended details for {{ activeStudent?.name }}</div>
          <q-form @submit.prevent="saveStudentProfile">
            <div class="row q-col-gutter-md q-pt-sm">
              <div class="col-12 col-md-6">
                <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Student ID Number</div>
                <q-input v-model="profileForm.student_id_number" outlined dense color="primary" />
              </div>
              <div class="col-12 col-md-6">
                <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Student Phone</div>
                <q-input v-model="profileForm.phone" outlined dense color="primary" />
              </div>
              <div class="col-12">
                <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">School Attending</div>
                <q-input v-model="profileForm.school" outlined dense color="primary" />
              </div>
              <div class="col-12">
                <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Home Address</div>
                <q-input v-model="profileForm.address" type="textarea" rows="2" outlined dense color="primary" />
              </div>
              <div class="col-12 col-md-6">
                <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Parent/Guardian Name</div>
                <q-input v-model="profileForm.parent_name" outlined dense color="primary" />
              </div>
              <div class="col-12 col-md-6">
                <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Parent/Guardian Phone</div>
                <q-input v-model="profileForm.parent_phone" outlined dense color="primary" />
              </div>
              <div class="col-12 row justify-end q-gutter-sm q-pt-md">
                <q-btn flat label="Cancel" color="grey-7" class="text-weight-bold" v-close-popup />
                <q-btn unelevated label="Save Profile" color="primary" type="submit" class="text-weight-bold q-px-lg shadow-2" style="border-radius: 8px;" />
              </div>
            </div>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>

    <!-- Manage Enrollments Dialog -->
    <q-dialog v-model="enrollmentDialog" backdrop-filter="blur(4px)">
      <q-card style="min-width: 480px; border-radius: 16px;">
        <q-card-section class="q-pa-xl">
          <div class="text-h6 text-weight-bolder text-dark q-mb-xs">Class Enrollments</div>
          <div class="text-caption text-grey-6 q-mb-md">Manage classes for {{ activeStudent?.name }}</div>
          
          <div class="q-mb-lg bg-teal-1 q-pa-md" style="border-radius: 8px;">
            <div class="text-subtitle2 text-teal-9 text-weight-bold q-mb-sm">Current Classes</div>
            <div v-if="activeStudent?.enrollments?.length > 0" class="q-gutter-sm">
              <q-chip
                v-for="e in activeStudent.enrollments" :key="e.id"
                removable @remove="unenrollClass(e.id)"
                color="white" text-color="dark" class="shadow-1"
              >
                {{ e.classes?.name }}
              </q-chip>
            </div>
            <div v-else class="text-caption text-teal-8">No active enrollments.</div>
          </div>

          <q-form @submit.prevent="enrollInClass" class="q-gutter-md">
            <div>
              <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Enroll into new Class</div>
              <div class="row q-gutter-sm items-center no-wrap">
                <q-select v-model="selectedClassToEnroll" :options="availableClassOptions" label="Select a Class" emit-value map-options outlined dense color="primary" class="col" />
                <q-btn unelevated icon="add" color="primary" type="submit" class="shadow-1" style="border-radius: 8px; height: 40px;" :disable="!selectedClassToEnroll" />
              </div>
            </div>
          </q-form>

        </q-card-section>
        <q-card-actions align="right" class="q-px-xl q-pb-xl q-pt-none">
          <q-btn flat label="Close" color="grey-7" class="text-weight-bold" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <!-- View ID Card Dialog -->
    <q-dialog v-model="idCardDialog">
      <q-card style="width: 320px; border-radius: 16px; overflow: hidden;" class="shadow-4">
        <div class="bg-primary text-white q-pa-md text-center">
          <div class="text-h6 text-weight-bolder" style="letter-spacing: 1px;">STUDENT ID</div>
          <div class="text-caption opacity-80">ClassMaster System</div>
        </div>
        <q-card-section class="q-pa-lg column items-center">
          <q-avatar size="90px" color="blue-1" text-color="primary" class="text-weight-bold shadow-1 q-mb-md" style="font-size: 2rem;">
            {{ activeStudent?.name?.charAt(0).toUpperCase() }}
          </q-avatar>
          <div class="text-h6 text-weight-bold text-center text-dark">{{ activeStudent?.name }}</div>
          <div class="text-caption text-grey-6 q-mb-md text-center">{{ activeStudent?.email }}</div>
          
          <div class="full-width q-px-md q-py-sm bg-grey-2 text-center q-mb-lg" style="border-radius: 8px; border: 1px dashed #ccc;">
            <div class="text-caption text-weight-bold text-grey-8">ID NUMBER</div>
            <div class="text-subtitle1 text-weight-bold text-dark" style="letter-spacing: 2px;">{{ activeStudent?.student_id_number || 'N/A' }}</div>
          </div>

          <!-- Dummy QR Code implementation using external API (QRServer) -->
          <img v-if="activeStudent?.student_id_number" :src="`https://api.qrserver.com/v1/create-qr-code/?size=120x120&color=212b36&data=${encodeURIComponent(activeStudent.student_id_number)}`" alt="QR Code" class="shadow-1" style="border-radius: 8px; padding: 4px; background: white;" />
          <div v-else class="text-caption text-negative text-center">Please assign a Student ID Number to generate QR.</div>

        </q-card-section>
        <q-card-actions align="center" class="bg-grey-1 q-py-sm">
          <q-btn flat label="Close" color="grey-8" v-close-popup />
          <q-btn flat label="Print" color="primary" disable />
        </q-card-actions>
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

const students = ref([])
const globalClasses = ref([])

const columns = [
  { name: 'student', label: 'Student Info', field: 'name', align: 'left', sortable: true },
  { name: 'idNumber', label: 'ID Number', field: 'student_id_number', align: 'left', sortable: true },
  { name: 'phone', label: 'Phone', field: 'phone', align: 'left' },
  { name: 'enrollments', label: 'Enrolled Classes', field: 'enrollments', align: 'left' },
  { name: 'actions', label: '', field: 'actions', align: 'right' },
]

const loadData = async () => {
  loading.value = true
  try {
    const [usersRes, profilesRes, enrollsRes, classesRes] = await Promise.all([
      supabase.functions.invoke('list-users', { method: 'GET' }),
      supabase.from('student_profiles').select('*'),
      supabase.from('student_enrollments').select('*, classes(name)'),
      supabase.from('classes').select('id, name')
    ])

    if (usersRes.error) throw new Error(usersRes.error.message || 'Error fetching users')
    
    globalClasses.value = classesRes.data || []
    
    const allUsers = usersRes.data.users || []
    const studentsRaw = allUsers.filter(u => u.role === 'Student')
    const profiles = profilesRes.data || []
    const enrollments = enrollsRes.data || []

    students.value = studentsRaw.map(u => {
      const profile = profiles.find(p => p.id === u.id) || {}
      const myEnrolls = enrollments.filter(e => e.student_id === u.id)
      return {
        ...u,
        ...profile,
        enrollments: myEnrolls
      }
    })
  } catch (err) {
    console.error(err)
    $q.notify({ type: 'negative', message: 'Failed to load students: ' + (err.message || ''), position: 'top-right' })
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  loadData()
})

const filteredStudents = computed(() => {
  const term = search.value.toLowerCase()
  if (!term) return students.value
  return students.value.filter(s => 
    s.name.toLowerCase().includes(term) ||
    s.email.toLowerCase().includes(term) ||
    (s.student_id_number && s.student_id_number.toLowerCase().includes(term))
  )
})

// Add Student Logic
const addStudentDialog = ref(false)
const addStudentForm = ref({ name: '', email: '', password: '', student_id_number: '', phone: '' })

const openAddStudentDialog = () => {
  addStudentForm.value = { name: '', email: '', password: '', student_id_number: '', phone: '' }
  addStudentDialog.value = true
}

const saveNewStudent = async () => {
  try {
    $q.loading.show()
    // 1. Create Auth User
    const { data: userData, error: fnError } = await supabase.functions.invoke('create-user', {
      body: { 
        name: addStudentForm.value.name, 
        email: addStudentForm.value.email, 
        password: addStudentForm.value.password, 
        role: 'Student' 
      }
    })
    if (fnError) throw new Error(fnError.message || 'Error creating auth user')
    if (userData.error) throw new Error(userData.error)

    const userId = userData.user.id
    
    // Auto-generate ID if empty
    const idNumber = addStudentForm.value.student_id_number || `STU${new Date().getFullYear()}${Math.floor(1000 + Math.random() * 9000)}`

    // 2. Insert Profile
    const { error: profileError } = await supabase.from('student_profiles').insert([{
      id: userId,
      student_id_number: idNumber,
      phone: addStudentForm.value.phone
    }])
    if (profileError) throw profileError

    $q.notify({ type: 'positive', message: 'Student registered successfully', position: 'top-right' })
    addStudentDialog.value = false
    await loadData()
  } catch (err) {
    $q.notify({ type: 'negative', message: err.message, position: 'top-right' })
  } finally {
    $q.loading.hide()
  }
}

// Edit Profile Logic
const editProfileDialog = ref(false)
const activeStudent = ref(null)
const profileForm = ref({ id: '', student_id_number: '', phone: '', address: '', parent_name: '', parent_phone: '', school: '' })

const openEditProfileDialog = (student) => {
  activeStudent.value = student
  profileForm.value = {
    id: student.id,
    student_id_number: student.student_id_number || '',
    phone: student.phone || '',
    address: student.address || '',
    parent_name: student.parent_name || '',
    parent_phone: student.parent_phone || '',
    school: student.school || ''
  }
  editProfileDialog.value = true
}

const saveStudentProfile = async () => {
  try {
    $q.loading.show()
    // Check if profile exists; Upsert is handled nicely by just checking if it exists or using an upsert option.
    // However Supabase insert with upsert: true requires careful config, so we just use upsert.
    const payload = { ...profileForm.value }
    const { error } = await supabase.from('student_profiles').upsert(payload, { onConflict: 'id' })
    if (error) throw error

    $q.notify({ type: 'positive', message: 'Profile updated', position: 'top-right' })
    editProfileDialog.value = false
    await loadData()
  } catch (err) {
    $q.notify({ type: 'negative', message: err.message, position: 'top-right' })
  } finally {
    $q.loading.hide()
  }
}

// Class Enrollment
const enrollmentDialog = ref(false)
const selectedClassToEnroll = ref(null)

const availableClassOptions = computed(() => {
  if (!activeStudent.value) return []
  const enrolledIds = activeStudent.value.enrollments.map(e => e.class_id)
  return globalClasses.value
    .filter(c => !enrolledIds.includes(c.id))
    .map(c => ({ label: c.name, value: c.id }))
})

const openEnrollmentDialog = (student) => {
  activeStudent.value = student
  selectedClassToEnroll.value = null
  enrollmentDialog.value = true
}

const enrollInClass = async () => {
  if (!selectedClassToEnroll.value) return
  try {
    $q.loading.show()
    const payload = {
      student_id: activeStudent.value.id,
      class_id: selectedClassToEnroll.value
    }
    const { error } = await supabase.from('student_enrollments').insert([payload])
    if (error) throw error
    
    $q.notify({ type: 'positive', message: 'Enrolled successfully', position: 'top-right' })
    selectedClassToEnroll.value = null
    await loadData()
    // Update local active student reference to immediately reflect changes in dialog
    activeStudent.value = students.value.find(s => s.id === activeStudent.value.id)
  } catch (err) {
    if(err.code === '23505') {
       $q.notify({ type: 'warning', message: 'Already enrolled in this class', position: 'top-right'})
    } else {
       $q.notify({ type: 'negative', message: err.message, position: 'top-right' })
    }
  } finally {
    $q.loading.hide()
  }
}

const unenrollClass = async (enrollmentId) => {
  try {
    $q.loading.show()
    const { error } = await supabase.from('student_enrollments').delete().eq('id', enrollmentId)
    if (error) throw error
    
    $q.notify({ type: 'positive', message: 'Unenrolled', position: 'top-right' })
    await loadData()
    activeStudent.value = students.value.find(s => s.id === activeStudent.value.id)
  } catch (err) {
    $q.notify({ type: 'negative', message: err.message, position: 'top-right' })
  } finally {
    $q.loading.hide()
  }
}

// ID Card Dialog
const idCardDialog = ref(false)

const openViewIdDialog = (student) => {
  activeStudent.value = student
  idCardDialog.value = true
}

</script>

<style scoped>
:deep(.custom-table .q-table__top),
:deep(.custom-table .q-table__bottom) {
  padding: 16px 20px;
}
:deep(.custom-table thead th) {
  font-weight: 700;
  color: #637381;
  font-size: 0.8rem;
  letter-spacing: 0.5px;
  text-transform: uppercase;
  background: #f9fafb;
  border-bottom: 1px solid #f0f0f0;
}
:deep(.custom-table tbody td) {
  border-bottom: 1px solid #f9fafb;
  padding: 14px 20px;
}
:deep(.custom-table tbody tr:hover td) {
  background: #f4f6f8;
}
:deep(.q-field--outlined .q-field__control) {
  border-radius: 8px; 
}
.action-menu-item {
  border-radius: 6px;
  transition: background 0.2s;
}
.action-menu-item:hover {
  background: #f4f6f8;
}
</style>
