<script setup>
import { ref, watch } from 'vue'

// ========================================
// PROPS & EMITS
// ========================================

const props = defineProps({
  editingRecord: {
    type: Object,
    default: null
  }
})

const emit = defineEmits([
  'add-record',
  'update-record',
  'cancel-edit',
  'validation-error'
])

// ========================================
// FORM DATA
// ========================================

const studentNumber = ref('')
const studentName = ref('')
const section = ref('')
const date = ref('')
const status = ref('Present')

// ========================================
// RESET FORM
// ========================================

function resetForm() {
  studentNumber.value = ''
  studentName.value = ''
  section.value = ''
  date.value = ''
  status.value = 'Present'
}

// ========================================
// LOAD RECORD INTO FORM
// ========================================

function loadRecord(record) {
  if (!record) {
    resetForm()
    return
  }

  studentNumber.value = record.studentNumber || ''
  studentName.value = record.studentName || ''
  section.value = record.section || ''
  date.value = record.date || ''
  status.value = record.status || 'Present'
}

// ========================================
// WATCH EDITING RECORD
// ========================================

watch(
  () => props.editingRecord,
  (record) => {
    loadRecord(record)
  },
  { immediate: true }
)

// ========================================
// SUBMIT FORM
// ========================================

function submitForm() {
  // Validate all fields
  if (
    !studentNumber.value.trim() ||
    !studentName.value.trim() ||
    !section.value.trim() ||
    !date.value ||
    !status.value
  ) {
    emit('validation-error', 'Please complete all required fields.')
    return
  }

  const record = {
    studentNumber: studentNumber.value.trim(),
    studentName: studentName.value.trim(),
    section: section.value.trim(),
    date: date.value,
    status: status.value
  }

  // UPDATE
  if (props.editingRecord) {
    emit('update-record', {
      id: props.editingRecord.id,
      ...record
    })
    return
  }

  // ADD
  emit('add-record', record)
  resetForm()
}

// ========================================
// CANCEL EDIT
// ========================================

function cancelEdit() {
  resetForm()
  emit('cancel-edit')
}
</script>

