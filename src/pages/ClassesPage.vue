<template>
  <q-page class="q-pa-lg bg-grey-1" style="min-height: 100vh;">
    <!-- Page Header -->
    <div class="row items-center justify-between q-mb-xl">
      <div>
        <div class="text-h4 text-weight-bolder text-dark" style="letter-spacing: -0.5px;">Classes & Subjects</div>
        <div class="text-subtitle1 text-grey-6 q-mt-xs">Manage class batches, schedules, and subjects.</div>
      </div>
    </div>

    <!-- Tabs for Classes and Subjects -->
    <q-card class="bg-white shadow-2" style="border-radius: 16px; border: 1px solid #f0f0f0;">
      <q-tabs
        v-model="activeTab"
        dense
        class="text-grey-7 q-pt-sm"
        active-color="primary"
        indicator-color="primary"
        align="left"
        narrow-indicator
      >
        <q-tab name="classes" label="Class Batches" icon="class" class="text-weight-bold" />
        <q-tab name="subjects" label="Subjects" icon="book" class="text-weight-bold" />
      </q-tabs>

      <q-separator />

      <q-tab-panels v-model="activeTab" animated>
        <!-- Classes Panel -->
        <q-tab-panel name="classes" class="q-pa-none">
          <div class="row items-center justify-between q-pa-lg" style="border-bottom: 1px solid #f5f5f5;">
            <q-input v-model="searchClass" dense outlined rounded placeholder="Search classes..." color="primary" style="width: 300px;">
              <template v-slot:prepend>
                <q-icon name="search" color="grey-6" />
              </template>
            </q-input>
            <q-btn unelevated color="primary" icon="add" label="Add Class Batch" class="text-weight-bold shadow-2" style="border-radius: 8px;" @click="openAddClassDialog" />
          </div>
          
          <q-table
            :rows="filteredClasses"
            :columns="classColumns"
            row-key="id"
            :loading="loading"
            flat
            :rows-per-page-options="[10, 20, 50]"
            class="custom-table"
          >
            <!-- Teacher Column -->
            <template v-slot:body-cell-teacher="props">
              <q-td :props="props">
                <div class="row items-center q-gutter-sm no-wrap" v-if="props.row.teacher_name">
                  <q-avatar size="32px" color="teal-1" text-color="teal" class="text-weight-bold shadow-1" style="font-size: 0.8rem;">
                    {{ props.row.teacher_name.charAt(0).toUpperCase() }}
                  </q-avatar>
                  <span class="text-weight-medium text-dark">{{ props.row.teacher_name }}</span>
                </div>
                <div v-else class="text-grey-5 italic">Unassigned</div>
              </q-td>
            </template>
            <!-- Subject Column -->
            <template v-slot:body-cell-subject="props">
              <q-td :props="props">
                <q-badge color="blue-1" text-color="primary" class="text-weight-bold q-px-sm q-py-xs" style="border-radius: 6px;">
                  {{ props.row.subject_name }}
                </q-badge>
              </q-td>
            </template>
            <!-- Schedule Column -->
            <template v-slot:body-cell-schedule="props">
              <q-td :props="props">
                <div class="row items-center text-grey-8">
                  <q-icon name="schedule" size="xs" class="q-mr-xs" v-if="props.row.schedule_day" />
                  {{ props.row.schedule_day }}
                  <span v-if="props.row.start_time" class="q-ml-sm text-weight-medium text-primary">
                    ({{ props.row.start_time }} <span v-if="props.row.end_time">- {{ props.row.end_time }}</span>)
                  </span>
                </div>
              </q-td>
            </template>
            <!-- Actions -->
            <template v-slot:body-cell-actions="props">
              <q-td :props="props" class="text-right">
                <q-btn flat round dense icon="more_vert" color="grey-7">
                  <q-menu anchor="bottom right" self="top right" style="border-radius: 10px; min-width: 140px;">
                    <q-list class="q-py-sm">
                      <q-item clickable v-close-popup class="action-menu-item" @click="openEditClassDialog(props.row)">
                        <q-item-section avatar><q-icon name="edit" color="primary" size="sm"/></q-item-section>
                        <q-item-section class="text-weight-medium">Edit Class</q-item-section>
                      </q-item>
                      <q-separator class="q-my-xs" />
                      <q-item clickable v-close-popup class="action-menu-item" @click="deleteClass(props.row)">
                        <q-item-section avatar><q-icon name="delete" color="negative" size="sm"/></q-item-section>
                        <q-item-section class="text-negative text-weight-bold">Delete</q-item-section>
                      </q-item>
                    </q-list>
                  </q-menu>
                </q-btn>
              </q-td>
            </template>
            <template v-slot:no-data>
              <div class="full-width column flex-center q-py-xl text-grey-5">
                <q-icon name="class" size="64px" class="q-mb-md opacity-50" />
                <div class="text-h6 text-weight-bold">No classes found</div>
                <div class="text-caption q-mt-xs">Add a new class to get started</div>
              </div>
            </template>
          </q-table>
        </q-tab-panel>

        <!-- Subjects Panel -->
        <q-tab-panel name="subjects" class="q-pa-none">
          <div class="row items-center justify-between q-pa-lg" style="border-bottom: 1px solid #f5f5f5;">
            <q-input v-model="searchSubject" dense outlined rounded placeholder="Search subjects..." color="primary" style="width: 300px;">
              <template v-slot:prepend>
                <q-icon name="search" color="grey-6" />
              </template>
            </q-input>
            <q-btn unelevated color="primary" icon="add" label="Add Subject" class="text-weight-bold shadow-2" style="border-radius: 8px;" @click="openAddSubjectDialog" />
          </div>
          <q-table
            :rows="filteredSubjects"
            :columns="subjectColumns"
            row-key="id"
            :loading="loading"
            flat
            :rows-per-page-options="[10, 20, 50]"
            class="custom-table"
          >
            <!-- Actions -->
            <template v-slot:body-cell-actions="props">
              <q-td :props="props" class="text-right">
                <q-btn flat round dense icon="more_vert" color="grey-7">
                  <q-menu anchor="bottom right" self="top right" style="border-radius: 10px; min-width: 140px;">
                    <q-list class="q-py-sm">
                      <q-item clickable v-close-popup class="action-menu-item" @click="openEditSubjectDialog(props.row)">
                        <q-item-section avatar><q-icon name="edit" color="primary" size="sm"/></q-item-section>
                        <q-item-section class="text-weight-medium">Edit Subject</q-item-section>
                      </q-item>
                      <q-separator class="q-my-xs" />
                      <q-item clickable v-close-popup class="action-menu-item" @click="deleteSubject(props.row)">
                        <q-item-section avatar><q-icon name="delete" color="negative" size="sm"/></q-item-section>
                        <q-item-section class="text-negative text-weight-bold">Delete</q-item-section>
                      </q-item>
                    </q-list>
                  </q-menu>
                </q-btn>
              </q-td>
            </template>
            <template v-slot:no-data>
              <div class="full-width column flex-center q-py-xl text-grey-5">
                <q-icon name="style" size="64px" class="q-mb-md opacity-50" />
                <div class="text-h6 text-weight-bold">No subjects found</div>
                <div class="text-caption q-mt-xs">Add a new subject (e.g., Physics, Maths)</div>
              </div>
            </template>
          </q-table>
        </q-tab-panel>
      </q-tab-panels>
    </q-card>

    <!-- Dialogs -->

    <!-- Add/Edit Subject Dialog -->
    <q-dialog v-model="subjectDialog" backdrop-filter="blur(4px)">
      <q-card style="min-width: 400px; border-radius: 16px;">
        <q-card-section class="q-pa-xl">
          <div class="text-h6 text-weight-bolder text-dark q-mb-xs">{{ isEditSubject ? 'Edit Subject' : 'Add New Subject' }}</div>
          <q-form @submit.prevent="saveSubject" class="q-gutter-md q-mt-md">
            <div>
              <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Subject Name</div>
              <q-input v-model="subjectForm.name" outlined dense color="primary" placeholder="e.g. A/L Physics"  required />
            </div>
            <div>
              <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Description</div>
              <q-input v-model="subjectForm.description" type="textarea" outlined dense color="primary" placeholder="Brief description regarding the subject..." rows="3" />
            </div>
            <div class="row justify-end q-gutter-sm q-pt-md">
              <q-btn flat label="Cancel" color="grey-7" class="text-weight-bold" v-close-popup />
              <q-btn unelevated :label="isEditSubject ? 'Save Changes' : 'Create Subject'" color="primary" type="submit" class="text-weight-bold q-px-lg shadow-2" style="border-radius: 8px;" />
            </div>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>

    <!-- Add/Edit Class Dialog -->
    <q-dialog v-model="classDialog" backdrop-filter="blur(4px)">
      <q-card style="min-width: 460px; border-radius: 16px;">
        <q-card-section class="q-pa-xl">
          <div class="text-h6 text-weight-bolder text-dark q-mb-xs">{{ isEditClass ? 'Edit Class Batch' : 'Add New Class Batch' }}</div>
          <q-form @submit.prevent="saveClass">
            <div class="row q-col-gutter-y-md q-col-gutter-x-md q-pt-sm">
              <div class="col-12">
                <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Batch Name</div>
                <q-input v-model="classForm.name" outlined dense color="primary" placeholder="e.g. 2026 Model Papers" required />
              </div>
              <div class="col-12 col-md-6">
                <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Subject</div>
                <q-select v-model="classForm.subject_id" :options="subjectOptions" emit-value map-options outlined dense color="primary" required />
              </div>
              <div class="col-12 col-md-6">
                <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Teacher</div>
                <q-select v-model="classForm.teacher_id" :options="teacherOptions" emit-value map-options outlined dense color="primary" clearable />
              </div>
              <div class="col-12 col-md-4">
                <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Schedule Day</div>
                <q-select v-model="classForm.schedule_day" :options="days" outlined dense color="primary" clearable />
              </div>
              <div class="col-6 col-md-4">
                <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">Start Time</div>
                <q-input v-model="classForm.start_time" type="time" outlined dense color="primary" />
              </div>
              <div class="col-6 col-md-4">
                <div class="text-subtitle2 text-weight-bold text-dark q-mb-sm">End Time</div>
                <q-input v-model="classForm.end_time" type="time" outlined dense color="primary" />
              </div>
              <div class="col-12 row justify-end q-gutter-sm q-pt-md">
                <q-btn flat label="Cancel" color="grey-7" class="text-weight-bold" v-close-popup />
                <q-btn unelevated :label="isEditClass ? 'Save Changes' : 'Create Class'" color="primary" type="submit" class="text-weight-bold q-px-lg shadow-2" style="border-radius: 8px;" />
              </div>
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

