<template>
  <q-page class="bg-primary text-secondary">
    <!-- Navbar -->
    <q-toolbar class="q-pa-sm q-pa-md-md q-px-md-lg text-secondary" style="background: rgba(0, 0, 0, 0.6); backdrop-filter: blur(12px); border-bottom: 1px solid rgba(255,255,255,0.05); position: sticky; top: 0; z-index: 1000;">
      <q-toolbar-title class="text-weight-bold text-h5 row items-center" style="letter-spacing: 2px;">
        <q-icon name="school" class="q-mr-sm" /> Class<span class="text-accent">Master</span>
      </q-toolbar-title>
      <q-space />
      <div class="gt-xs">
        <q-btn flat label="Features" class="q-mr-sm text-weight-medium text-capitalize" />
        <q-btn flat label="How it Works" class="q-mr-sm text-weight-medium text-capitalize" />
        <q-btn flat label="Testimonials" class="q-mr-lg text-weight-medium text-capitalize" />
      </div>
      <div class="gt-xs row items-center">
        <q-btn outline label="Login" color="secondary" class="q-mr-sm q-px-md text-capitalize text-weight-bold" style="border-radius: 8px;" />
        <q-btn unelevated label="Get Started" color="secondary" text-color="primary" class="q-px-md text-capitalize text-weight-bold" style="border-radius: 8px;" />
      </div>
      <q-btn flat dense icon="menu" class="lt-sm" color="secondary" />
    </q-toolbar>

    <!-- Hero Section -->
    <div class="row items-center justify-center q-col-gutter-md q-col-gutter-lg-xl q-px-md q-px-lg-xl q-py-lg q-py-md-xl hero-container">
      <div class="col-12 col-md-5 col-lg-5 text-left q-mb-xl q-mb-md-none">
        <div class="row items-center justify-start q-gutter-sm q-mb-md">
          <q-chip outline color="secondary" text-color="secondary" class="q-px-md shadow-2" style="border-color: rgba(255,255,255,0.2);">
            Version 2.0 Now Available
          </q-chip>
          <q-chip color="primary" text-color="secondary" class="q-px-md shadow-2" style="border: 1px solid rgba(255,255,255,0.2);">
            <q-icon name="code" class="q-mr-sm" size="xs" />
            100% Free & Open Source
          </q-chip>
        </div>
        <div class="text-h2 text-weight-bolder q-mb-md text-white q-mt-sm" style="line-height: 1.1; letter-spacing: -1.5px;">
          Modernize Your <br />
          <span class="text-secondary" style="opacity: 0.8;">Tuition Classes</span>
        </div>
        <p class="text-h6 text-accent q-mb-lg text-weight-regular">
          The completely free and open-source management system designed specifically for tuition centers and independent tutors. Fast, reliable, beautifully minimal, and costs you absolutely nothing.
        </p>
        <div class="row q-gutter-sm q-gutter-md-md justify-start col-12">
          <q-btn size="lg" unelevated label="Start for free" color="secondary" text-color="primary" class="full-width-xs q-px-xl text-weight-bold text-capitalize shadow-10" style="border-radius: 10px;" />
          <q-btn size="lg" outline label="Learn more" color="secondary" class="full-width-xs q-px-xl text-capitalize text-weight-bold" style="border-radius: 10px;" />
        </div>
        
        <div class="q-mt-xl row items-center justify-start pt-sm">
          <q-avatar size="45px" class="q-mr-sm shadow-5" v-for="i in 4" :key="i" style="border: 2px solid rgba(255, 255, 255, 0.2); margin-left: -10px;">
            <img :src="`https://cdn.quasar.dev/img/avatar${i}.jpg`">
          </q-avatar>
          <div class="q-ml-md text-subtitle1 text-accent q-pl-sm" style="margin-left: 0;">
            Trusted by <strong class="text-secondary text-weight-bolder">500+</strong> Tutors
          </div>
        </div>
      </div>
      
      <div class="col-12 col-md-6 col-lg-6 text-center q-mt-lg q-mt-md-none">
        <!-- Dashboard Image -->
        <q-img
          src="~assets/hero_dashboard.png"
          class="rounded-borders glossy shadow-24 hero-img"
        />
      </div>
    </div>

    <!-- Features Section -->
    <div class="q-px-md q-px-lg-xl q-py-xl bg-dark" style="border-top: 1px solid rgba(255,255,255,0.05); border-bottom: 1px solid rgba(255,255,255,0.05);">
      <div class="text-center q-mb-xl q-pb-md">
        <h3 class="text-weight-bold q-mb-sm text-secondary" style="letter-spacing: -1px;">Everything you need</h3>
        <p class="text-accent text-h6 text-weight-regular">Built to give you full control over your operations.</p>
      </div>

      <div class="row q-col-gutter-md q-col-gutter-lg-xl justify-center max-w-1200 q-mx-auto">
        <!-- Feature 1 -->
        <div class="col-12 col-sm-6 col-md-3" v-for="feature in features" :key="feature.title">
          <q-card class="bg-primary text-secondary feature-card q-pa-lg shadow-10" style="border: 1px solid rgba(255,255,255,0.08); border-radius: 20px; height: 100%;">
            <q-card-section class="q-pa-none">
              <q-avatar size="64px" color="dark" text-color="secondary" class="q-mb-lg" style="border: 1px solid rgba(255,255,255,0.1);">
                <q-icon :name="feature.icon" size="32px" />
              </q-avatar>
              <div class="text-h6 text-weight-bold q-mb-sm">{{ feature.title }}</div>
              <div class="text-accent" style="line-height: 1.6;">{{ feature.description }}</div>
            </q-card-section>
          </q-card>
        </div>
      </div>
    </div>

    <!-- Info / Split Section -->
    <div class="row items-center q-px-md q-px-lg-xl q-py-xl max-w-1200 q-mx-auto" style="min-height: 60vh;">
      <div class="col-12 col-md-5 text-center text-md-left">
        <q-img
          src="~assets/education_illustration.png"
          class="rounded-borders"
          style="max-width: 100%; border-radius: 20px;"
        />
      </div>
      <div class="col-12 offset-md-1 col-md-6 q-mt-xl q-mt-md-none">
        <div class="text-h3 text-weight-bold q-mb-lg" style="letter-spacing: -1px;">Simplicity at its core</div>
        <p class="text-h6 text-accent q-mb-xl text-weight-regular" style="line-height: 1.6;">
          Managing students, classes, and payments shouldn't be complicated. We designed an interface that stays out of your way, so you can focus on what matters most — teaching.
        </p>
        <q-list class="text-secondary q-mb-xl">
          <q-item v-for="item in highlights" :key="item" class="q-px-none q-py-sm">
            <q-item-section avatar class="min-w-fit q-pr-md">
              <div class="bg-dark rounded-borders q-pa-sm" style="border: 1px solid rgba(255,255,255,0.1);">
                <q-icon name="check" color="secondary" size="xs" />
              </div>
            </q-item-section>
            <q-item-section class="text-h6 text-weight-regular" style="font-size: 1.1rem;">
              {{ item }}
            </q-item-section>
          </q-item>
        </q-list>
        <q-btn outline label="Explore Dashboard" color="secondary" class="q-px-xl q-py-sm text-capitalize text-weight-bold" style="border-radius: 10px;" />
      </div>
    </div>

    <!-- CTA Section -->
    <div class="q-px-md q-px-lg-xl q-py-xl bg-secondary text-primary text-center">
      <div class="q-py-xl q-px-sm mx-auto" style="max-width: 700px; margin: 0 auto;">
        <h2 class="text-weight-bolder q-mb-lg" style="letter-spacing: -1.5px; line-height: 1.1;">Ready to elevate your classes?</h2>
        <p class="text-h5 text-grey-8 q-mb-xl text-weight-regular">Join hundreds of tutors who are already managing their classes seamlessly with ClassMaster.</p>
        <div class="row justify-center q-gutter-sm q-gutter-md-md">
          <q-btn size="lg" unelevated label="Create your account" color="primary" text-color="secondary" class="full-width-xs q-px-xl text-weight-bold text-capitalize" style="border-radius: 12px; box-shadow: 0 10px 20px rgba(0,0,0,0.2);" />
          <q-btn size="lg" outline label="View on GitHub" icon="code" color="primary" class="full-width-xs q-px-xl text-weight-bold text-capitalize" style="border-radius: 12px; border-width: 2px;" />
        </div>
      </div>
    </div>

    <!-- Footer -->
    <q-toolbar class="q-pa-lg bg-dark text-accent justify-between column sm-row flex-center" style="border-top: 1px solid rgba(255,255,255,0.05); min-height: 80px; gap: 15px;">
      <div class="text-body1 text-center">&copy; {{ new Date().getFullYear() }} NexGen software solutions (Pvt) Ltd. All rights reserved.</div>
      <div class="row q-gutter-md">
        <q-btn flat round icon="facebook" size="sm" class="text-accent hover-text-secondary" />
        <q-btn flat round icon="mail" size="sm" class="text-accent hover-text-secondary" />
        <q-btn flat round icon="share" size="sm" class="text-accent hover-text-secondary" />
      </div>
    </q-toolbar>
  </q-page>
