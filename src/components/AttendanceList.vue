<script setup>
import { ref, computed, watch, onUnmounted } from 'vue'

const props = defineProps({
  records: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['delete-record', 'edit-record'])

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
// DEBOUNCE WITH MEMORY CLEANUP
// ========================================

let debounceTimer = null

watch(searchInput, (newValue) => {
  isTyping.value = true
  clearTimeout(debounceTimer)
  
  debounceTimer = setTimeout(() => {
    searchQuery.value = newValue
    isTyping.value = false
  }, 150)
})

onUnmounted(() => {
  if (debounceTimer) clearTimeout(debounceTimer)
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
  const unique = new Set(props.records.map(r => r.section).filter(Boolean))
  return Array.from(unique).sort()
})

const availableDates = computed(() => {
  const unique = new Set(props.records.map(r => r.date).filter(Boolean))
  return Array.from(unique).sort((a, b) => new Date(b) - new Date(a))
})

// ========================================
// FILTER LOGIC WITH SAFEGUARDS
// ========================================

const filteredRecords = computed(() => {
  const search = searchQuery.value.toLowerCase().trim()

  return props.records.filter(record => {
    const name = record.studentName?.toLowerCase() || ''
    const number = record.studentNumber?.toLowerCase() || ''

    const matchesSearch =
      !search ||
      name.includes(search) ||
      number.includes(search)

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
  <div class="bg-white text-black dark:bg-black dark:text-white transition-colors duration-300">
    <!-- SEARCH AND FILTERS BAR -->
    <div class="flex flex-col gap-4 border-b border-neutral-200 bg-neutral-50 p-4 transition-all duration-300 dark:border-neutral-800 dark:bg-neutral-950 sm:p-5">
      <!-- Top Row: Search Input & Reset Pill -->
      <div class="flex flex-col gap-3 sm:flex-row sm:items-center">
        <div class="relative flex-1">
          <!-- Focus Border Glow -->
          <div
            class="pointer-events-none absolute -inset-0.5 rounded-xl bg-gradient-to-r from-neutral-400 to-neutral-600 opacity-0 blur-sm transition-opacity duration-300 dark:from-neutral-600 dark:to-neutral-400"
            :class="{ 'opacity-25': isFocused }"
          ></div>

          <input
            v-model="searchInput"
            type="text"
            @focus="isFocused = true"
            @blur="isFocused = false"
            placeholder="Search student name or student number..."
            class="relative w-full rounded-xl border border-neutral-300 bg-white py-2.5 pl-4 pr-10 text-sm text-neutral-900 outline-none transition-all duration-300 ease-out placeholder:text-neutral-400 hover:border-neutral-400 focus:border-black dark:border-neutral-800 dark:bg-neutral-900 dark:text-white dark:placeholder:text-neutral-500 dark:hover:border-neutral-700 dark:focus:border-white"
          />

          <!-- Live Typing Pulse Indicator -->
          <div
            v-if="isTyping"
            class="pointer-events-none absolute right-3 top-1/2 flex h-4 w-4 -translate-y-1/2 items-center justify-center"
          >
            <span class="h-2.5 w-2.5 animate-ping rounded-full bg-black opacity-75 dark:bg-white"></span>
          </div>

          <!-- Clear Search Button -->
          <button
            v-else-if="searchInput"
            type="button"
            @click="clearSearchInput"
            class="absolute right-3 top-1/2 flex h-5 w-5 -translate-y-1/2 items-center justify-center rounded-full bg-neutral-200 text-xs font-bold text-neutral-700 transition-all duration-200 hover:scale-110 hover:bg-neutral-300 hover:text-black active:scale-95 dark:bg-neutral-800 dark:text-neutral-300 dark:hover:bg-neutral-700 dark:hover:text-white"
            title="Clear search"
          >
            ✕
          </button>

          <!-- Search Glass Icon -->
          <span
            v-else
            class="pointer-events-none absolute right-3 top-1/2 flex h-4 w-4 -translate-y-1/2 items-center justify-center text-neutral-400 dark:text-neutral-500"
          >
            <span class="relative block h-3.5 w-3.5 rounded-full border-2 border-current">
              <span class="absolute -bottom-1 -right-1 h-2 w-0.5 rotate-[-45deg] bg-current"></span>
            </span>
          </span>
        </div>

        <!-- Reset Pill -->
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
            class="inline-flex items-center justify-center gap-2 rounded-xl border border-neutral-300 bg-white px-4 py-2.5 text-xs font-bold text-black shadow-sm transition-all duration-300 hover:border-black hover:bg-black hover:text-white hover:shadow-md active:scale-95 dark:border-neutral-800 dark:bg-neutral-900 dark:text-white dark:hover:border-white dark:hover:bg-white dark:hover:text-black"
          >
            <span class="flex h-4 w-4 items-center justify-center rounded-full bg-neutral-200 text-[10px] font-bold text-black group-hover:bg-neutral-800 group-hover:text-white dark:bg-neutral-800 dark:text-white">
              {{ activeFilterCount }}
            </span>
            <span>Reset Filters</span>
          </button>
        </Transition>
      </div>

      <!-- Bottom Row: Filter Dropdowns -->
      <div class="grid grid-cols-1 gap-3 sm:grid-cols-3">
        <div class="flex flex-col gap-1">
          <label class="text-[11px] font-bold uppercase tracking-wider text-neutral-500 dark:text-neutral-400">
            Course & Section
          </label>
          <select
            v-model="selectedSection"
            class="w-full rounded-xl border border-neutral-300 bg-white px-3 py-2 text-sm text-neutral-900 outline-none transition-all duration-300 ease-out hover:border-neutral-400 focus:border-black dark:border-neutral-800 dark:bg-neutral-900 dark:text-white dark:hover:border-neutral-700 dark:focus:border-white"
          >
            <option value="All Sections">All Courses / Sections</option>
            <option v-for="sec in availableSections" :key="sec" :value="sec">
              {{ sec }}
            </option>
          </select>
        </div>

        <div class="flex flex-col gap-1">
          <label class="text-[11px] font-bold uppercase tracking-wider text-neutral-500 dark:text-neutral-400">
            Date Logged
          </label>
          <select
            v-model="selectedDate"
            class="w-full rounded-xl border border-neutral-300 bg-white px-3 py-2 text-sm text-neutral-900 outline-none transition-all duration-300 ease-out hover:border-neutral-400 focus:border-black dark:border-neutral-800 dark:bg-neutral-900 dark:text-white dark:hover:border-neutral-700 dark:focus:border-white"
          >
            <option value="All Dates">Date</option>
            <option v-for="d in availableDates" :key="d" :value="d">
              {{ d }}
            </option>
          </select>
        </div>

        <div class="flex flex-col gap-1">
          <label class="text-[11px] font-bold uppercase tracking-wider text-neutral-500 dark:text-neutral-400">
            Attendance Status
          </label>
          <select
            v-model="selectedStatus"
            class="w-full rounded-xl border border-neutral-300 bg-white px-3 py-2 text-sm text-neutral-900 outline-none transition-all duration-300 ease-out hover:border-neutral-400 focus:border-black dark:border-neutral-800 dark:bg-neutral-900 dark:text-white dark:hover:border-neutral-700 dark:focus:border-white"
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

    <!-- TABLE -->
    <div class="overflow-x-auto">
      <table class="w-full min-w-[850px]">
        <thead>
          <tr class="border-b border-neutral-200 bg-neutral-100 transition-colors duration-300 dark:border-neutral-800 dark:bg-neutral-900">
            <th class="px-5 py-3 text-left text-xs font-bold uppercase tracking-wider text-neutral-600 dark:text-neutral-400">
              Student Information
            </th>
            <th class="px-5 py-3 text-left text-xs font-bold uppercase tracking-wider text-neutral-600 dark:text-neutral-400">
              Section
            </th>
            <th class="px-5 py-3 text-left text-xs font-bold uppercase tracking-wider text-neutral-600 dark:text-neutral-400">
              Date
            </th>
            <th class="px-5 py-3 text-left text-xs font-bold uppercase tracking-wider text-neutral-600 dark:text-neutral-400">
              Status
            </th>
            <th class="px-5 py-3 text-left text-xs font-bold uppercase tracking-wider text-neutral-600 dark:text-neutral-400">
              Actions
            </th>
          </tr>
        </thead>

        <TransitionGroup
          tag="tbody"
          name="fluid-table"
          class="relative"
        >
          <!-- NO RECORDS -->
          <tr v-if="props.records.length === 0" key="no-records">
            <td colspan="5" class="px-5 py-16 text-center">
              <div class="mx-auto flex h-14 w-14 items-center justify-center rounded-2xl border border-neutral-300 bg-neutral-100 text-neutral-600 transition-all duration-300 hover:-translate-y-1 hover:border-black hover:bg-neutral-200 dark:border-neutral-800 dark:bg-neutral-900 dark:text-neutral-400 dark:hover:border-white dark:hover:bg-neutral-800">
                <span class="relative h-6 w-5 rounded-md border-2 border-current">
                  <span class="absolute -top-2 left-1/2 h-3 w-3 -translate-x-1/2 rounded-sm border-2 border-current bg-neutral-100 dark:bg-neutral-900"></span>
                  <span class="absolute left-1 top-2 h-0.5 w-3 bg-current"></span>
                  <span class="absolute left-1 top-4 h-0.5 w-3 bg-current"></span>
                </span>
              </div>
              <h3 class="mt-4 text-sm font-semibold text-neutral-900 dark:text-white">
                No attendance records
              </h3>
              <p class="mt-1 text-xs text-neutral-500 dark:text-neutral-400">
                Add an attendance record to see it here.
              </p>
            </td>
          </tr>

          <!-- NO FILTER MATCHES -->
          <tr v-else-if="filteredRecords.length === 0" key="no-matches">
            <td colspan="5" class="px-5 py-16 text-center">
              <div class="mx-auto flex h-14 w-14 items-center justify-center rounded-2xl border border-neutral-300 bg-neutral-100 text-neutral-600 transition-all duration-300 dark:border-neutral-800 dark:bg-neutral-900 dark:text-neutral-400">
                <span class="relative block h-5 w-5 rounded-full border-2 border-current">
                  <span class="absolute -bottom-1 -right-1 h-2.5 w-0.5 rotate-[-45deg] bg-current"></span>
                </span>
              </div>
              <h3 class="mt-4 text-sm font-semibold text-neutral-900 dark:text-white">
                No matching records
              </h3>
              <p class="mt-1 text-xs text-neutral-500 dark:text-neutral-400">
                Try changing your search, course/section, date, or status filters.
              </p>
              <button
                type="button"
                @click="clearFilters"
                class="mt-4 rounded-xl border border-neutral-300 px-4 py-2 text-xs font-semibold text-black transition-all duration-300 hover:-translate-y-0.5 hover:border-black hover:bg-neutral-100 active:translate-y-0 active:scale-95 dark:border-neutral-700 dark:text-white dark:hover:border-white dark:hover:bg-neutral-800"
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
            class="fluid-row border-b border-neutral-200 transition-all duration-300 ease-out hover:bg-neutral-100/70 hover:pl-2 dark:border-neutral-800 dark:hover:bg-neutral-900/70"
          >
            <td class="px-5 py-4">
              <p class="text-sm font-semibold text-neutral-900 transition-colors duration-300 dark:text-white">
                {{ record.studentName }}
              </p>
              <p class="mt-1 text-xs text-neutral-500 dark:text-neutral-400">
                ID: {{ record.studentNumber }}
              </p>
            </td>

            <td class="px-5 py-4 text-sm text-neutral-700 dark:text-neutral-300">
              {{ record.section }}
            </td>

            <td class="px-5 py-4 text-sm text-neutral-700 dark:text-neutral-300">
              {{ record.date }}
            </td>

            <!-- Monochrome Status Badges -->
            <td class="px-5 py-4">
              <span
                class="inline-flex rounded-full px-3 py-1 text-xs font-bold uppercase tracking-wider transition-all duration-300 hover:scale-105"
                :class="{
                  'bg-black text-white dark:bg-white dark:text-black':
                    record.status === 'Present',
                  'bg-neutral-200 text-neutral-800 dark:bg-neutral-800 dark:text-neutral-200':
                    record.status === 'Late',
                  'border border-neutral-400 text-neutral-600 dark:border-neutral-600 dark:text-neutral-400':
                    record.status === 'Absent',
                  'bg-neutral-100 text-neutral-600 border border-neutral-200 dark:bg-neutral-900 dark:text-neutral-400 dark:border-neutral-800':
                    record.status === 'Excused'
                }"
              >
                {{ record.status }}
              </span>
            </td>

            <td class="px-5 py-4">
              <div class="flex gap-2">
                <button
                  type="button"
                  @click="emit('edit-record', record)"
                  class="rounded-xl border border-neutral-300 px-3 py-1.5 text-xs font-semibold text-neutral-900 transition-all duration-300 ease-out hover:-translate-y-0.5 hover:border-black hover:bg-black hover:text-white active:translate-y-0 active:scale-95 dark:border-neutral-700 dark:text-neutral-200 dark:hover:border-white dark:hover:bg-white dark:hover:text-black"
                >
                  Edit
                </button>

                <button
                  type="button"
                  @click="emit('delete-record', record.id)"
                  class="rounded-xl border border-neutral-200 bg-neutral-100 px-3 py-1.5 text-xs font-semibold text-neutral-500 transition-all duration-300 ease-out hover:-translate-y-0.5 hover:border-black hover:bg-black hover:text-white active:translate-y-0 active:scale-95 dark:border-neutral-800 dark:bg-neutral-900 dark:text-neutral-400 dark:hover:border-white dark:hover:bg-white dark:hover:text-black"
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

.fluid-row {
  animation-delay: var(--delay, 0s);
}

.fluid-table-leave-active {
  position: absolute;
  left: 0;
  right: 0;
}
</style>