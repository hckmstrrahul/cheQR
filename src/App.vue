<script setup lang="ts">
import LanguageSelector from '@/components/LanguageSelector.vue'
import MobileMenu from '@/components/MobileMenu.vue'
import QRCodeScan from '@/components/QRCodeScan.vue'
import QRCodeCreate from '@/components/QRCodeCreate.vue'
import AppFooter from '@/components/AppFooter.vue'
import useDarkModePreference from '@/utils/useDarkModePreference'
import { computed, ref, onMounted, onUnmounted } from 'vue'
import { useI18n } from 'vue-i18n'

const { t } = useI18n()
const { isDarkMode, isDarkModePreferenceSetBySystem, toggleDarkModePreference } =
  useDarkModePreference()

const capturedData = ref<string>('')
const qrCodeScanRef = ref<InstanceType<typeof QRCodeScan> | null>(null)

const lastScrollTop = ref(0)
const isHeaderCollapsed = ref(false)
const scrollThreshold = 50

const handleScroll = () => {
  const currentScrollTop = document.querySelector('#app')?.scrollTop
  if (!currentScrollTop) return

  if (currentScrollTop > lastScrollTop.value && currentScrollTop > scrollThreshold) {
    isHeaderCollapsed.value = true
  } else if (currentScrollTop < lastScrollTop.value || currentScrollTop < scrollThreshold) {
    isHeaderCollapsed.value = false
  }

  lastScrollTop.value = currentScrollTop
}

onMounted(() => {
  document.querySelector('#app')?.addEventListener('scroll', handleScroll)
})

onUnmounted(() => {
  document.querySelector('#app')?.removeEventListener('scroll', handleScroll)
})

enum AppMode {
  Create = 'create',
  Scan = 'scan'
}

const appMode = ref<AppMode>(AppMode.Create)
const setAppMode = (mode: AppMode) => {
  if (
    appMode.value === AppMode.Scan &&
    mode === AppMode.Create &&
    qrCodeScanRef.value?.capturedData
  ) {
    capturedData.value = qrCodeScanRef.value.capturedData
  }

  appMode.value = mode
}

const useCapturedDataInCreateMode = (data: string) => {
  capturedData.value = data
  appMode.value = AppMode.Create
}

const isModeToggleDisabled = computed(() => {
  return appMode.value === AppMode.Scan && !!qrCodeScanRef.value && !!qrCodeScanRef.value.isLoading
})
</script>