</template>

<script setup>
import { ref } from 'vue'

const features = ref([
  {
    icon: 'group',
    title: 'Student Management',
    description: 'Keep track of student details, performance, and parental contacts easily in one central hub.'
  },
  {
    icon: 'payments',
    title: 'Payment Tracking',
    description: 'Automate fee collections, track pending payments, and generate digital receipts instantly.'
  },
  {
    icon: 'event',
    title: 'Class Scheduling',
    description: 'Organize timetables effortlessly. Avoid clashes and notify students of changes in real-time.'
  },
  {
    icon: 'fact_check',
    title: 'Attendance App',
    description: 'Mark attendance with a few clicks or use QR code scanning for seamless, faster processing.'
  }
])

const highlights = ref([
  '100% Free and Open Source forever',
  'Minimalist, distraction-free interface',
  'Real-time data synchronization across devices',
  'Automated SMS & Email notifications for parents',
  'Comprehensive analytical reports and insights'
])
</script>

<style scoped>
.hero-container {
  min-height: 85vh;
  position: relative;
  overflow: hidden;
}

.hero-img {
  max-width: 100%; 
  border-radius: 24px; 
  border: 1px solid rgba(255,255,255,0.1);
  transform: scale(1);
  transition: transform 0.5s ease;
}

.hero-img:hover {
  transform: scale(1.02);
}

.max-w-1200 {
  max-width: 1200px;
}

.feature-card {
  transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

.feature-card:hover {
  transform: translateY(-10px);
  border-color: rgba(255, 255, 255, 0.3) !important;
  box-shadow: 0 15px 30px rgba(0,0,0,0.6) !important;
  background-color: #0a0a0a !important;
}

.min-w-fit {
  min-width: fit-content;
}

.hover-text-secondary:hover {
  color: #fff !important;
}

@media (max-width: 599px) {
  .text-h2 {
    font-size: 2.2rem !important;
    letter-spacing: -1px !important;
  }
  .text-h3 {
    font-size: 1.8rem !important;
  }
  .hero-container {
    min-height: auto;
    padding-top: 2rem;
  }
  .full-width-xs {
    width: 100%;
    margin-left: 0 !important;
    margin-right: 0 !important;
    margin-top: 8px !important;
  }
}
</style>
