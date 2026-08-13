<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  records: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits([
  'delete-record',
  'edit-record'
])

// ========================================
// SEARCH AND FILTER STATE
// ========================================

const searchInput = ref('')
const searchQuery = ref('')
const isTyping = ref(false)
const isFocused = ref(false)

const selectedStatus = ref('All Statuses')
const selectedSection = ref('All Sections')
const selectedDate = ref('All Dates')

// ========================================
// DEBOUNCE WITH VISUAL FEEDBACK
// ========================================

let debounceTimer = null

watch(searchInput, (newValue) => {
  isTyping.value = true
  clearTimeout(debounceTimer)
  
  debounceTimer = setTimeout(() => {
    searchQuery.value = newValue
    isTyping.value = false
  }, 150) // Ultra-responsive 150ms delay
})

function clearSearchInput() {
  searchInput.value = ''
  searchQuery.value = ''
  isTyping.value = false
}

// ========================================
// DYNAMIC FILTER OPTIONS
// ========================================

const availableSections = computed(() => {
  const sections = props.records
    .map(r => r.section)
    .filter((section, index, self) => section && self.indexOf(section) === index)
  return sections.sort()
})

const availableDates = computed(() => {
  const dates = props.records
    .map(r => r.date)
    .filter((date, index, self) => date && self.indexOf(date) === index)
  return dates.sort((a, b) => new Date(b) - new Date(a))
})

// ========================================
// FILTER LOGIC
// ========================================

const filteredRecords = computed(() => {
  const search = searchQuery.value.toLowerCase().trim()

  return props.records.filter(record => {
    const matchesSearch =
      !search ||
      record.studentName.toLowerCase().includes(search) ||
      record.studentNumber.toLowerCase().includes(search)

    const matchesStatus =
      selectedStatus.value === 'All Statuses' ||
      record.status === selectedStatus.value

    const matchesSection =
      selectedSection.value === 'All Sections' ||
      record.section === selectedSection.value

    const matchesDate =
      selectedDate.value === 'All Dates' ||
      record.date === selectedDate.value

    return matchesSearch && matchesStatus && matchesSection && matchesDate
  })
})

// Count how many active filters are applied
const activeFilterCount = computed(() => {
  let count = 0
  if (searchInput.value !== '') count++
  if (selectedStatus.value !== 'All Statuses') count++
  if (selectedSection.value !== 'All Sections') count++
  if (selectedDate.value !== 'All Dates') count++
  return count
})

function clearFilters() {
  searchInput.value = ''
  searchQuery.value = ''
  isTyping.value = false
  selectedStatus.value = 'All Statuses'
  selectedSection.value = 'All Sections'
  selectedDate.value = 'All Dates'
}
</script>

