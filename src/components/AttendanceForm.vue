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
    class="fixed inset-0 z-50 flex items-center justify-center bg-black/60 p-4 backdrop-blur-sm transition-all duration-300 dark:bg-black/80"
    @click.self="cancelEdit"
  >
    <div
      class="w-full max-w-2xl rounded-2xl border border-[#4F46E5]/20 bg-white p-6 shadow-2xl transition-all duration-300 dark:border-[#818CF8]/30 dark:bg-[#11102A] dark:shadow-indigo-950/50"
    >
      <!-- Modal Header -->
      <div class="mb-5 flex items-center justify-between border-b border-[#4F46E5]/10 pb-3 transition-colors duration-300 dark:border-[#818CF8]/20">
        <div>
          <h3 class="text-lg font-bold text-[#1E1B4B] transition-colors duration-300 dark:text-white">
            Edit Student Record
          </h3>
          <p class="text-xs text-[#1E1B4B]/60 transition-colors duration-300 dark:text-white/60">
            Update the student's attendance details below.
          </p>
        </div>

        <!-- Close (X) Button -->
        <button
          type="button"
          @click="cancelEdit"
          class="rounded-lg p-1.5 text-[#1E1B4B]/50 transition-colors duration-300 hover:bg-[#4F46E5]/10 hover:text-[#1E1B4B] dark:text-white/50 dark:hover:bg-[#818CF8]/20 dark:hover:text-white"
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
          <label class="mb-1 block text-xs font-semibold text-[#1E1B4B]/80 transition-colors duration-300 dark:text-white/80">
            Student Number
          </label>
          <input
            v-model="studentNumber"
            type="text"
            class="w-full rounded-xl border border-[#4F46E5]/15 bg-[#F5F3FF] px-3 py-2.5 text-sm text-[#1E1B4B] outline-none transition-all duration-300 placeholder:text-[#1E1B4B]/40 hover:border-[#4F46E5]/30 focus:border-[#4F46E5] focus:bg-white focus:ring-4 focus:ring-[#4F46E5]/10 dark:border-[#818CF8]/20 dark:bg-[#1E1B4B] dark:text-white dark:placeholder:text-white/40 dark:hover:border-[#818CF8]/40 dark:focus:border-[#818CF8] dark:focus:bg-[#171638] dark:focus:ring-4 dark:focus:ring-[#818CF8]/10"
          />
        </div>

        <!-- Student Name -->
        <div>
          <label class="mb-1 block text-xs font-semibold text-[#1E1B4B]/80 transition-colors duration-300 dark:text-white/80">
            Student Name
          </label>
          <input
            v-model="studentName"
            type="text"
            class="w-full rounded-xl border border-[#4F46E5]/15 bg-[#F5F3FF] px-3 py-2.5 text-sm text-[#1E1B4B] outline-none transition-all duration-300 placeholder:text-[#1E1B4B]/40 hover:border-[#4F46E5]/30 focus:border-[#4F46E5] focus:bg-white focus:ring-4 focus:ring-[#4F46E5]/10 dark:border-[#818CF8]/20 dark:bg-[#1E1B4B] dark:text-white dark:placeholder:text-white/40 dark:hover:border-[#818CF8]/40 dark:focus:border-[#818CF8] dark:focus:bg-[#171638] dark:focus:ring-4 dark:focus:ring-[#818CF8]/10"
          />
        </div>

        <!-- Section -->
        <div>
          <label class="mb-1 block text-xs font-semibold text-[#1E1B4B]/80 transition-colors duration-300 dark:text-white/80">
            Section
          </label>
          <input
            v-model="section"
            type="text"
            class="w-full rounded-xl border border-[#4F46E5]/15 bg-[#F5F3FF] px-3 py-2.5 text-sm text-[#1E1B4B] outline-none transition-all duration-300 placeholder:text-[#1E1B4B]/40 hover:border-[#4F46E5]/30 focus:border-[#4F46E5] focus:bg-white focus:ring-4 focus:ring-[#4F46E5]/10 dark:border-[#818CF8]/20 dark:bg-[#1E1B4B] dark:text-white dark:placeholder:text-white/40 dark:hover:border-[#818CF8]/40 dark:focus:border-[#818CF8] dark:focus:bg-[#171638] dark:focus:ring-4 dark:focus:ring-[#818CF8]/10"
          />
        </div>

        <!-- Date -->
        <div>
          <label class="mb-1 block text-xs font-semibold text-[#1E1B4B]/80 transition-colors duration-300 dark:text-white/80">
            Date
          </label>
          <input
            v-model="date"
            type="date"
            class="w-full rounded-xl border border-[#4F46E5]/15 bg-[#F5F3FF] px-3 py-2.5 text-sm text-[#1E1B4B] outline-none transition-all duration-300 hover:border-[#4F46E5]/30 focus:border-[#4F46E5] focus:bg-white focus:ring-4 focus:ring-[#4F46E5]/10 dark:border-[#818CF8]/20 dark:bg-[#1E1B4B] dark:text-white dark:hover:border-[#818CF8]/40 dark:focus:border-[#818CF8] dark:focus:bg-[#171638] dark:focus:ring-4 dark:focus:ring-[#818CF8]/10"
          />
        </div>

        <!-- Status -->
        <div class="sm:col-span-2">
          <label class="mb-1 block text-xs font-semibold text-[#1E1B4B]/80 transition-colors duration-300 dark:text-white/80">
            Status
          </label>
          <select
            v-model="status"
            class="w-full rounded-xl border border-[#4F46E5]/15 bg-[#F5F3FF] px-3 py-2.5 text-sm text-[#1E1B4B] outline-none transition-all duration-300 hover:border-[#4F46E5]/30 focus:border-[#4F46E5] focus:bg-white focus:ring-4 focus:ring-[#4F46E5]/10 dark:border-[#818CF8]/20 dark:bg-[#1E1B4B] dark:text-white dark:hover:border-[#818CF8]/40 dark:focus:border-[#818CF8] dark:focus:bg-[#171638] dark:focus:ring-4 dark:focus:ring-[#818CF8]/10"
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
            class="rounded-xl border border-[#4F46E5]/20 px-4 py-2.5 text-sm font-semibold text-[#1E1B4B] transition-all duration-300 hover:bg-[#818CF8]/10 dark:border-[#818CF8]/20 dark:text-white dark:hover:bg-[#818CF8]/10"
          >
            Cancel
          </button>
          <button
            type="submit"
            class="rounded-xl bg-[#4F46E5] px-5 py-2.5 text-sm font-semibold text-white shadow-md shadow-[#4F46E5]/20 transition-all duration-300 hover:-translate-y-0.5 hover:bg-[#4338CA] hover:shadow-lg dark:bg-[#818CF8] dark:text-[#1E1B4B] dark:shadow-[#818CF8]/20 dark:hover:bg-[#A5B4FC]"
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
    class="border-b border-[#4F46E5]/10 bg-white p-4 transition-all duration-300 ease-out dark:border-[#818CF8]/20 dark:bg-[#1E1B4B] sm:p-5"
  >
    <div class="mb-4">
      <h3 class="text-sm font-bold text-[#1E1B4B] transition-colors duration-300 dark:text-white sm:text-base">
        Add Attendance Record
      </h3>
      <p class="mt-0.5 text-xs text-[#1E1B4B]/60 transition-colors duration-300 dark:text-white/60">
        Enter the student attendance information below.
      </p>
    </div>

    <form @submit.prevent="submitForm" class="grid grid-cols-1 gap-3 sm:grid-cols-2 lg:grid-cols-6">
      <div>
        <label class="mb-1.5 block text-xs font-semibold text-[#1E1B4B]/80 dark:text-white/80">Student Number</label>
        <input
          v-model="studentNumber"
          type="text"
          placeholder="E.g. 12342026"
          class="w-full rounded-xl border border-[#4F46E5]/15 bg-[#F5F3FF] px-3 py-2.5 text-sm text-[#1E1B4B] outline-none transition-all duration-300 ease-out placeholder:text-[#1E1B4B]/40 hover:border-[#4F46E5]/30 focus:border-[#4F46E5] focus:bg-white focus:ring-4 focus:ring-[#4F46E5]/10 dark:border-[#818CF8]/20 dark:bg-[#11102A] dark:text-white dark:placeholder:text-white/40 dark:hover:border-[#818CF8]/40 dark:focus:border-[#818CF8] dark:focus:bg-[#171638] dark:focus:ring-4 dark:focus:ring-[#818CF8]/10"
        />
      </div>

      <div class="sm:col-span-1 lg:col-span-2">
        <label class="mb-1.5 block text-xs font-semibold text-[#1E1B4B]/80 dark:text-white/80">Student Name</label>
        <input
          v-model="studentName"
          type="text"
          placeholder="E.g. Juan Dela Cruz"
          class="w-full rounded-xl border border-[#4F46E5]/15 bg-[#F5F3FF] px-3 py-2.5 text-sm text-[#1E1B4B] outline-none transition-all duration-300 ease-out placeholder:text-[#1E1B4B]/40 hover:border-[#4F46E5]/30 focus:border-[#4F46E5] focus:bg-white focus:ring-4 focus:ring-[#4F46E5]/10 dark:border-[#818CF8]/20 dark:bg-[#11102A] dark:text-white dark:placeholder:text-white/40 dark:hover:border-[#818CF8]/40 dark:focus:border-[#818CF8] dark:focus:bg-[#171638] dark:focus:ring-4 dark:focus:ring-[#818CF8]/10"
        />
      </div>

      <div>
        <label class="mb-1.5 block text-xs font-semibold text-[#1E1B4B]/80 dark:text-white/80">Section</label>
        <input
          v-model="section"
          type="text"
          placeholder="E.g. BSCS 1Z"
          class="w-full rounded-xl border border-[#4F46E5]/15 bg-[#F5F3FF] px-3 py-2.5 text-sm text-[#1E1B4B] outline-none transition-all duration-300 ease-out placeholder:text-[#1E1B4B]/40 hover:border-[#4F46E5]/30 focus:border-[#4F46E5] focus:bg-white focus:ring-4 focus:ring-[#4F46E5]/10 dark:border-[#818CF8]/20 dark:bg-[#11102A] dark:text-white dark:placeholder:text-white/40 dark:hover:border-[#818CF8]/40 dark:focus:border-[#818CF8] dark:focus:bg-[#171638] dark:focus:ring-4 dark:focus:ring-[#818CF8]/10"
        />
      </div>

      <div>
        <label class="mb-1.5 block text-xs font-semibold text-[#1E1B4B]/80 dark:text-white/80">Date</label>
        <input
          v-model="date"
          type="date"
          class="w-full rounded-xl border border-[#4F46E5]/15 bg-[#F5F3FF] px-3 py-2.5 text-sm text-[#1E1B4B] outline-none transition-all duration-300 ease-out hover:border-[#4F46E5]/30 focus:border-[#4F46E5] focus:bg-white focus:ring-4 focus:ring-[#4F46E5]/10 dark:border-[#818CF8]/20 dark:bg-[#11102A] dark:text-white dark:hover:border-[#818CF8]/40 dark:focus:border-[#818CF8] dark:focus:bg-[#171638] dark:focus:ring-4 dark:focus:ring-[#818CF8]/10"
        />
      </div>

      <div>
        <label class="mb-1.5 block text-xs font-semibold text-[#1E1B4B]/80 dark:text-white/80">Status</label>
        <select
          v-model="status"
          class="w-full rounded-xl border border-[#4F46E5]/15 bg-[#F5F3FF] px-3 py-2.5 text-sm text-[#1E1B4B] outline-none transition-all duration-300 ease-out hover:border-[#4F46E5]/30 focus:border-[#4F46E5] focus:bg-white focus:ring-4 focus:ring-[#4F46E5]/10 dark:border-[#818CF8]/20 dark:bg-[#11102A] dark:text-white dark:hover:border-[#818CF8]/40 dark:focus:border-[#818CF8] dark:focus:bg-[#171638] dark:focus:ring-4 dark:focus:ring-[#818CF8]/10"
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
          class="flex-1 rounded-xl bg-[#4F46E5] px-4 py-2.5 text-sm font-semibold text-white shadow-md shadow-[#4F46E5]/20 transition-all duration-300 ease-out hover:-translate-y-0.5 hover:bg-[#4338CA] hover:shadow-lg hover:shadow-[#4F46E5]/25 active:translate-y-0 active:scale-[0.97] dark:bg-[#818CF8] dark:text-[#1E1B4B] dark:shadow-[#818CF8]/20 dark:hover:bg-[#A5B4FC] dark:hover:shadow-lg"
        >
          Add Record
        </button>
      </div>
    </form>
  </div>
</template>