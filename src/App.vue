<script setup lang="ts">
import LanguageSelector from '@/components/LanguageSelector.vue'
import MobileMenu from '@/components/MobileMenu.vue'
import QRCodeScan from '@/components/QRCodeScan.vue'
import QRCodeCreate from '@/components/QRCodeCreate.vue'
import AppFooter from '@/components/AppFooter.vue'
import { computed, ref, onMounted, onUnmounted } from 'vue'
import { useI18n } from 'vue-i18n'

const { t } = useI18n()

const capturedData = ref<string>('')
const qrCodeScanRef = ref<InstanceType<typeof QRCodeScan> | null>(null)

const lastScrollTop = ref(0)
const isHeaderCollapsed = ref(false)
const isHeaderSticky = ref(false)
const scrollThreshold = 50

const handleScroll = () => {
  const currentScrollTop = document.querySelector('#app')?.scrollTop
  if (currentScrollTop === undefined) return

  isHeaderSticky.value = currentScrollTop > 0

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
  <main class="relative min-h-screen" style="background-color: var(--bg-body)">
    <!-- Desktop header - Dark background -->
    <header
      class="sticky top-0 z-50 hidden h-20 items-center justify-between px-8 md:flex transition-colors duration-200"
      :style="{ backgroundColor: isHeaderSticky ? 'var(--accent-hover)' : 'var(--accent-primary)' }"
    >
      <a href="/" class="flex items-center gap-2 no-underline hover:opacity-100">
        <svg width="28" height="28" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
          <rect x="2" y="2" width="9" height="9" rx="3" stroke="white" stroke-width="2.5" />
          <rect x="13" y="2" width="9" height="9" rx="3" stroke="white" stroke-width="2.5" />
          <rect x="2" y="13" width="9" height="9" rx="3" stroke="white" stroke-width="2.5" />
          <path d="M16 16H20V20" stroke="white" stroke-width="2.5" stroke-linecap="round" />
        </svg>
        <span class="text-xl font-bold tracking-tight text-white" style="letter-spacing: -0.04em">cheQR</span>
      </a>

      <!-- Mode toggle - Segmented control -->
      <div class="header-dark">
        <div class="segmented-control" style="background: rgba(255, 255, 255, 0.1); backdrop-filter: blur(10px)">
          <button
            :class="['segmented-btn', appMode === AppMode.Create ? 'active' : '']"
            @click="setAppMode(AppMode.Create)"
            :disabled="isModeToggleDisabled"
            :aria-label="t('Switch to Create Mode')"
          >
            {{ t('Create') }}
          </button>
          <button
            :class="['segmented-btn', appMode === AppMode.Scan ? 'active' : '']"
            @click="setAppMode(AppMode.Scan)"
            :disabled="isModeToggleDisabled"
            :aria-label="t('Switch to Scan Mode')"
          >
            {{ t('Scan') }}
          </button>
        </div>
      </div>

      <div class="flex items-center gap-2">
        <LanguageSelector :dark-header="true" />
      </div>
    </header>

    <!-- Mobile sticky header -->
    <div
      class="scroll-header-container fixed inset-x-0 top-0 z-50 px-4 pt-3 md:hidden"
      :class="{ 'header-collapsed': isHeaderCollapsed }"
    >
      <div class="flex justify-center">
        <div
          class="flex items-center gap-1 rounded-full p-1 shadow-lg"
          style="background: #09090b"
        >
          <button
            :class="[
              'flex items-center gap-1.5 rounded-full px-4 py-2 text-sm font-semibold outline-none transition-all duration-200',
              appMode === AppMode.Create
                ? 'bg-white text-black shadow-md'
                : 'text-zinc-400 hover:text-white',
              isHeaderCollapsed ? 'py-1.5 text-xs' : 'py-2 text-sm'
            ]"
            @click="setAppMode(AppMode.Create)"
            :disabled="isModeToggleDisabled"
            :aria-label="t('Switch to Create Mode')"
          >
            <span>{{ t('Create') }}</span>
          </button>
          <button
            :class="[
              'flex items-center gap-1.5 rounded-full px-4 py-2 text-sm font-semibold outline-none transition-all duration-200',
              appMode === AppMode.Scan
                ? 'bg-white text-black shadow-md'
                : 'text-zinc-400 hover:text-white',
              isHeaderCollapsed ? 'py-1.5 text-xs' : 'py-2 text-sm'
            ]"
            @click="setAppMode(AppMode.Scan)"
            :disabled="isModeToggleDisabled"
            :aria-label="t('Switch to Scan Mode')"
          >
            <span>{{ t('Scan') }}</span>
          </button>

          <MobileMenu />
        </div>
      </div>
    </div>

    <!-- Main content area -->
    <div
      class="relative z-10 mx-auto min-h-[calc(100vh-80px)] w-full max-w-[1600px] p-6 pt-20 md:pt-6"
      style="background-color: var(--accent-primary)"
    >
      <div v-if="appMode === AppMode.Create">
        <QRCodeCreate :initial-data="capturedData" />
      </div>
      <div v-else class="flex flex-col items-center justify-center py-8">
        <QRCodeScan ref="qrCodeScanRef" @create-qr="useCapturedDataInCreateMode" />
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