<template>
  <!-- =========================================================================
       1. EDIT MODAL WINDOW (Directly rendered to inherit dark mode classes)
  ========================================================================= -->
  <div
    v-if="editingRecord"
    class="fixed inset-0 z-50 flex items-center justify-center bg-black/70 p-4 backdrop-blur-sm transition-all duration-300 dark:bg-black/85"
    @click.self="cancelEdit"
  >
    <div
      class="w-full max-w-2xl rounded-2xl border border-black/10 bg-white p-6 shadow-2xl transition-all duration-300 dark:border-white/10 dark:bg-black dark:shadow-neutral-900/50"
    >
      <!-- Modal Header -->
      <div class="mb-5 flex items-center justify-between border-b border-black/10 pb-3 transition-colors duration-300 dark:border-white/10">
        <div>
          <h3 class="text-lg font-bold text-black transition-colors duration-300 dark:text-white">
            Edit Student Record
          </h3>
          <p class="text-xs text-neutral-600 transition-colors duration-300 dark:text-neutral-400">
            Update the student's attendance details below.
          </p>
        </div>

        <!-- Close (X) Button -->
        <button
          type="button"
          @click="cancelEdit"
          class="rounded-lg p-1.5 text-neutral-500 transition-colors duration-300 hover:bg-black/5 hover:text-black dark:text-neutral-400 dark:hover:bg-white/10 dark:hover:text-white"
        >
          <svg class="h-5 w-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>
      </div>

      <!-- Modal Form Content -->
      <form @submit.prevent="submitForm" class="grid grid-cols-1 gap-4 sm:grid-cols-2">
        <!-- Student Number -->
        <div>
          <label class="mb-1 block text-xs font-semibold text-neutral-800 transition-colors duration-300 dark:text-neutral-200">
            Student Number
          </label>
          <input
            v-model="studentNumber"
            type="text"
            class="w-full rounded-xl border border-neutral-300 bg-neutral-50 px-3 py-2.5 text-sm text-black outline-none transition-all duration-300 placeholder:text-neutral-400 hover:border-black/40 focus:border-black focus:bg-white focus:ring-2 focus:ring-black/10 dark:border-neutral-800 dark:bg-neutral-900 dark:text-white dark:placeholder:text-neutral-500 dark:hover:border-white/40 dark:focus:border-white dark:focus:bg-black dark:focus:ring-2 dark:focus:ring-white/10"
          />
        </div>

        <!-- Student Name -->
        <div>
          <label class="mb-1 block text-xs font-semibold text-neutral-800 transition-colors duration-300 dark:text-neutral-200">
            Student Name
          </label>
          <input
            v-model="studentName"
            type="text"
            class="w-full rounded-xl border border-neutral-300 bg-neutral-50 px-3 py-2.5 text-sm text-black outline-none transition-all duration-300 placeholder:text-neutral-400 hover:border-black/40 focus:border-black focus:bg-white focus:ring-2 focus:ring-black/10 dark:border-neutral-800 dark:bg-neutral-900 dark:text-white dark:placeholder:text-neutral-500 dark:hover:border-white/40 dark:focus:border-white dark:focus:bg-black dark:focus:ring-2 dark:focus:ring-white/10"
          />
        </div>

        <!-- Section -->
        <div>
          <label class="mb-1 block text-xs font-semibold text-neutral-800 transition-colors duration-300 dark:text-neutral-200">
            Section
          </label>
          <input
            v-model="section"
            type="text"
            class="w-full rounded-xl border border-neutral-300 bg-neutral-50 px-3 py-2.5 text-sm text-black outline-none transition-all duration-300 placeholder:text-neutral-400 hover:border-black/40 focus:border-black focus:bg-white focus:ring-2 focus:ring-black/10 dark:border-neutral-800 dark:bg-neutral-900 dark:text-white dark:placeholder:text-neutral-500 dark:hover:border-white/40 dark:focus:border-white dark:focus:bg-black dark:focus:ring-2 dark:focus:ring-white/10"
          />
        </div>

        <!-- Date -->
        <div>
          <label class="mb-1 block text-xs font-semibold text-neutral-800 transition-colors duration-300 dark:text-neutral-200">
            Date
          </label>
          <input
            v-model="date"
            type="date"
            class="w-full rounded-xl border border-neutral-300 bg-neutral-50 px-3 py-2.5 text-sm text-black outline-none transition-all duration-300 hover:border-black/40 focus:border-black focus:bg-white focus:ring-2 focus:ring-black/10 dark:border-neutral-800 dark:bg-neutral-900 dark:text-white dark:hover:border-white/40 dark:focus:border-white dark:focus:bg-black dark:focus:ring-2 dark:focus:ring-white/10"
          />
        </div>

        <!-- Status -->
        <div class="sm:col-span-2">
          <label class="mb-1 block text-xs font-semibold text-neutral-800 transition-colors duration-300 dark:text-neutral-200">
            Status
          </label>
          <select
            v-model="status"
            class="w-full rounded-xl border border-neutral-300 bg-neutral-50 px-3 py-2.5 text-sm text-black outline-none transition-all duration-300 hover:border-black/40 focus:border-black focus:bg-white focus:ring-2 focus:ring-black/10 dark:border-neutral-800 dark:bg-neutral-900 dark:text-white dark:hover:border-white/40 dark:focus:border-white dark:focus:bg-black dark:focus:ring-2 dark:focus:ring-white/10"
          >
            <option value="Present">Present</option>
            <option value="Late">Late</option>
            <option value="Absent">Absent</option>
            <option value="Excused">Excused</option>
          </select>
        </div>

        <!-- Modal Action Buttons -->
        <div class="mt-3 flex justify-end gap-2 sm:col-span-2">
          <button
            type="button"
            @click="cancelEdit"
            class="rounded-xl border border-neutral-300 px-4 py-2.5 text-sm font-semibold text-black transition-all duration-300 hover:bg-neutral-100 dark:border-neutral-700 dark:text-white dark:hover:bg-neutral-800"
          >
            Cancel
          </button>
          <button
            type="submit"
            class="rounded-xl bg-black px-5 py-2.5 text-sm font-semibold text-white shadow-md transition-all duration-300 hover:-translate-y-0.5 hover:bg-neutral-800 dark:bg-white dark:text-black dark:hover:bg-neutral-200"
          >
            Save Changes
          </button>
        </div>
      </form>
    </div>
  </div>

  <!-- =========================================================================
       2. INLINE ADD FORM
  ========================================================================= -->
  <div
    v-if="!editingRecord"
    class="border-b border-neutral-200 bg-white p-4 transition-all duration-300 ease-out dark:border-neutral-800 dark:bg-black sm:p-5"
  >
    <div class="mb-4">
      <h3 class="text-sm font-bold text-black transition-colors duration-300 dark:text-white sm:text-base">
        Add Attendance Record
      </h3>
      <p class="mt-0.5 text-xs text-neutral-600 transition-colors duration-300 dark:text-neutral-400">
        Enter the student attendance information below.
      </p>
    </div>

    <form @submit.prevent="submitForm" class="grid grid-cols-1 gap-3 sm:grid-cols-2 lg:grid-cols-6">
      <div>
        <label class="mb-1.5 block text-xs font-semibold text-neutral-800 dark:text-neutral-200">Student Number</label>
        <input
          v-model="studentNumber"
          type="text"
          placeholder="E.g. 12342026"
          class="w-full rounded-xl border border-neutral-300 bg-neutral-50 px-3 py-2.5 text-sm text-black outline-none transition-all duration-300 ease-out placeholder:text-neutral-400 hover:border-black/40 focus:border-black focus:bg-white focus:ring-2 focus:ring-black/10 dark:border-neutral-800 dark:bg-neutral-900 dark:text-white dark:placeholder:text-neutral-500 dark:hover:border-white/40 dark:focus:border-white dark:focus:bg-black dark:focus:ring-2 dark:focus:ring-white/10"
        />
      </div>

      <div class="sm:col-span-1 lg:col-span-2">
        <label class="mb-1.5 block text-xs font-semibold text-neutral-800 dark:text-neutral-200">Student Name</label>
        <input
          v-model="studentName"
          type="text"
          placeholder="E.g. Juan Dela Cruz"
          class="w-full rounded-xl border border-neutral-300 bg-neutral-50 px-3 py-2.5 text-sm text-black outline-none transition-all duration-300 ease-out placeholder:text-neutral-400 hover:border-black/40 focus:border-black focus:bg-white focus:ring-2 focus:ring-black/10 dark:border-neutral-800 dark:bg-neutral-900 dark:text-white dark:placeholder:text-neutral-500 dark:hover:border-white/40 dark:focus:border-white dark:focus:bg-black dark:focus:ring-2 dark:focus:ring-white/10"
        />
      </div>

      <div>
        <label class="mb-1.5 block text-xs font-semibold text-neutral-800 dark:text-neutral-200">Section</label>
        <input
          v-model="section"
          type="text"
          placeholder="E.g. BSCS 1Z"
          class="w-full rounded-xl border border-neutral-300 bg-neutral-50 px-3 py-2.5 text-sm text-black outline-none transition-all duration-300 ease-out placeholder:text-neutral-400 hover:border-black/40 focus:border-black focus:bg-white focus:ring-2 focus:ring-black/10 dark:border-neutral-800 dark:bg-neutral-900 dark:text-white dark:placeholder:text-neutral-500 dark:hover:border-white/40 dark:focus:border-white dark:focus:bg-black dark:focus:ring-2 dark:focus:ring-white/10"
        />
      </div>

      <div>
        <label class="mb-1.5 block text-xs font-semibold text-neutral-800 dark:text-neutral-200">Date</label>
        <input
          v-model="date"
          type="date"
          class="w-full rounded-xl border border-neutral-300 bg-neutral-50 px-3 py-2.5 text-sm text-black outline-none transition-all duration-300 ease-out hover:border-black/40 focus:border-black focus:bg-white focus:ring-2 focus:ring-black/10 dark:border-neutral-800 dark:bg-neutral-900 dark:text-white dark:hover:border-white/40 dark:focus:border-white dark:focus:bg-black dark:focus:ring-2 dark:focus:ring-white/10"
        />
      </div>

      <div>
        <label class="mb-1.5 block text-xs font-semibold text-neutral-800 dark:text-neutral-200">Status</label>
        <select
          v-model="status"
          class="w-full rounded-xl border border-neutral-300 bg-neutral-50 px-3 py-2.5 text-sm text-black outline-none transition-all duration-300 ease-out hover:border-black/40 focus:border-black focus:bg-white focus:ring-2 focus:ring-black/10 dark:border-neutral-800 dark:bg-neutral-900 dark:text-white dark:hover:border-white/40 dark:focus:border-white dark:focus:bg-black dark:focus:ring-2 dark:focus:ring-white/10"
        >
          <option value="Present">Present</option>
          <option value="Late">Late</option>
          <option value="Absent">Absent</option>
          <option value="Excused">Excused</option>
        </select>
      </div>

      <div class="flex items-end gap-2 sm:col-span-2 lg:col-span-1">
        <button
          type="submit"
          class="flex-1 rounded-xl bg-black px-4 py-2.5 text-sm font-semibold text-white shadow-md transition-all duration-300 ease-out hover:-translate-y-0.5 hover:bg-neutral-800 active:translate-y-0 active:scale-[0.97] dark:bg-white dark:text-black dark:hover:bg-neutral-200"
        >
          Add Record
        </button>
      </div>
    </form>
  </div>
</template>