// State
const activeTab = ref('classes')
const loading = ref(false)
const searchClass = ref('')
const searchSubject = ref('')

const subjects = ref([])
const classes = ref([])
const teachers = ref([])

// Columns
const subjectColumns = [
  { name: 'name', label: 'Subject Name', field: 'name', align: 'left', sortable: true },
  { name: 'description', label: 'Description', field: 'description', align: 'left' },
  { name: 'actions', label: '', field: 'actions', align: 'right' },
]

const classColumns = [
  { name: 'name', label: 'Batch Name', field: 'name', align: 'left', sortable: true },
  { name: 'subject', label: 'Subject', field: 'subject_name', align: 'left', sortable: true },
  { name: 'teacher', label: 'Assigned Teacher', field: 'teacher_name', align: 'left', sortable: true },
  { name: 'schedule', label: 'Schedule Info', field: 'schedule', align: 'left' },
  { name: 'actions', label: '', field: 'actions', align: 'right' },
]

const days = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday']

// Load Data
const loadData = async () => {
  loading.value = true
  try {
    // 1. Fetch Subjects
    const { data: subs, error: subErr } = await supabase.from('subjects').select('*').order('created_at', { ascending: false })
    if (subErr) throw subErr
    subjects.value = subs

    // 2. Fetch Teachers via existing Edge function "list-users"
    const { data: usersData, error: uErr } = await supabase.functions.invoke('list-users', { method: 'GET' })
    if (uErr) {
       console.error('Failed to list teachers', uErr)
    } else {
       teachers.value = usersData.users.filter(u => u.role === 'Teacher' || u.role === 'Admin')
    }

    // 3. Fetch Classes
    const { data: cls, error: clsErr } = await supabase.from('classes').select('*').order('created_at', { ascending: false })
    if (clsErr) throw clsErr
    
    // Map associations
    classes.value = cls.map(c => {
      const subject = subjects.value.find(s => s.id === c.subject_id)
      const teacher = teachers.value.find(t => t.id === c.teacher_id)
      return {
        ...c,
        subject_name: subject ? subject.name : 'Unknown',
        teacher_name: teacher ? teacher.name : null
      }
    })

  } catch (err) {
    console.error(err)
    $q.notify({ type: 'negative', message: 'Failed to load data: ' + (err.message || ''), position: 'top-right' })
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  loadData()
})