<template>
  <main class="relative min-h-screen">
    <!-- Subtle background pattern -->
    <div class="pointer-events-none fixed inset-0 overflow-hidden">
      <div
        class="absolute -left-40 -top-40 h-[500px] w-[500px] rounded-full bg-violet-500/5 blur-3xl dark:bg-violet-500/10"
      ></div>
      <div
        class="absolute -bottom-40 -right-40 h-[500px] w-[500px] rounded-full bg-indigo-500/5 blur-3xl dark:bg-indigo-500/10"
      ></div>
    </div>

    <!-- Desktop header -->
    <div
      class="relative z-10 hidden md:mx-auto md:mb-2 md:mt-6 md:flex md:w-5/6 md:flex-row md:items-center md:justify-between md:px-4"
    >
      <div class="flex items-center gap-6">
        <a href="/" class="group flex items-center gap-2 no-underline hover:no-underline">
          <div
            class="flex h-10 w-10 items-center justify-center rounded-xl bg-gradient-to-br from-violet-600 to-indigo-600 shadow-lg shadow-violet-500/25 transition-transform duration-200 group-hover:scale-105"
          >
            <svg xmlns="http://www.w3.org/2000/svg" width="22" height="22" viewBox="0 0 24 24">
              <path
                fill="white"
                d="M3 11h8V3H3zm2-6h4v4H5zM3 21h8v-8H3zm2-6h4v4H5zm8-12v8h8V3zm6 6h-4V5h4zm-5 5h2v2h-2zm2 2h2v2h-2zm2 2h2v2h-2zm-4 2h2v2h-2zm2 2h2v2h-2zm2-4h2v2h-2zm2 2h2v2h-2z"
              />
            </svg>
          </div>
          <h1
            class="text-2xl font-bold tracking-tight"
            style="font-family: 'Space Grotesk', sans-serif"
          >
            <span class="gradient-text">cheQR</span>
          </h1>
        </a>

        <!-- Mode toggle -->
        <div
          class="flex items-center gap-1 rounded-2xl border border-gray-200/80 bg-white/60 p-1 shadow-sm backdrop-blur-xl dark:border-white/10 dark:bg-white/5"
        >
          <button
            :class="[
              'flex items-center gap-2 rounded-xl px-4 py-2 text-sm font-medium outline-none transition-all duration-200 focus-visible:ring-2 focus-visible:ring-violet-500/40',
              appMode === AppMode.Create
                ? 'bg-gradient-to-r from-violet-600 to-indigo-600 text-white shadow-md shadow-violet-500/25'
                : 'text-gray-500 hover:text-gray-800 dark:text-gray-400 dark:hover:text-gray-200'
            ]"
            @click="setAppMode(AppMode.Create)"
            :disabled="isModeToggleDisabled"
            :aria-label="t('Switch to Create Mode')"
          >
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24">
              <path
                fill="currentColor"
                d="M3 11h8V3H3zm2-6h4v4H5zM3 21h8v-8H3zm2-6h4v4H5zm8-12v8h8V3zm6 6h-4V5h4zm-6 12h8v-8h-8zm2-6h4v4h-4z"
              />
            </svg>
            <span>{{ t('Create') }}</span>
          </button>
          <button
            :class="[
              'flex items-center gap-2 rounded-xl px-4 py-2 text-sm font-medium outline-none transition-all duration-200 focus-visible:ring-2 focus-visible:ring-violet-500/40',
              appMode === AppMode.Scan
                ? 'bg-gradient-to-r from-violet-600 to-indigo-600 text-white shadow-md shadow-violet-500/25'
                : 'text-gray-500 hover:text-gray-800 dark:text-gray-400 dark:hover:text-gray-200'
            ]"
            @click="setAppMode(AppMode.Scan)"
            :disabled="isModeToggleDisabled"
            :aria-label="t('Switch to Scan Mode')"
          >
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24">
              <path
                fill="currentColor"
                d="M4 4h4V2H2v6h2zm16-2h-4v2h4v4h2V2zM4 16H2v6h6v-2H4zm16 4h-4v2h6v-6h-2zM12 9a3 3 0 1 0 0 6a3 3 0 0 0 0-6"
              />
            </svg>
            <span>{{ t('Scan') }}</span>
          </button>
        </div>
      </div>

      <div class="flex items-center gap-1">
        <button
          class="icon-button flex h-9 w-9 items-center justify-center rounded-xl"
          @click="toggleDarkModePreference"
          :aria-label="t('Toggle dark mode')"
        >
          <span v-if="isDarkModePreferenceSetBySystem">
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24">
              <g fill="currentColor" opacity="0.7">
                <path d="M12 16a4 4 0 0 0 0-8z" />
                <path
                  fill-rule="evenodd"
                  d="M12 2C6.477 2 2 6.477 2 12s4.477 10 10 10s10-4.477 10-10S17.523 2 12 2m0 2v4a4 4 0 1 0 0 8v4a8 8 0 1 0 0-16"
                  clip-rule="evenodd"
                />
              </g>
            </svg>
          </span>
          <span v-else-if="isDarkMode">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
              stroke-width="2"
              width="20"
              height="20"
              class="opacity-70"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z"
              />
            </svg>
          </span>
          <span v-else>
            <svg
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke-width="2"
              width="20"
              height="20"
              class="opacity-70"
            >
              <path
                fill="currentColor"
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z"
              />
            </svg>
          </span>
        </button>
        <LanguageSelector />
      </div>
    </div>

    <!-- Mobile sticky header -->
    <div
      class="scroll-header-container fixed inset-x-0 top-0 z-50 px-4 pt-3 md:hidden"
      :class="{ 'header-collapsed': isHeaderCollapsed }"
    >
      <div class="flex justify-center">
        <div
          class="flex items-center gap-1 rounded-2xl border border-gray-200/60 bg-white/70 p-1 shadow-lg backdrop-blur-xl dark:border-white/10 dark:bg-gray-900/70"
        >
          <button
            :class="[
              'flex items-center gap-1.5 rounded-xl px-3 py-1.5 text-sm font-medium outline-none transition-all duration-200 focus-visible:ring-2 focus-visible:ring-violet-500/40',
              appMode === AppMode.Create
                ? 'bg-gradient-to-r from-violet-600 to-indigo-600 text-white shadow-md shadow-violet-500/25'
                : 'text-gray-500 hover:text-gray-800 dark:text-gray-400 dark:hover:text-gray-200',
              isHeaderCollapsed ? 'py-1 text-xs' : 'py-1.5 text-sm'
            ]"
            @click="setAppMode(AppMode.Create)"
            :disabled="isModeToggleDisabled"
            :aria-label="t('Switch to Create Mode')"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              :width="isHeaderCollapsed ? 12 : 14"
              :height="isHeaderCollapsed ? 12 : 14"
              viewBox="0 0 24 24"
            >
              <path
                fill="currentColor"
                d="M3 11h8V3H3zm2-6h4v4H5zM3 21h8v-8H3zm2-6h4v4H5zm8-12v8h8V3zm6 6h-4V5h4zm-6 12h8v-8h-8zm2-6h4v4h-4z"
              />
            </svg>
            <span>{{ t('Create') }}</span>
          </button>
          <button
            :class="[
              'flex items-center gap-1.5 rounded-xl px-3 py-1.5 text-sm font-medium outline-none transition-all duration-200 focus-visible:ring-2 focus-visible:ring-violet-500/40',
              appMode === AppMode.Scan
                ? 'bg-gradient-to-r from-violet-600 to-indigo-600 text-white shadow-md shadow-violet-500/25'
                : 'text-gray-500 hover:text-gray-800 dark:text-gray-400 dark:hover:text-gray-200',
              isHeaderCollapsed ? 'py-1 text-xs' : 'py-1.5 text-sm'
            ]"
            @click="setAppMode(AppMode.Scan)"
            :disabled="isModeToggleDisabled"
            :aria-label="t('Switch to Scan Mode')"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              :width="isHeaderCollapsed ? 12 : 14"
              :height="isHeaderCollapsed ? 12 : 14"
              viewBox="0 0 24 24"
            >
              <path
                fill="currentColor"
                d="M4 4h4V2H2v6h2zm16-2h-4v2h4v4h2V2zM4 16H2v6h6v-2H4zm16 4h-4v2h6v-6h-2zM12 9a3 3 0 1 0 0 6a3 3 0 0 0 0-6"
              />
            </svg>
            <span>{{ t('Scan') }}</span>
          </button>

          <MobileMenu
            :isDarkMode="isDarkMode"
            :isDarkModePreferenceSetBySystem="isDarkModePreferenceSetBySystem"
            @toggle-dark-mode="toggleDarkModePreference"
          />
        </div>
      </div>
    </div>

    <div
      class="relative z-10 grid min-h-screen place-items-center items-start bg-transparent p-6 pt-16 md:px-6 md:pt-4"
    >
      <div class="w-full lg:w-5/6">
        <div v-if="appMode === AppMode.Create">
          <QRCodeCreate :initial-data="capturedData" />
        </div>
        <div v-else class="flex flex-col items-center justify-center py-8">
          <QRCodeScan ref="qrCodeScanRef" @create-qr="useCapturedDataInCreateMode" />
        </div>
      </div>
    </div>
    <AppFooter />
  </main>
</template>

<style lang="postcss" scoped>
.scroll-header-container {
  transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.header-collapsed {
  transform: translateY(-40%);
}

.header-collapsed button {
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}
</style>
