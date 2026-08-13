<script setup>
defineProps({
  darkMode: {
    type: Boolean,
    default: true
  }
})

const emit = defineEmits(['toggle-theme'])
</script>

<template>
  <header
    class="border-b border-[#4F46E5]/10 bg-[#F5F3FF] transition-colors duration-300 dark:border-[#818CF8]/20 dark:bg-[#1E1B4B]"
  >
    <div
      class="mx-auto flex w-full max-w-[1250px] items-center justify-between px-3 py-3 sm:px-6 sm:py-4 lg:px-8"
    >
      <!-- ========================================
           LOGO AND TITLE
      ======================================== -->
      <div class="flex min-w-0 items-center gap-2 sm:gap-3">
        <!-- Logo -->
        <div
          class="flex h-9 w-9 shrink-0 items-center justify-center rounded-lg bg-[#4F46E5] text-xs font-bold text-white shadow-lg shadow-[#4F46E5]/20 transition-all duration-300 dark:bg-[#818CF8] dark:text-[#1E1B4B] dark:shadow-[#818CF8]/20 sm:h-11 sm:w-11 sm:rounded-xl sm:text-sm"
        >
          AM
        </div>

        <!-- Application Title -->
        <div class="min-w-0">
          <h1
            class="truncate text-sm font-bold tracking-tight text-[#1E1B4B] transition-colors duration-300 dark:text-white sm:text-lg"
          >
            Attendance Monitoring System
          </h1>
        </div>
      </div>

      <!-- ========================================
           RIGHT SIDE
      ======================================== -->
      <div class="flex shrink-0 items-center">
        <!-- ========================================
             LIGHT / DARK MODE SWITCH TOGGLE
        ======================================== -->
        <button
          type="button"
          role="switch"
          :aria-checked="darkMode"
          :aria-label="
            darkMode
              ? 'Switch to light mode'
              : 'Switch to dark mode'
          "
          :title="
            darkMode
              ? 'Switch to light mode'
              : 'Switch to dark mode'
          "
          @click="emit('toggle-theme')"
          class="relative flex h-9 w-[68px] shrink-0 items-center rounded-full border p-1 shadow-sm transition-all duration-200 ease-out hover:shadow-md active:scale-95 focus:outline-none focus:ring-2 focus:ring-[#4F46E5]/30 dark:focus:ring-[#818CF8]/30 sm:h-10 sm:w-[74px]"
          :class="
            darkMode
              ? 'border-[#818CF8]/30 bg-[#11102A]'
              : 'border-[#4F46E5]/20 bg-white'
          "
        >
          <!-- Background Sun Icon (Left Side) -->
          <svg
            class="absolute left-2.5 h-4 w-4 transition-opacity duration-200"
            :class="
              darkMode
                ? 'opacity-30 text-white'
                : 'opacity-100 text-[#4F46E5]'
            "
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
            stroke-width="2"
            aria-hidden="true"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z"
            />
          </svg>

          <!-- Background Moon Icon (Right Side) -->
          <svg
            class="absolute right-2.5 h-4 w-4 transition-opacity duration-200"
            :class="
              darkMode
                ? 'opacity-100 text-[#818CF8]'
                : 'opacity-30 text-[#1E1B4B]'
            "
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
            stroke-width="2"
            aria-hidden="true"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z"
            />
          </svg>

          <!-- Sliding Knob -->
          <span
            class="relative z-10 flex h-7 w-7 items-center justify-center rounded-full shadow-md transition-transform duration-200 ease-out sm:h-8 sm:w-8"
            :class="
              darkMode
                ? 'translate-x-[28px] bg-[#818CF8] text-[#1E1B4B] sm:translate-x-[32px]'
                : 'translate-x-0 bg-[#4F46E5] text-white'
            "
          >
            <!-- Moon Icon inside Knob (Dark Mode) -->
            <svg
              v-if="darkMode"
              class="h-4 w-4 sm:h-4.5 sm:w-4.5"
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
              stroke-width="2"
              aria-hidden="true"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z"
              />
            </svg>

            <!-- Sun Icon inside Knob (Light Mode) -->
            <svg
              v-else
              class="h-4 w-4 sm:h-4.5 sm:w-4.5"
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
              stroke-width="2"
              aria-hidden="true"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z"
              />
            </svg>
          </span>
        </button>
      </div>
    </div>
  </header>
</template>