// Filters
const filteredSubjects = computed(() => {
  return subjects.value.filter(s => s.name.toLowerCase().includes(searchSubject.value.toLowerCase()))
})

const filteredClasses = computed(() => {
  return classes.value.filter(c => 
    c.name.toLowerCase().includes(searchClass.value.toLowerCase()) ||
    c.subject_name.toLowerCase().includes(searchClass.value.toLowerCase()) ||
    (c.teacher_name && c.teacher_name.toLowerCase().includes(searchClass.value.toLowerCase()))
  )
})

const subjectOptions = computed(() => {
  return subjects.value.map(s => ({ label: s.name, value: s.id }))
})

const teacherOptions = computed(() => {
  return teachers.value.map(t => ({ label: t.name, value: t.id }))
})

// Subject Management
const subjectDialog = ref(false)
const isEditSubject = ref(false)
const subjectForm = ref({ id: null, name: '', description: '' })

const openAddSubjectDialog = () => {
  isEditSubject.value = false
  subjectForm.value = { id: null, name: '', description: '' }
  subjectDialog.value = true
}

const openEditSubjectDialog = (row) => {
  isEditSubject.value = true
  subjectForm.value = { ...row }
  subjectDialog.value = true
}

const saveSubject = async () => {
  try {
    $q.loading.show()
    const payload = { name: subjectForm.value.name, description: subjectForm.value.description }
    if (isEditSubject.value) {
      const { error } = await supabase.from('subjects').update(payload).eq('id', subjectForm.value.id)
      if (error) throw error
    } else {
      const { error } = await supabase.from('subjects').insert([payload])
      if (error) throw error
    }
    await loadData()
    subjectDialog.value = false
    $q.notify({ type: 'positive', message: 'Subject saved successfully', position: 'top-right' })
  } catch (err) {
    $q.notify({ type: 'negative', message: err.message, position: 'top-right' })
  } finally {
    $q.loading.hide()
  }
}

