<script setup lang="ts">
import { ref } from 'vue'
import { useI18n } from 'vue-i18n'

const { t } = useI18n()
const hideCredits = ['1', 'true'].includes((import.meta.env.VITE_HIDE_CREDITS ?? '').toLowerCase())
</script>

<template>
  <footer
    v-if="!hideCredits"
    class="fixed inset-x-0 bottom-0 hidden p-3 text-xs text-gray-400 dark:text-gray-500 md:flex md:justify-center"
  >
    <div class="flex items-center gap-2">
      <span class="font-medium" style="font-family: 'Space Grotesk', sans-serif">cheQR</span>
      <span class="text-gray-300 dark:text-gray-600">·</span>
      <span>{{ t('Free & open source') }}</span>
      <span class="text-gray-300 dark:text-gray-600">·</span>
      <span>
        {{ t('Based on') }}
        <a
          href="https://github.com/lyqht/mini-qr"
          target="_blank"
          class="text-gray-500 hover:text-violet-500 dark:text-gray-400 dark:hover:text-violet-400"
          >Mini QR</a
        >
        {{ t('by') }}
        <a
          href="https://github.com/lyqht"
          target="_blank"
          class="text-gray-500 hover:text-violet-500 dark:text-gray-400 dark:hover:text-violet-400"
          >Estee Tey</a
        >
      </span>
    </div>
  </footer>
</template>

<style scoped>
footer {
  background: rgba(255, 255, 255, 0.6);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
}

.dark footer {
  background: rgba(10, 10, 15, 0.6);
}
</style>
