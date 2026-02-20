<script setup lang="ts">
import { Combobox } from '@/components/ui/Combobox'
import { computed, ref, onMounted, watch } from 'vue'
import { useI18n } from 'vue-i18n'
import { sortedLocales } from '@/utils/language'

defineProps<{
  darkHeader?: boolean
}>()

const STORAGE_KEY = 'preferred-language'

const isLocaleSelectOpen = ref(false)
const { t, locale } = useI18n()
const locales = computed(() =>
  sortedLocales.map((loc) => ({
    value: loc,
    label: t(loc)
  }))
)

const getFontFamilyForLocale = (localeValue: string): string => {
  const baseFallback = "-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif"
  
  // English locales use Google Sans
  if (localeValue === 'en' || localeValue === 'enUS' || localeValue === 'enGB') {
    return `'Google Sans', ${baseFallback}`
  }
  
  // Chinese (Simplified)
  if (localeValue === 'zhHANS') {
    return `'Noto Sans SC', 'Noto Sans', ${baseFallback}`
  }
  
  // Chinese (Traditional)
  if (localeValue === 'zh') {
    return `'Noto Sans TC', 'Noto Sans', ${baseFallback}`
  }
  
  // Japanese
  if (localeValue === 'ja') {
    return `'Noto Sans JP', 'Noto Sans', ${baseFallback}`
  }
  
  // Korean
  if (localeValue === 'ko') {
    return `'Noto Sans KR', 'Noto Sans', ${baseFallback}`
  }
  
  // Arabic, Persian (Farsi)
  if (localeValue === 'ar' || localeValue === 'fa') {
    return `'Noto Sans Arabic', 'Noto Sans', ${baseFallback}`
  }
  
  // Hindi, Marathi and other Devanagari script languages
  if (localeValue === 'hi') {
    return `'Noto Sans Devanagari', 'Noto Sans', ${baseFallback}`
  }
  
  // Bengali
  if (localeValue === 'bn') {
    return `'Noto Sans Bengali', 'Noto Sans', ${baseFallback}`
  }
  
  // Thai
  if (localeValue === 'th') {
    return `'Noto Sans Thai', 'Noto Sans', ${baseFallback}`
  }
  
  // All other languages use Noto Sans
  return `'Noto Sans', ${baseFallback}`
}

const updateFontFamily = (localeValue: string) => {
  const fontFamily = getFontFamilyForLocale(localeValue)
  document.documentElement.style.setProperty('--font-family', fontFamily)
  document.body.style.fontFamily = fontFamily
}

watch(locale, (newLocale) => {
  if (newLocale) {
    localStorage.setItem(STORAGE_KEY, newLocale)
    updateFontFamily(newLocale)
  }
})

onMounted(() => {
  const savedLocale = localStorage.getItem(STORAGE_KEY)

  if (savedLocale && sortedLocales.includes(savedLocale)) {
    locale.value = savedLocale
    updateFontFamily(savedLocale)
    return
  }

  // Default to English
  locale.value = 'en'
  updateFontFamily('en')
})
</script>

<template>
  <Combobox
    :items="locales"
    v-model:value="locale"
    v-model:open="isLocaleSelectOpen"
    :button-label="t('Select language')"
    :dark-header="darkHeader"
  >
    <template #button-icon>
      <svg
        xmlns="http://www.w3.org/2000/svg"
        class="icon -ms-1.5 size-5 shrink-0"
        width="20"
        height="20"
        viewBox="0 0 24 24"
      >
        <g
          fill="none"
          :stroke="darkHeader ? 'rgba(255,255,255,0.7)' : 'currentColor'"
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
        >
          <path d="M4 5h7M7 4c0 4.846 0 7 .5 8" />
          <path
            d="M10 8.5c0 2.286-2 4.5-3.5 4.5S4 11.865 4 11c0-2 1-3 3-3s5 .57 5 2.857c0 1.524-.667 2.571-2 3.143m2 6l4-9l4 9m-.9-2h-6.2"
          />
        </g>
      </svg>
    </template>
  </Combobox>
</template>