const deleteSubject = (row) => {
  $q.dialog({
    title: 'Confirm',
    message: `Are you sure you want to delete ${row.name}? All underlying classes will also be deleted.`,
    persistent: true,
    ok: { color: 'negative', label: 'Delete', unelevated: true, style: 'border-radius: 8px;' },
    cancel: { flat: true, color: 'grey-7', style: 'border-radius: 8px;' }
  }).onOk(async () => {
    try {
      $q.loading.show()
      const { error } = await supabase.from('subjects').delete().eq('id', row.id)
      if (error) throw error
      await loadData()
      $q.notify({ type: 'positive', message: 'Subject deleted', position: 'top-right' })
    } catch (err) {
      $q.notify({ type: 'negative', message: err.message, position: 'top-right' })
    } finally {
      $q.loading.hide()
    }
  })
}

// Class Management
const classDialog = ref(false)
const isEditClass = ref(false)
const classForm = ref({ id: null, name: '', subject_id: null, teacher_id: null, schedule_day: null, start_time: '', end_time: '' })

const openAddClassDialog = () => {
  isEditClass.value = false
  classForm.value = { id: null, name: '', subject_id: null, teacher_id: null, schedule_day: null, start_time: '', end_time: '' }
  classDialog.value = true
}

const openEditClassDialog = (row) => {
  isEditClass.value = true
  classForm.value = { ...row }
  classDialog.value = true
}

const saveClass = async () => {
  try {
    $q.loading.show()
    const payload = { 
      name: classForm.value.name,
      subject_id: classForm.value.subject_id,
      teacher_id: classForm.value.teacher_id,
      schedule_day: classForm.value.schedule_day,
      start_time: classForm.value.start_time,
      end_time: classForm.value.end_time
    }

    if (isEditClass.value) {
      const { error } = await supabase.from('classes').update(payload).eq('id', classForm.value.id)
      if (error) throw error
    } else {
      const { error } = await supabase.from('classes').insert([payload])
      if (error) throw error
    }
    await loadData()
    classDialog.value = false
    $q.notify({ type: 'positive', message: 'Class saved successfully', position: 'top-right' })
  } catch (err) {
    $q.notify({ type: 'negative', message: err.message, position: 'top-right' })
  } finally {
    $q.loading.hide()
  }
}

const deleteClass = (row) => {
  $q.dialog({
    title: 'Confirm',
    message: `Are you sure you want to delete the class ${row.name}?`,
    persistent: true,
    ok: { color: 'negative', label: 'Delete', unelevated: true, style: 'border-radius: 8px;' },
    cancel: { flat: true, color: 'grey-7', style: 'border-radius: 8px;' }
  }).onOk(async () => {
    try {
      $q.loading.show()
      const { error } = await supabase.from('classes').delete().eq('id', row.id)
      if (error) throw error
      await loadData()
      $q.notify({ type: 'positive', message: 'Class deleted', position: 'top-right' })
    } catch (err) {
      $q.notify({ type: 'negative', message: err.message, position: 'top-right' })
    } finally {
      $q.loading.hide()
    }
  })
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
  border-radius: 8px; /* Modern border radius */
}
.action-menu-item {
  border-radius: 6px;
  transition: background 0.2s;
}
.action-menu-item:hover {
  background: #f4f6f8;
}
</style>