<template>
  <div>
    <!-- ========================================
         SEARCH AND FILTERS BAR
    ======================================== -->
    <div
      class="flex flex-col gap-4 border-b border-[#4F46E5]/10 bg-white p-4 transition-all duration-300 dark:border-[#818CF8]/20 dark:bg-[#1E1B4B] sm:p-5"
    >
      <!-- Top Row: Search Input & Reset Pill -->
      <div class="flex flex-col gap-3 sm:flex-row sm:items-center">
        <!-- Search Field with Animated Glow -->
        <div class="relative flex-1">
          <div
            class="pointer-events-none absolute -inset-0.5 rounded-xl bg-gradient-to-r from-[#4F46E5] to-[#818CF8] opacity-0 blur-sm transition-opacity duration-300 dark:from-[#818CF8] dark:to-[#A5B4FC]"
            :class="{ 'opacity-30': isFocused }"
          ></div>

          <input
            v-model="searchInput"
            type="text"
            @focus="isFocused = true"
            @blur="isFocused = false"
            placeholder="Search student name or student number..."
            class="relative w-full rounded-xl border border-[#4F46E5]/15 bg-[#F5F3FF] py-2.5 pl-4 pr-10 text-sm text-[#1E1B4B] outline-none transition-all duration-300 ease-out placeholder:text-[#1E1B4B]/40 hover:border-[#4F46E5]/30 focus:border-[#4F46E5] focus:bg-white dark:border-[#818CF8]/20 dark:bg-[#11102A] dark:text-white dark:placeholder:text-white/40 dark:hover:border-[#818CF8]/40 dark:focus:border-[#818CF8] dark:focus:bg-[#171638]"
          />

          <!-- Live Typing Pulse Indicator -->
          <div
            v-if="isTyping"
            class="pointer-events-none absolute right-3 top-1/2 flex h-4 w-4 -translate-y-1/2 items-center justify-center"
          >
            <span class="h-2.5 w-2.5 animate-ping rounded-full bg-[#4F46E5] opacity-75 dark:bg-[#818CF8]"></span>
          </div>

          <!-- Clear Search Icon Button -->
          <button
            v-else-if="searchInput"
            type="button"
            @click="clearSearchInput"
            class="absolute right-3 top-1/2 flex h-5 w-5 -translate-y-1/2 items-center justify-center rounded-full bg-[#1E1B4B]/10 text-xs font-bold text-[#1E1B4B]/60 transition-all duration-200 hover:scale-110 hover:bg-[#1E1B4B]/20 hover:text-[#1E1B4B] active:scale-95 dark:bg-white/10 dark:text-white/60 dark:hover:bg-white/20 dark:hover:text-white"
            title="Clear search"
          >
            ✕
          </button>

          <!-- Default Search Icon -->
          <span
            v-else
            class="pointer-events-none absolute right-3 top-1/2 flex h-4 w-4 -translate-y-1/2 items-center justify-center text-[#1E1B4B]/40 dark:text-white/40"
          >
            <span class="relative block h-3.5 w-3.5 rounded-full border-2 border-current">
              <span class="absolute -bottom-1 -right-1 h-2 w-0.5 rotate-[-45deg] bg-current"></span>
            </span>
          </span>
        </div>

        <!-- Animated Reset Pill with Filter Badge Count -->
        <Transition
          enter-active-class="transition-all duration-300 ease-out"
          enter-from-class="opacity-0 scale-90 -translate-x-2"
          enter-to-class="opacity-100 scale-100 translate-x-0"
          leave-active-class="transition-all duration-200 ease-in"
          leave-from-class="opacity-100 scale-100 translate-x-0"
          leave-to-class="opacity-0 scale-90 -translate-x-2"
        >
          <button
            v-if="activeFilterCount > 0"
            type="button"
            @click="clearFilters"
            class="inline-flex items-center justify-center gap-2 rounded-xl border border-[#4F46E5]/20 bg-[#F5F3FF] px-4 py-2.5 text-xs font-semibold text-[#4F46E5] shadow-sm transition-all duration-300 hover:border-[#4F46E5] hover:bg-[#4F46E5] hover:text-white hover:shadow-md active:scale-95 dark:border-[#818CF8]/30 dark:bg-[#11102A] dark:text-[#A5B4FC] dark:hover:border-[#818CF8] dark:hover:bg-[#818CF8] dark:hover:text-[#11102A]"
          >
            <span class="flex h-4 w-4 items-center justify-center rounded-full bg-[#4F46E5]/15 text-[10px] font-bold text-[#4F46E5] group-hover:bg-white group-hover:text-[#4F46E5] dark:bg-[#818CF8]/20 dark:text-[#A5B4FC]">
              {{ activeFilterCount }}
            </span>
            <span>Reset Filters</span>
          </button>
        </Transition>
      </div>

      <!-- Bottom Row: Filter Dropdowns -->
      <div class="grid grid-cols-1 gap-3 sm:grid-cols-3">
        <!-- Section / Course Filter -->
        <div class="flex flex-col gap-1">
          <label class="text-[11px] font-bold uppercase tracking-wider text-[#1E1B4B]/60 dark:text-white/60">
            Course & Section
          </label>
          <select
            v-model="selectedSection"
            class="w-full rounded-xl border border-[#4F46E5]/15 bg-[#F5F3FF] px-3 py-2 text-sm text-[#1E1B4B] outline-none transition-all duration-300 ease-out hover:border-[#4F46E5]/30 focus:border-[#4F46E5] focus:bg-white dark:border-[#818CF8]/20 dark:bg-[#11102A] dark:text-white dark:hover:border-[#818CF8]/40 dark:focus:border-[#818CF8] dark:focus:bg-[#171638]"
          >
            <option value="All Sections">All Courses / Sections</option>
            <option v-for="sec in availableSections" :key="sec" :value="sec">
              {{ sec }}
            </option>
          </select>
        </div>

        <!-- Date Filter -->
        <div class="flex flex-col gap-1">
          <label class="text-[11px] font-bold uppercase tracking-wider text-[#1E1B4B]/60 dark:text-white/60">
            Date Logged
          </label>
          <select
            v-model="selectedDate"
            class="w-full rounded-xl border border-[#4F46E5]/15 bg-[#F5F3FF] px-3 py-2 text-sm text-[#1E1B4B] outline-none transition-all duration-300 ease-out hover:border-[#4F46E5]/30 focus:border-[#4F46E5] focus:bg-white dark:border-[#818CF8]/20 dark:bg-[#11102A] dark:text-white dark:hover:border-[#818CF8]/40 dark:focus:border-[#818CF8] dark:focus:bg-[#171638]"
          >
            <option value="All Dates">Date</option>
            <option v-for="d in availableDates" :key="d" :value="d">
              {{ d }}
            </option>
          </select>
        </div>

        <!-- Status Filter -->
        <div class="flex flex-col gap-1">
          <label class="text-[11px] font-bold uppercase tracking-wider text-[#1E1B4B]/60 dark:text-white/60">
            Attendance Status
          </label>
          <select
            v-model="selectedStatus"
            class="w-full rounded-xl border border-[#4F46E5]/15 bg-[#F5F3FF] px-3 py-2 text-sm text-[#1E1B4B] outline-none transition-all duration-300 ease-out hover:border-[#4F46E5]/30 focus:border-[#4F46E5] focus:bg-white dark:border-[#818CF8]/20 dark:bg-[#11102A] dark:text-white dark:hover:border-[#818CF8]/40 dark:focus:border-[#818CF8] dark:focus:bg-[#171638]"
          >
            <option value="All Statuses">Status</option>
            <option value="Present">Present</option>
            <option value="Late">Late</option>
            <option value="Absent">Absent</option>
            <option value="Excused">Excused</option>
          </select>
        </div>
      </div>
    </div>

    <!-- ========================================
         TABLE WITH STAGGERED FLUID TRANSITIONS
    ======================================== -->
    <div class="overflow-x-auto">
      <table class="w-full min-w-[850px]">
        <thead>
          <tr
            class="border-b border-[#4F46E5]/10 bg-[#F5F3FF] transition-colors duration-300 dark:border-[#818CF8]/20 dark:bg-[#11102A]"
          >
            <th class="px-5 py-3 text-left text-xs font-bold uppercase tracking-wider text-[#1E1B4B]/60 dark:text-white/60">
              Student Information
            </th>
            <th class="px-5 py-3 text-left text-xs font-bold uppercase tracking-wider text-[#1E1B4B]/60 dark:text-white/60">
              Section
            </th>
            <th class="px-5 py-3 text-left text-xs font-bold uppercase tracking-wider text-[#1E1B4B]/60 dark:text-white/60">
              Date
            </th>
            <th class="px-5 py-3 text-left text-xs font-bold uppercase tracking-wider text-[#1E1B4B]/60 dark:text-white/60">
              Status
            </th>
            <th class="px-5 py-3 text-left text-xs font-bold uppercase tracking-wider text-[#1E1B4B]/60 dark:text-white/60">
              Actions
            </th>
          </tr>
        </thead>

        <!-- FLUID TRANSITION GROUP -->
        <TransitionGroup
          tag="tbody"
          name="fluid-table"
          class="relative"
        >
          <!-- NO RECORDS STATE -->
          <tr v-if="props.records.length === 0" key="no-records">
            <td colspan="5" class="px-5 py-16 text-center">
              <div
                class="mx-auto flex h-14 w-14 items-center justify-center rounded-2xl border border-[#4F46E5]/10 bg-[#F5F3FF] text-[#4F46E5]/60 transition-all duration-300 hover:-translate-y-1 hover:border-[#818CF8]/30 hover:bg-[#818CF8]/10 hover:text-[#4F46E5] dark:border-[#818CF8]/20 dark:bg-[#11102A] dark:text-[#818CF8]/60 dark:hover:border-[#818CF8]/40 dark:hover:bg-[#818CF8]/10 dark:hover:text-[#A5B4FC]"
              >
                <span class="relative h-6 w-5 rounded-md border-2 border-current">
                  <span class="absolute -top-2 left-1/2 h-3 w-3 -translate-x-1/2 rounded-sm border-2 border-current bg-[#F5F3FF] dark:bg-[#11102A]"></span>
                  <span class="absolute left-1 top-2 h-0.5 w-3 bg-current"></span>
                  <span class="absolute left-1 top-4 h-0.5 w-3 bg-current"></span>
                </span>
              </div>
              <h3 class="mt-4 text-sm font-semibold text-[#1E1B4B] dark:text-white">
                No attendance records
              </h3>
              <p class="mt-1 text-xs text-[#1E1B4B]/60 dark:text-white/60">
                Add an attendance record to see it here.
              </p>
            </td>
          </tr>

          <!-- NO MATCHING SEARCH/FILTER RESULTS -->
          <tr v-else-if="filteredRecords.length === 0" key="no-matches">
            <td colspan="5" class="px-5 py-16 text-center">
              <div
                class="mx-auto flex h-14 w-14 items-center justify-center rounded-2xl border border-[#4F46E5]/10 bg-[#F5F3FF] text-[#4F46E5]/60 transition-all duration-300 dark:border-[#818CF8]/20 dark:bg-[#11102A] dark:text-[#818CF8]/60"
              >
                <span class="relative block h-5 w-5 rounded-full border-2 border-current">
                  <span class="absolute -bottom-1 -right-1 h-2.5 w-0.5 rotate-[-45deg] bg-current"></span>
                </span>
              </div>
              <h3 class="mt-4 text-sm font-semibold text-[#1E1B4B] dark:text-white">
                No matching records
              </h3>
              <p class="mt-1 text-xs text-[#1E1B4B]/60 dark:text-white/60">
                Try changing your search, course/section, date, or status filters.
              </p>
              <button
                type="button"
                @click="clearFilters"
                class="mt-4 rounded-xl border border-[#4F46E5]/20 px-4 py-2 text-xs font-semibold text-[#4F46E5] transition-all duration-300 hover:-translate-y-0.5 hover:border-[#4F46E5]/30 hover:bg-[#818CF8]/10 active:translate-y-0 active:scale-95 dark:border-[#818CF8]/20 dark:text-[#A5B4FC] dark:hover:border-[#818CF8]/40 dark:hover:bg-[#818CF8]/10"
              >
                Clear All Filters
              </button>
            </td>
          </tr>

          <!-- RECORD ROWS -->
          <tr
            v-for="(record, index) in filteredRecords"
            :key="record.id"
            :style="{ '--delay': `${(index % 10) * 0.03}s` }"
            class="fluid-row border-b border-[#4F46E5]/10 transition-all duration-300 ease-out hover:bg-[#F5F3FF]/80 hover:pl-2 dark:border-[#818CF8]/10 dark:hover:bg-[#171638]/80"
          >
            <!-- Student -->
            <td class="px-5 py-4">
              <p class="text-sm font-semibold text-[#1E1B4B] transition-colors duration-300 dark:text-white">
                {{ record.studentName }}
              </p>
              <p class="mt-1 text-xs text-[#1E1B4B]/50 dark:text-white/50">
                ID: {{ record.studentNumber }}
              </p>
            </td>

            <!-- Section -->
            <td class="px-5 py-4 text-sm text-[#1E1B4B]/70 dark:text-white/70">
              {{ record.section }}
            </td>

            <!-- Date -->
            <td class="px-5 py-4 text-sm text-[#1E1B4B]/70 dark:text-white/70">
              {{ record.date }}
            </td>

            <!-- Status Badge -->
            <td class="px-5 py-4">
              <span
                class="inline-flex rounded-full px-3 py-1 text-xs font-semibold transition-all duration-300 hover:scale-105"
                :class="{
                  'bg-emerald-100 text-emerald-700 dark:bg-emerald-950/50 dark:text-emerald-400':
                    record.status === 'Present',
                  'bg-amber-100 text-amber-700 dark:bg-amber-950/50 dark:text-amber-400':
                    record.status === 'Late',
                  'bg-red-100 text-red-700 dark:bg-red-950/50 dark:text-red-400':
                    record.status === 'Absent',
                  'bg-sky-100 text-sky-700 dark:bg-sky-950/50 dark:text-sky-400':
                    record.status === 'Excused'
                }"
              >
                {{ record.status }}
              </span>
            </td>

            <!-- Actions -->
            <td class="px-5 py-4">
              <div class="flex gap-2">
                <button
                  type="button"
                  @click="emit('edit-record', record)"
                  class="rounded-xl border border-[#4F46E5]/20 px-3 py-1.5 text-xs font-semibold text-[#4F46E5] transition-all duration-300 ease-out hover:-translate-y-0.5 hover:border-[#4F46E5] hover:bg-[#4F46E5] hover:text-white hover:shadow-md hover:shadow-[#4F46E5]/15 active:translate-y-0 active:scale-95 dark:border-[#818CF8]/30 dark:text-[#A5B4FC] dark:hover:border-[#818CF8] dark:hover:bg-[#818CF8] dark:hover:text-[#1E1B4B]"
                >
                  Edit
                </button>

                <button
                  type="button"
                  @click="emit('delete-record', record.id)"
                  class="rounded-xl border border-red-200 px-3 py-1.5 text-xs font-semibold text-red-600 transition-all duration-300 ease-out hover:-translate-y-0.5 hover:border-red-300 hover:bg-red-50 hover:shadow-md hover:shadow-red-500/10 active:translate-y-0 active:scale-95 dark:border-red-900/50 dark:text-red-400 dark:hover:border-red-800 dark:hover:bg-red-950/30"
                >
                  Delete
                </button>
              </div>
            </td>
          </tr>
        </TransitionGroup>
      </table>
    </div>
  </div>
</template>

<style scoped>
/* Smooth FLIP Animations for Row Movement */
.fluid-table-move,
.fluid-table-enter-active,
.fluid-table-leave-active {
  transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1);
}

.fluid-table-enter-from {
  opacity: 0;
  transform: translateY(8px) scale(0.99);
}

.fluid-table-leave-to {
  opacity: 0;
  transform: translateY(-8px) scale(0.99);
}

/* Staggered Delay Effect on Entrance */
.fluid-row {
  animation-delay: var(--delay, 0s);
}

/* Fix table row position during exit transition */
.fluid-table-leave-active {
  position: absolute;
  width: 100%;
}
</style>