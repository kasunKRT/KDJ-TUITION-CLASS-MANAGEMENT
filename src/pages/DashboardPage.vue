<template>
  <q-page class="q-pa-lg bg-grey-1" style="min-height: 100vh;">
    <!-- Dashboard Header -->
    <div class="row items-center justify-between q-mb-xl">
      <div>
        <div class="text-h4 text-weight-bolder text-dark" style="letter-spacing: -0.5px;">Dashboard Overview</div>
        <div class="text-subtitle1 text-grey-6 q-mt-xs">Welcome back, here's what's happening today.</div>
      </div>
      <q-btn
        unelevated
        color="primary"
        icon="add"
        label="New Registration"
        class="text-weight-bold q-px-md shadow-2"
        style="border-radius: 8px; font-size: 0.95rem; text-transform: none;"
      />
    </div>

    <!-- Stats Row -->
    <div class="row q-col-gutter-lg q-mb-xl">
      <!-- Total Students -->
      <div class="col-12 col-md-3">
        <q-card class="stat-card bg-white shadow-2" style="border-radius: 16px; border: 1px solid #f0f0f0;">
          <q-card-section class="q-pa-lg row items-center no-wrap">
            <q-avatar size="64px" color="blue-1" text-color="primary" class="q-mr-md" style="border-radius: 16px;">
              <q-icon name="people" size="36px"/>
            </q-avatar>
            <div class="column">
              <span class="text-subtitle2 text-grey-8 text-weight-bold q-mb-xs">Total Students</span>
              <span class="text-h3 text-weight-bolder text-dark" style="line-height: 1;">{{ stats.totalStudents }}</span>
            </div>
          </q-card-section>
        </q-card>
      </div>

      <!-- Active Classes -->
      <div class="col-12 col-md-3">
        <q-card class="stat-card bg-white shadow-2" style="border-radius: 16px; border: 1px solid #f0f0f0;">
          <q-card-section class="q-pa-lg row items-center no-wrap">
            <q-avatar size="64px" color="orange-1" text-color="orange" class="q-mr-md" style="border-radius: 16px;">
              <q-icon name="class" size="36px"/>
            </q-avatar>
            <div class="column">
              <span class="text-subtitle2 text-grey-8 text-weight-bold q-mb-xs">Active Classes</span>
              <span class="text-h3 text-weight-bolder text-dark" style="line-height: 1;">{{ stats.activeClasses }}</span>
            </div>
          </q-card-section>
        </q-card>
      </div>

      <!-- Today's Earnings -->
      <div class="col-12 col-md-3">
        <q-card class="stat-card bg-white shadow-2" style="border-radius: 16px; border: 1px solid #f0f0f0;">
          <q-card-section class="q-pa-lg row items-center no-wrap">
            <q-avatar size="64px" color="green-1" text-color="green" class="q-mr-md" style="border-radius: 16px;">
              <q-icon name="payments" size="36px"/>
            </q-avatar>
            <div class="column">
              <span class="text-subtitle2 text-grey-8 text-weight-bold q-mb-xs">Today's Earnings</span>
              <span class="text-h3 text-weight-bolder text-dark" style="line-height: 1;"><span class="text-h5">Rs.</span> {{ stats.todaysEarnings }}</span>
            </div>
          </q-card-section>
        </q-card>
      </div>

      <!-- Pending Payments -->
      <div class="col-12 col-md-3">
        <q-card class="stat-card bg-white shadow-2" style="border-radius: 16px; border: 1px solid #f0f0f0;">
          <q-card-section class="q-pa-lg row items-center no-wrap">
            <q-avatar size="64px" color="red-1" text-color="negative" class="q-mr-md" style="border-radius: 16px;">
              <q-icon name="warning" size="36px"/>
            </q-avatar>
            <div class="column">
              <span class="text-subtitle2 text-grey-8 text-weight-bold q-mb-xs">Pending Dues</span>
              <span class="text-h3 text-weight-bolder text-dark" style="line-height: 1;">{{ stats.pendingDues }}</span>
            </div>
          </q-card-section>
        </q-card>
      </div>
    </div>

    <!-- Main Content Area -->
    <div class="row q-col-gutter-lg">
      
      <!-- Chart Area -->
      <div class="col-12 col-md-8">
        <q-card class="bg-white shadow-2" style="border-radius: 16px; min-height: 400px; border: 1px solid #f0f0f0;">
          <q-card-section class="q-pa-lg">
            <div class="row items-center justify-between q-mb-md">
              <div class="text-h6 text-weight-bolder text-dark">Revenue Overview</div>
              <q-btn outline color="grey-5" text-color="dark" label="This Week" icon-right="expand_more" no-caps dense class="q-px-sm" style="border-radius: 6px;"/>
            </div>
            
            <!-- Placeholder for chart -->
            <div class="flex flex-center bg-grey-1" style="height: 300px; border-radius: 12px; border: 1px dashed #e0e0e0;">
              <div class="column items-center">
                <q-icon name="bar_chart" size="64px" color="grey-4" />
                <div class="text-grey-6 text-weight-medium q-mt-sm">Chart data will appear here</div>
                <div class="text-caption text-grey-5">Waiting for sufficient data</div>
              </div>
            </div>
          </q-card-section>
        </q-card>
      </div>

      <!-- Recent Activities -->
      <div class="col-12 col-md-4">
        <q-card class="bg-white shadow-2" style="border-radius: 16px; min-height: 400px; border: 1px solid #f0f0f0;">
          <q-card-section class="q-pa-lg">
            <div class="row items-center justify-between q-mb-md">
              <div class="text-h6 text-weight-bolder text-dark">Recent Activities</div>
              <q-btn flat color="primary" label="View All" no-caps dense v-if="activities.length > 0"/>
            </div>

            <q-list class="q-mt-sm" v-if="activities.length > 0">
              <q-item v-for="(activity, index) in activities" :key="index" class="q-py-md q-px-none" style="border-bottom: 1px solid #f5f5f5;">
                <q-item-section avatar>
                  <q-avatar size="40px" :color="activity.color + '-1'" :text-color="activity.color" class="text-weight-bold shadow-1">
                    {{ activity.initial }}
                  </q-avatar>
                </q-item-section>
                
                <q-item-section>
                  <q-item-label class="text-weight-bold text-dark" style="font-size: 0.95rem;">
                    {{ activity.title }}
                  </q-item-label>
                  <q-item-label caption class="text-grey-6">
                    {{ activity.description }}
                  </q-item-label>
                </q-item-section>

                <q-item-section side>
                  <q-item-label caption class="text-grey-5">{{ activity.time }}</q-item-label>
                </q-item-section>
              </q-item>
            </q-list>

            <div v-else class="flex flex-center q-pt-xl q-pb-md text-grey-5 column">
              <q-icon name="history" size="48px" class="q-mb-sm opacity-50"/>
              <div>No recent activities</div>
            </div>
          </q-card-section>
        </q-card>
      </div>
      
    </div>
  </q-page>
</template>

<script setup>
import { ref } from 'vue'

const stats = ref({
  totalStudents: 0,
  activeClasses: 0,
  todaysEarnings: '0.00',
  pendingDues: 0
})

const activities = ref([])
</script>

<style scoped>
.stat-card {
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  border-color: #f4f6f8;
}
.stat-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 24px -10px rgba(0, 0, 0, 0.1) !important;
  border-color: #e2e8f0;
}
</style>
