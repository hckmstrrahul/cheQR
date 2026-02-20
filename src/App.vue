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
  <main class="relative min-h-screen" style="background-color: var(--accent-primary)">
    <!-- Desktop header - Dark background -->
    <header
      class="sticky top-0 z-50 hidden h-20 items-center justify-between px-8 md:flex transition-colors duration-200"
      :style="{ backgroundColor: isHeaderSticky ? 'var(--accent-hover)' : 'var(--accent-primary)' }"
    >
      <a href="/" class="flex items-center gap-2 no-underline hover:opacity-100">
        <svg width="28" height="28" viewBox="0 0 1024 1024" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path fill-rule="evenodd" clip-rule="evenodd" d="M979.6 120.876L891.5 120.892L891.505 277.21C920.785 277.41 950.36 276.922 979.575 277.288C979.715 303.189 979.88 329.734 979.59 355.613C950.575 355.616 920.39 355.203 891.495 355.669L891.521 434.089L979.565 434.031L979.62 511.995L894.495 512.021L894.445 590.985C922.38 590.49 951.57 590.905 979.57 590.92C979.925 617.005 979.585 643.855 979.55 669.98L894.485 669.955V748.67L979.63 748.585L979.555 985.905L894.325 985.935L894.435 748.775C876.46 748.18 854.159 748.8 835.935 748.805L716.49 748.83L716.515 985.835C684.855 986.235 652.59 985.93 620.885 985.94L620.785 355.605L469.422 355.688L469.352 511.98L383.998 512.01L384.039 42.54L469.421 42.5449L469.424 198.737C469.422 224.71 469.113 251.273 469.443 277.188L620.84 277.249L620.81 69.7402C620.81 61.2773 620.535 50.8888 620.885 42.5557L710.81 42.5371L710.725 461.471C710.72 478.058 710.45 495.464 710.73 511.985C740.84 511.985 771.45 512.285 801.515 511.955V42.5371L979.65 42.5361L979.6 120.876ZM716.5 591.03L716.521 669.725C768.06 670.49 819.89 669.215 871.455 669.775C878.99 669.86 886.83 669.895 894.35 669.705C894.34 644.05 893.865 616.54 894.445 590.985L716.5 591.03Z" fill="white"/>
          <path d="M42.7453 590.95C70.8535 590.62 99.9065 590.835 128.035 590.935C169.958 591.435 212.745 591.02 254.753 591.01L384.002 590.965C412.097 590.835 441.37 590.515 469.389 590.96L469.388 827.7C441.53 828.19 411.944 828.02 384.009 827.84L384.007 906.555C357.047 907.645 326.09 906.585 298.609 906.83L298.61 985.885C279.533 986.215 259.607 985.925 240.477 985.93L128.054 985.95L128.08 906.545C100.222 907.18 70.753 906.735 42.7708 906.69L42.6715 721.58C42.6594 678.505 42.0199 633.91 42.7453 590.95Z" fill="var(--accent-primary)"/>
          <path d="M384.002 590.965C412.097 590.835 441.37 590.515 469.389 590.96L469.388 827.7C441.53 828.19 411.944 828.02 384.009 827.84L384.007 906.555C357.047 907.645 326.09 906.585 298.609 906.83L298.61 985.885C279.533 986.215 259.607 985.925 240.477 985.93L128.054 985.95L128.08 906.545L298.49 906.5L298.576 827.67C325.961 826.945 356.379 827.61 384.006 827.615L384.002 590.965Z" fill="white"/>
          <path d="M42.7453 590.95C70.8535 590.62 99.9065 590.835 128.035 590.935L128.08 906.545C100.222 907.18 70.753 906.735 42.7708 906.69L42.6715 721.58C42.6594 678.505 42.0199 633.91 42.7453 590.95Z" fill="white"/>
          <path d="M213.233 748.685L298.633 748.6L298.576 827.67C271.446 828.525 240.518 827.85 213.156 827.84C213.144 801.855 212.706 774.56 213.233 748.685Z" fill="white"/>
          <path d="M42.6441 42.5417L298.69 42.5461L298.723 277.344L213.185 277.338L213.207 120.898L128.114 120.919C127.412 155.706 128.077 193.027 128.077 227.943L128.111 434.129L213.171 434.064L213.253 355.585C241.542 355.271 270.352 355.547 298.684 355.54C299.622 407.027 298.653 460.286 298.868 511.99L383.998 512.01L384.002 590.965L254.753 591.01C212.745 591.02 169.958 591.435 128.035 590.935L128.03 512.03L42.5951 512.015L42.6441 42.5417Z" fill="white"/>
          <path d="M384.007 906.555L469.402 906.53L469.368 985.875L384.003 985.955L384.007 906.555Z" fill="white"/>
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
