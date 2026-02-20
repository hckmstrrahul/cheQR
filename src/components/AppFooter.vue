<script setup lang="ts">
import { useI18n } from 'vue-i18n'

const { t } = useI18n()
const hideCredits = ['1', 'true'].includes((import.meta.env.VITE_HIDE_CREDITS ?? '').toLowerCase())
const currentYear = new Date().getFullYear()
</script>

<template>
  <footer
    v-if="!hideCredits"
    class="hidden h-10 items-center justify-center gap-4 text-xs md:flex"
    style="background-color: var(--accent-primary); color: rgba(255, 255, 255, 0.4)"
  >
    <span class="font-medium">© Feb {{ currentYear }} cheQR</span>
    <span>•</span>
    <span>
      Built by
      <a
        href="https://x.com/hckmstrrahul"
        target="_blank"
        class="transition-colors hover:text-white"
        style="color: rgba(255, 255, 255, 0.6)"
      >
        @hckmstrrahul
      </a>
    </span>
    <span>•</span>
    <a
      href="https://github.com/hckmstrrahul/cheQR"
      target="_blank"
      class="transition-colors hover:text-white"
      style="color: rgba(255, 255, 255, 0.6)"
    >
      {{ t('Free & open source') }}
    </a>
    <span>•</span>
    <span style="color: rgba(255, 255, 255, 0.4)">
      {{ t('Based on') }}
      <a
        href="https://github.com/lyqht/mini-qr"
        target="_blank"
        class="transition-colors hover:text-white"
        style="color: rgba(255, 255, 255, 0.6)"
      >
        Mini QR
      </a>
      {{ t('by') }}
      <a
        href="https://github.com/lyqht"
        target="_blank"
        class="transition-colors hover:text-white"
        style="color: rgba(255, 255, 255, 0.6)"
      >
        Estee Tey
      </a>
    </span>
  </footer>
</template>
