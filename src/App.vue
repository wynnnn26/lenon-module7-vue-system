<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

import AppHeader from './components/AppHeader.vue'
import AttendanceForm from './components/AttendanceForm.vue'
import AttendanceList from './components/AttendanceList.vue'
import AppFooter from './components/AppFooter.vue'

// ========================================
// APPLICATION STATE
// ========================================

const darkMode = ref(true)
const records = ref([])
const editingRecord = ref(null)

// ========================================
// DELETE CONFIRMATION
// ========================================

const showDeleteModal = ref(false)
const recordToDelete = ref(null)

// ========================================
// NOTIFICATION
// ========================================

const notification = ref({
  show: false,
  message: '',
  type: 'success'
})

let notificationTimer = null

function showNotification(message, type = 'success') {
  notification.value = {
    show: true,
    message,
    type
  }

  clearTimeout(notificationTimer)

  notificationTimer = setTimeout(() => {
    notification.value.show = false
  }, 3000)
}

// ========================================
// LOAD SAVED DATA
// ========================================

onMounted(() => {
  const savedTheme = localStorage.getItem('attendance-theme')
  if (savedTheme) {
    darkMode.value = savedTheme === 'dark'
  }

  const savedRecords = localStorage.getItem('attendance-records')
  if (savedRecords) {
    try {
      records.value = JSON.parse(savedRecords)
    } catch (error) {
      console.error('Unable to load attendance records:', error)
      records.value = []
    }
  }
})

// ========================================
// DARK / LIGHT MODE
// ========================================

function toggleTheme() {
  darkMode.value = !darkMode.value
  localStorage.setItem('attendance-theme', darkMode.value ? 'dark' : 'light')
}

// ========================================
// ADD RECORD
// ========================================

function addRecord(record) {
  const newRecord = {
    id: Date.now(),
    ...record
  }

  records.value.push(newRecord)
  saveRecords()
  showNotification('Attendance record added successfully!')
}

// ========================================
// EDIT RECORD
// ========================================

function editRecord(record) {
  editingRecord.value = { ...record }
  window.scrollTo({
    top: 0,
    behavior: 'smooth'
  })
}

// ========================================
// UPDATE RECORD
// ========================================

function updateRecord(updatedRecord) {
  const index = records.value.findIndex(record => record.id === updatedRecord.id)

  if (index !== -1) {
    records.value[index] = { ...updatedRecord }
  }

  editingRecord.value = null
  saveRecords()
  showNotification('Attendance record updated successfully!')
}

// ========================================
// CANCEL EDIT
// ========================================

function cancelEdit() {
  editingRecord.value = null
}

// ========================================
// DELETE RECORD
// ========================================

function deleteRecord(id) {
  const record = records.value.find(record => record.id === id)
  if (!record) return

  recordToDelete.value = record
  showDeleteModal.value = true
}

// ========================================
// CONFIRM DELETE
// ========================================

function confirmDelete() {
  if (!recordToDelete.value) return

  const id = recordToDelete.value.id
  records.value = records.value.filter(record => record.id !== id)

  if (editingRecord.value && editingRecord.value.id === id) {
    editingRecord.value = null
  }

  saveRecords()
  showDeleteModal.value = false
  recordToDelete.value = null

  showNotification('Attendance record deleted successfully!')
}

// ========================================
// CANCEL DELETE
// ========================================

function cancelDelete() {
  showDeleteModal.value = false
  recordToDelete.value = null
}

// ========================================
// SAVE RECORDS
// ========================================

function saveRecords() {
  localStorage.setItem('attendance-records', JSON.stringify(records.value))
}

// ========================================
// EXPORT CSV
// ========================================

function exportRecords() {
  if (records.value.length === 0) {
    showNotification('There are no attendance records to export.', 'error')
    return
  }

  const headers = ['Student Number', 'Student Name', 'Section', 'Date', 'Status']
  const rows = records.value.map(record => [
    record.studentNumber,
    record.studentName,
    record.section,
    record.date,
    record.status
  ])

  const csvContent = [headers, ...rows]
    .map(row => row.map(value => `"${String(value).replace(/"/g, '""')}"`).join(','))
    .join('\n')

  const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')

  link.href = url
  link.download = `attendance-report-${new Date().toISOString().split('T')[0]}.csv`
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
  URL.revokeObjectURL(url)

  showNotification('Attendance report exported successfully!')
}

// ========================================
// STATISTICS
// ========================================

