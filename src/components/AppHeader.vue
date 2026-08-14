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
    class="border-b border-black/10 bg-white/90 backdrop-blur-md transition-colors duration-300 dark:border-white/10 dark:bg-black/90"
  >
    <div
      class="mx-auto flex w-full max-w-[1280px] items-center justify-between gap-3 px-4 py-3 sm:px-6 sm:py-4 lg:px-8"
    >
      <!-- ========================================
           LOGO AND TITLE
      ======================================== -->
      <div class="flex min-w-0 items-center gap-2.5 sm:gap-3">
        

        <!-- Application Title -->
        <div class="min-w-0">
          <h1
            class="truncate text-base font-extrabold tracking-tight text-black transition-colors duration-300 dark:text-white sm:text-lg"
          >
            Attendance Monitoring System
          </h1>
        </div>
      </div>

      <!-- ========================================
           LIGHT / DARK MODE SWITCH
      ======================================== -->
      <div class="flex shrink-0 items-center">
        <button
          type="button"
          role="switch"
          :aria-checked="darkMode"
          :aria-label="darkMode ? 'Switch to light mode' : 'Switch to dark mode'"
          :title="darkMode ? 'Switch to light mode' : 'Switch to dark mode'"
          @click="emit('toggle-theme')"
          class="relative flex h-9 w-[68px] shrink-0 items-center rounded-full border p-1 transition-all duration-200 ease-out active:scale-95 focus:outline-none focus:ring-2 focus:ring-black dark:focus:ring-white sm:h-10 sm:w-[74px]"
          :class="
            darkMode
              ? 'border-neutral-700 bg-neutral-900'
              : 'border-neutral-300 bg-neutral-100'
          "
        >
          <!-- Background Sun Icon -->
          <svg
            class="absolute left-2.5 h-4 w-4 transition-opacity duration-200"
            :class="darkMode ? 'opacity-20 text-neutral-400' : 'opacity-100 text-black'"
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

          <!-- Background Moon Icon -->
          <svg
            class="absolute right-2.5 h-4 w-4 transition-opacity duration-200"
            :class="darkMode ? 'opacity-100 text-white' : 'opacity-20 text-neutral-400'"
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
                ? 'translate-x-[28px] bg-white text-black sm:translate-x-[32px]'
                : 'translate-x-0 bg-black text-white'
            "
          >
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