const totalRecords = computed(() => records.value.length)
const presentRecords = computed(() => records.value.filter(r => r.status === 'Present').length)
const lateRecords = computed(() => records.value.filter(r => r.status === 'Late').length)
const absentRecords = computed(() => records.value.filter(r => r.status === 'Absent').length)
const excusedRecords = computed(() => records.value.filter(r => r.status === 'Excused').length)

// Rate calculations for enterprise metrics
const presentRate = computed(() => {
  if (totalRecords.value === 0) return '0%'
  return Math.round((presentRecords.value / totalRecords.value) * 100) + '%'
})

// ========================================
// MOUSE GLOW EFFECT
// ========================================

const cursorX = ref(0)
const cursorY = ref(0)
const cursorVisible = ref(false)

function updateCursorPosition(event) {
  cursorX.value = event.clientX
  cursorY.value = event.clientY
  cursorVisible.value = true
}

function hideCursor() {
  cursorVisible.value = false
}

onMounted(() => {
  window.addEventListener('mousemove', updateCursorPosition)
  window.addEventListener('mouseout', hideCursor)
})

onBeforeUnmount(() => {
  window.removeEventListener('mousemove', updateCursorPosition)
  window.removeEventListener('mouseout', hideCursor)
})
</script>

<template>
  <!-- Custom Neon Cursor Glow -->
  <div
    v-if="cursorVisible"
    class="pointer-events-none fixed z-[9999] hidden h-64 w-64 -translate-x-1/2 -translate-y-1/2 rounded-full bg-cyan-500/10 blur-3xl transition-opacity duration-300 dark:bg-fuchsia-500/20 md:block"
    :style="{
      left: `${cursorX}px`,
      top: `${cursorY}px`
    }"
  ></div>

  <!-- Application Container -->
  <div :class="{ dark: darkMode }" class="min-h-screen transition-colors duration-300 ease-in-out">
    <div
      class="min-h-screen bg-slate-100 text-slate-900 transition-colors duration-300 dark:bg-[#09090b] dark:text-zinc-100"
    >
      <!-- App Header -->
      <AppHeader :dark-mode="darkMode" @toggle-theme="toggleTheme" />

      <!-- Main Section -->
      <main class="mx-auto w-full max-w-[1280px] px-4 py-6 sm:px-6 sm:py-8 lg:px-8">
        <!-- Page Title & Export Action -->
        <div
          class="mb-6 flex flex-col gap-4 border-b border-slate-200 pb-6 dark:border-zinc-800/80 sm:flex-row sm:items-center sm:justify-between"
        >
          <div>
            <div class="flex items-center gap-2">
              <p class="text-xs font-bold uppercase tracking-wider text-cyan-600 dark:text-cyan-400 dark:drop-shadow-[0_0_8px_rgba(34,211,238,0.8)]">
                Management Dashboard
              </p>
            </div>
            <h2 class="mt-1 text-2xl font-extrabold tracking-tight text-slate-900 dark:text-white sm:text-3xl">
              Student Attendance
            </h2>
            <p class="mt-1 text-sm text-slate-500 dark:text-zinc-400">
              Track, record, and generate logs for ongoing classes and academic sessions.
            </p>
          </div>

          <!-- Export CSV Button -->
          <button
            type="button"
            @click="exportRecords"
            class="group inline-flex items-center justify-center gap-2 rounded-xl bg-slate-900 px-5 py-2.5 text-sm font-semibold text-white shadow-md transition-all duration-200 ease-out hover:-translate-y-0.5 hover:bg-slate-800 active:translate-y-0 dark:border dark:border-cyan-500/50 dark:bg-black dark:text-cyan-300 dark:shadow-[0_0_15px_rgba(6,182,212,0.4)] dark:hover:bg-cyan-950/40 dark:hover:shadow-[0_0_25px_rgba(6,182,212,0.7)] sm:w-auto"
          >
            <svg
              class="h-4 w-4 transition-transform duration-200 group-hover:translate-y-0.5"
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
              stroke-width="2.5"
            >
              <path stroke-linecap="round" stroke-linejoin="round" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4" />
            </svg>
            <span>Export CSV Report</span>
          </button>
        </div>

        <!-- Statistics Metric Grid (Neon Colors in Dark Mode) -->
        <div class="mb-8 grid grid-cols-2 gap-4 sm:grid-cols-3 lg:grid-cols-5">
          
          <!-- Total Records Card (Cyan/Neon Blue) -->
          <div
            class="group relative overflow-hidden rounded-2xl border border-cyan-200 bg-cyan-50/50 p-5 shadow-sm transition-all duration-300 hover:-translate-y-1 hover:shadow-md dark:border-cyan-500/50 dark:bg-black/80 dark:shadow-[0_0_20px_rgba(6,182,212,0.25)] dark:hover:border-cyan-400 dark:hover:shadow-[0_0_30px_rgba(6,182,212,0.5)]"
          >
            <div class="flex items-center justify-between">
              <p class="text-xs font-semibold uppercase tracking-wider text-cyan-800 dark:text-cyan-300">Total Logged</p>
              <span class="rounded-lg bg-cyan-100 p-2 text-cyan-700 dark:bg-cyan-950/80 dark:text-cyan-400 dark:shadow-[0_0_10px_rgba(34,211,238,0.5)]">
                <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 20h5v-2a3 3 0 00-5.356-1.857M17 20H7m10 0v-2c0-.656-.126-1.283-.356-1.857M7 20H2v-2a3 3 0 015.356-1.857M7 20v-2c0-.656.126-1.283.356-1.857m0 0a5.002 5.002 0 019.288 0M15 7a3 3 0 11-6 0 3 3 0 016 0z" />
                </svg>
              </span>
            </div>
            <p class="mt-3 text-3xl font-black tracking-tight text-cyan-950 dark:text-cyan-300 dark:drop-shadow-[0_0_10px_rgba(34,211,238,0.8)]">
              {{ totalRecords }}
            </p>
            <p class="mt-1 text-xs font-medium text-cyan-700 dark:text-cyan-400/80">Recorded students</p>
          </div>

          <!-- Present Card (Emerald/Neon Green) -->
          <div
            class="group relative overflow-hidden rounded-2xl border border-emerald-200 bg-emerald-50/50 p-5 shadow-sm transition-all duration-300 hover:-translate-y-1 hover:shadow-md dark:border-emerald-500/50 dark:bg-black/80 dark:shadow-[0_0_20px_rgba(16,185,129,0.25)] dark:hover:border-emerald-400 dark:hover:shadow-[0_0_30px_rgba(16,185,129,0.5)]"
          >
            <div class="flex items-center justify-between">
              <p class="text-xs font-semibold uppercase tracking-wider text-emerald-800 dark:text-emerald-300">Present</p>
              <span class="rounded-lg bg-emerald-100 p-2 text-emerald-700 dark:bg-emerald-950/80 dark:text-emerald-400 dark:shadow-[0_0_10px_rgba(52,211,153,0.5)]">
                <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M5 13l4 4L19 7" />
                </svg>
              </span>
            </div>
            <p class="mt-3 text-3xl font-black tracking-tight text-emerald-950 dark:text-emerald-300 dark:drop-shadow-[0_0_10px_rgba(52,211,153,0.8)]">
              {{ presentRecords }}
            </p>
            <p class="mt-1 text-xs font-medium text-emerald-700 dark:text-emerald-400/80">{{ presentRate }} turnout rate</p>
          </div>

          <!-- Late Card (Amber/Neon Yellow) -->
          <div
            class="group relative overflow-hidden rounded-2xl border border-amber-200 bg-amber-50/50 p-5 shadow-sm transition-all duration-300 hover:-translate-y-1 hover:shadow-md dark:border-amber-500/50 dark:bg-black/80 dark:shadow-[0_0_20px_rgba(245,158,11,0.25)] dark:hover:border-amber-400 dark:hover:shadow-[0_0_30px_rgba(245,158,11,0.5)]"
          >
            <div class="flex items-center justify-between">
              <p class="text-xs font-semibold uppercase tracking-wider text-amber-800 dark:text-amber-300">Late</p>
              <span class="rounded-lg bg-amber-100 p-2 text-amber-700 dark:bg-amber-950/80 dark:text-amber-400 dark:shadow-[0_0_10px_rgba(251,191,36,0.5)]">
                <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
                </svg>
              </span>
            </div>
            <p class="mt-3 text-3xl font-black tracking-tight text-amber-950 dark:text-amber-300 dark:drop-shadow-[0_0_10px_rgba(251,191,36,0.8)]">
              {{ lateRecords }}
            </p>
            <p class="mt-1 text-xs font-medium text-amber-700 dark:text-amber-400/80">Requires review</p>
          </div>

          <!-- Absent Card (Rose/Neon Pink) -->
          <div
            class="group relative overflow-hidden rounded-2xl border border-rose-200 bg-rose-50/50 p-5 shadow-sm transition-all duration-300 hover:-translate-y-1 hover:shadow-md dark:border-pink-500/50 dark:bg-black/80 dark:shadow-[0_0_20px_rgba(236,72,153,0.25)] dark:hover:border-pink-400 dark:hover:shadow-[0_0_30px_rgba(236,72,153,0.5)]"
          >
            <div class="flex items-center justify-between">
              <p class="text-xs font-semibold uppercase tracking-wider text-rose-800 dark:text-pink-300">Absent</p>
              <span class="rounded-lg bg-rose-100 p-2 text-rose-700 dark:bg-pink-950/80 dark:text-pink-400 dark:shadow-[0_0_10px_rgba(244,114,182,0.5)]">
                <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M6 18L18 6M6 6l12 12" />
                </svg>
              </span>
            </div>
            <p class="mt-3 text-3xl font-black tracking-tight text-rose-950 dark:text-pink-300 dark:drop-shadow-[0_0_10px_rgba(244,114,182,0.8)]">
              {{ absentRecords }}
            </p>
            <p class="mt-1 text-xs font-medium text-rose-700 dark:text-pink-400/80">Unexcused missing</p>
          </div>

          <!-- Excused Card (Fuchsia/Neon Purple) -->
          <div
            class="col-span-2 group relative overflow-hidden rounded-2xl border border-fuchsia-200 bg-fuchsia-50/50 p-5 shadow-sm transition-all duration-300 hover:-translate-y-1 hover:shadow-md dark:border-fuchsia-500/50 dark:bg-black/80 dark:shadow-[0_0_20px_rgba(217,70,239,0.25)] dark:hover:border-fuchsia-400 dark:hover:shadow-[0_0_30px_rgba(217,70,239,0.5)] sm:col-span-1"
          >
            <div class="flex items-center justify-between">
              <p class="text-xs font-semibold uppercase tracking-wider text-fuchsia-800 dark:text-fuchsia-300">Excused</p>
              <span class="rounded-lg bg-fuchsia-100 p-2 text-fuchsia-700 dark:bg-fuchsia-950/80 dark:text-fuchsia-400 dark:shadow-[0_0_10px_rgba(232,121,249,0.5)]">
                <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z" />
                </svg>
              </span>
            </div>
            <p class="mt-3 text-3xl font-black tracking-tight text-fuchsia-950 dark:text-fuchsia-300 dark:drop-shadow-[0_0_10px_rgba(232,121,249,0.8)]">
              {{ excusedRecords }}
            </p>
            <p class="mt-1 text-xs font-medium text-fuchsia-700 dark:text-fuchsia-400/80">Permitted absences</p>
          </div>
        </div>

        <!-- Attendance Panel Container -->
        <div
          class="overflow-hidden rounded-2xl border border-slate-200 bg-white shadow-sm dark:border-zinc-800/80 dark:bg-zinc-900/90 dark:shadow-[0_0_30px_rgba(0,0,0,0.8)]"
        >
          <AttendanceForm
            :editing-record="editingRecord"
            @add-record="addRecord"
            @update-record="updateRecord"
            @cancel-edit="cancelEdit"
            @validation-error="msg => showNotification(msg, 'error')"
          />

          <AttendanceList
            :records="records"
            @delete-record="deleteRecord"
            @edit-record="editRecord"
          />
        </div>
      </main>

      <!-- App Footer -->
      <AppFooter />

      <!-- Delete Confirmation Modal -->
      <Transition
        enter-active-class="transition-all duration-200 ease-out"
        enter-from-class="opacity-0 scale-95"
        enter-to-class="opacity-100 scale-100"
        leave-active-class="transition-all duration-150 ease-in"
        leave-from-class="opacity-100 scale-100"
        leave-to-class="opacity-0 scale-95"
      >
        <div
          v-if="showDeleteModal"
          class="fixed inset-0 z-50 flex items-center justify-center bg-black/80 p-4 backdrop-blur-md"
          @click.self="cancelDelete"
        >
          <div
            class="w-full max-w-md rounded-2xl border border-slate-200 bg-white p-6 shadow-2xl dark:border-pink-500/40 dark:bg-zinc-950 dark:shadow-[0_0_30px_rgba(236,72,153,0.3)]"
          >
            <!-- Warning Icon Header -->
            <div class="mx-auto flex h-12 w-12 items-center justify-center rounded-full bg-rose-100 text-rose-600 dark:bg-pink-950/80 dark:text-pink-400 dark:shadow-[0_0_15px_rgba(236,72,153,0.6)]">
              <svg class="h-6 w-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
              </svg>
            </div>

            <!-- Modal Content -->
            <div class="mt-4 text-center">
              <h3 class="text-lg font-bold text-slate-900 dark:text-white">
                Delete Attendance Record
              </h3>
              <p class="mt-1.5 text-sm text-slate-500 dark:text-zinc-400">
                Are you sure you want to remove this record? This action is permanent and cannot be undone.
              </p>
            </div>

            <!-- Record Details -->
            <div
              v-if="recordToDelete"
              class="mt-4 rounded-xl border border-slate-200 bg-slate-50 p-3.5 dark:border-zinc-800 dark:bg-black/60"
            >
              <p class="text-sm font-bold text-slate-900 dark:text-white">
                {{ recordToDelete.studentName }}
              </p>
              <p class="mt-0.5 text-xs text-slate-500 dark:text-zinc-400">
                Student ID: {{ recordToDelete.studentNumber }}
              </p>

              <div class="mt-2 flex flex-wrap items-center gap-2 text-xs font-medium text-slate-600 dark:text-zinc-300">
                <span class="rounded bg-slate-200 px-2 py-0.5 dark:bg-zinc-800">
                  {{ recordToDelete.section }}
                </span>
                <span>•</span>
                <span>{{ recordToDelete.date }}</span>
                <span>•</span>
                <span class="font-bold text-slate-900 dark:text-cyan-400">
                  {{ recordToDelete.status }}
                </span>
              </div>
            </div>

            <!-- Action Buttons -->
            <div class="mt-6 flex flex-col-reverse gap-2 sm:flex-row sm:justify-end">
              <button
                type="button"
                @click="cancelDelete"
                class="w-full rounded-xl border border-slate-300 px-4 py-2.5 text-sm font-semibold text-slate-700 transition-all duration-150 hover:bg-slate-100 dark:border-zinc-700 dark:text-zinc-300 dark:hover:bg-zinc-800 sm:w-auto"
              >
                Cancel
              </button>
              <button
                type="button"
                @click="confirmDelete"
                class="w-full rounded-xl bg-rose-600 px-4 py-2.5 text-sm font-semibold text-white shadow-sm transition-all duration-150 hover:bg-rose-500 active:scale-[0.98] dark:border dark:border-pink-500 dark:bg-pink-600 dark:shadow-[0_0_15px_rgba(236,72,153,0.5)] dark:hover:bg-pink-500 sm:w-auto"
              >
                Delete Record
              </button>
            </div>
          </div>
        </div>
      </Transition>

      <!-- Toast Notification -->
      <Transition
        enter-active-class="transition-all duration-300 ease-out"
        enter-from-class="translate-y-4 opacity-0 scale-95"
        enter-to-class="translate-y-0 opacity-100 scale-100"
        leave-active-class="transition-all duration-200 ease-in"
        leave-from-class="translate-y-0 opacity-100 scale-100"
        leave-to-class="translate-y-4 opacity-0 scale-95"
      >
        <div
          v-if="notification.show"
          class="fixed bottom-5 right-5 z-[60] w-[calc(100%-2.5rem)] max-w-sm rounded-2xl border border-slate-200 bg-white/95 p-4 text-slate-900 shadow-xl backdrop-blur-md transition-all duration-300 dark:border-cyan-500/50 dark:bg-black/90 dark:text-white dark:shadow-[0_0_20px_rgba(6,182,212,0.4)] sm:bottom-6 sm:right-6"
        >
          <div class="flex items-start gap-3">
            <!-- Icon -->
            <div
              class="flex h-8 w-8 shrink-0 items-center justify-center rounded-lg bg-slate-100 text-slate-900 dark:bg-cyan-950 dark:text-cyan-300 dark:shadow-[0_0_10px_rgba(34,211,238,0.5)]"
            >
              <svg v-if="notification.type === 'error'" class="h-5 w-5 text-rose-500 dark:text-pink-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
              </svg>
              <svg v-else class="h-5 w-5 text-emerald-600 dark:text-emerald-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
              </svg>
            </div>

            <!-- Text Content -->
            <div class="min-w-0 flex-1">
              <p class="text-xs font-bold uppercase tracking-wider text-slate-500 dark:text-cyan-400">
                {{ notification.type === 'error' ? 'Error' : 'Success' }}
              </p>
              <p class="mt-0.5 text-sm font-medium leading-snug">
                {{ notification.message }}
              </p>
            </div>

            <!-- Close Action -->
            <button
              type="button"
              @click="notification.show = false"
              class="text-slate-400 transition-colors hover:text-slate-600 dark:hover:text-zinc-200"
              aria-label="Close notification"
            >
              <svg class="h-5 w-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
              </svg>
            </button>
          </div>
        </div>
      </Transition>
    </div>
  </div>
</template>