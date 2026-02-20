<script setup lang="ts">
import {
  IS_PASTE_IMAGE_FROM_CLIPBOARD_SUPPORTED,
  KEY_COMBINATION_PASTE,
  getFileFromClipboardItems,
  getFileFromDataTransferItemList
} from '@/utils/clipboard'
import { Html5Qrcode } from 'html5-qrcode'
import { computed, onMounted, ref } from 'vue'
import { useI18n } from 'vue-i18n'
import QRCodeCameraScanner from './QRCodeCameraScanner.vue'

defineEmits<{
  'create-qr': [data: string]
}>()

const { t } = useI18n()

// #region Core QR Code Data
const capturedData = ref<string>('')
const errorMessage = ref<string | null>(null)
// #endregion Core QR Code Data

// #region QR Code Type Detection
const qrCodeType = computed(() => {
  const data = capturedData.value

  // URL detection (more comprehensive than just http)
  if (
    /^(https?:\/\/)?[a-zA-Z0-9][a-zA-Z0-9-]*[a-zA-Z0-9](\.[a-zA-Z]{2,})+([/?#][^\s]*)?$/i.test(data)
  ) {
    return 'url'
  }

  // Email detection
  if (
    /^mailto:(.+)$/i.test(data) ||
    /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/.test(data)
  ) {
    return 'email'
  }

  // Phone number detection
  if (/^tel:(.+)$/i.test(data) || /^[+]?[(]?[0-9]{1,4}[)]?[-\s./0-9]*$/.test(data)) {
    return 'tel'
  }

  // SMS detection
  if (/^sms:(.+)$/i.test(data)) {
    return 'sms'
  }

  // WiFi detection
  if (/^WIFI:(.+)$/i.test(data)) {
    return 'wifi'
  }

  // vCard detection
  if (/^BEGIN:VCARD[\s\S]*END:VCARD$/i.test(data)) {
    return 'vcard'
  }

  // Calendar event detection
  if (/^BEGIN:VEVENT[\s\S]*END:VEVENT$/i.test(data)) {
    return 'calendar'
  }

  // Geo location detection
  if (/^geo:(.+)$/i.test(data)) {
    return 'geo'
  }

  // Default to text
  return 'text'
})

const formattedData = computed(() => {
  const data = capturedData.value
  const type = qrCodeType.value
  const hasProtocol = data.startsWith('http://') || data.startsWith('https://')

  switch (type) {
    case 'url':
      return hasProtocol ? data : `https://${data}`
    case 'email':
      return data.startsWith('mailto:') ? data : `mailto:${data}`
    case 'tel':
      return data.startsWith('tel:') ? data : `tel:${data}`
    case 'sms':
      return data.startsWith('sms:') ? data : `sms:${data}`
    case 'wifi':
      // Return as is for display purposes
      return data
    case 'vcard':
    case 'calendar':
    case 'geo':
      return data
    default:
      return data
  }
})

const isActionable = computed(() => {
  return ['url', 'email', 'tel', 'sms', 'geo'].includes(qrCodeType.value)
})
// #endregion QR Code Type Detection

// #region UI Display Properties
const qrCodeTypeIcon = computed(() => {
  switch (qrCodeType.value) {
    case 'url':
      return `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24"><path fill="currentColor" d="M17 7h-4v2h4c1.65 0 3 1.35 3 3s-1.35 3-3 3h-4v2h4c2.76 0 5-2.24 5-5s-2.24-5-5-5m-6 8H7c-1.65 0-3-1.35-3-3s1.35-3 3-3h4V7H7c-2.76 0-5 2.24-5 5s2.24 5 5 5h4zm-3-4h8v2H8z"/></svg>`
    case 'email':
      return `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24"><path fill="currentColor" d="M20 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V6c0-1.1-.9-2-2-2m0 4l-8 5l-8-5V6l8 5l8-5z"/></svg>`
    case 'tel':
      return `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24"><path fill="currentColor" d="M6.62 10.79c1.44 2.83 3.76 5.14 6.59 6.59l2.2-2.2c.27-.27.67-.36 1.02-.24c1.12.37 2.33.57 3.57.57c.55 0 1 .45 1 1V20c0 .55-.45 1-1 1c-9.39 0-17-7.61-17-17c0-.55.45-1 1-1h3.5c.55 0 1 .45 1 1c0 1.25.2 2.45.57 3.57c.11.35.03.74-.25 1.02z"/></svg>`
    case 'sms':
      return `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24"><path fill="currentColor" d="M20 2H4c-1.1 0-1.99.9-1.99 2L2 22l4-4h14c1.1 0 2-.9 2-2V4c0-1.1-.9-2-2-2M9 11H7V9h2zm4 0h-2V9h2zm4 0h-2V9h2z"/></svg>`
    case 'wifi':
      return `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24"><path fill="currentColor" d="M1 9l2 2c4.97-4.97 13.03-4.97 18 0l2-2C16.93 2.93 7.08 2.93 1 9m8 8l3 3l3-3a4.237 4.237 0 0 0-6 0m-4-4l2 2a7.074 7.074 0 0 1 10 0l2-2C15.14 9.14 8.87 9.14 5 13z"/></svg>`
    case 'vcard':
      return `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24"><path fill="currentColor" d="M20 4H4c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V6c0-1.1-.9-2-2-2m-7 2h2.5v12H13zm-2 12H8.5V6H11zM4 6h2.5v12H4zm16 12h-2.5V6H20z"/></svg>`
    case 'calendar':
      return `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24"><path fill="currentColor" d="M19 3h-1V1h-2v2H8V1H6v2H5c-1.11 0-1.99.9-1.99 2L3 19a2 2 0 0 0 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2m0 16H5V8h14zm-7-5h5v5h-5z"/></svg>`
    case 'geo':
      return `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24"><path fill="currentColor" d="M12 2C8.13 2 5 5.13 5 9c0 5.25 7 13 7 13s7-7.75 7-13c0-3.87-3.13-7-7-7m0 9.5a2.5 2.5 0 0 1 0-5a2.5 2.5 0 0 1 0 5z"/></svg>`
    default:
      return `<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24"><path fill="currentColor" d="M14.06 9.02l.92.92L5.92 19H5v-.92l9.06-9.06M17.66 3c-.25 0-.51.1-.7.29l-1.83 1.83l3.75 3.75l1.83-1.83a.996.996 0 0 0 0-1.41l-2.34-2.34c-.2-.2-.45-.29-.71-.29m-3.6 3.19L3 17.25V21h3.75L17.81 9.94l-3.75-3.75z"/></svg>`
  }
})

const typeLabel = computed(() => {
  switch (qrCodeType.value) {
    case 'url':
      return t('URL')
    case 'email':
      return t('Email')
    case 'tel':
      return t('Phone Number')
    case 'sms':
      return t('SMS')
    case 'wifi':
      return t('WiFi')
    case 'vcard':
      return t('Contact Card')
    case 'calendar':
      return t('Calendar Event')
    case 'geo':
      return t('Location')
    default:
      return t('Text')
  }
})

// #endregion UI Display Properties

// #region User Actions
const copySuccess = ref(false)
const showCameraScanner = ref(false)

const copyToClipboard = async () => {
  if (!capturedData.value) return

  try {
    await navigator.clipboard.writeText(capturedData.value)
    copySuccess.value = true

    // Clear the success message after 3 seconds
    setTimeout(() => {
      copySuccess.value = false
    }, 3000)
  } catch (err) {
    errorMessage.value = t('Failed to copy to clipboard')
    console.error(err)
  }
}

/**
 * Handle both an event of paste from clipboard and a button press 'Paste from clipboard'
 * @param event : ClipboardEvent | null
 */
const pasteFromClipboard = async (event: ClipboardEvent | null) => {
  try {
    let file: File | null = null

    if (event && event.clipboardData && event.clipboardData.items) {
      // user action Ctrl+V
      file = await getFileFromDataTransferItemList(event.clipboardData.items)
    } else {
      // button press
      file = await getFileFromClipboardItems(await navigator.clipboard.read())
    }

    if (file == null) {
      errorMessage.value = t('No image found in clipboard.')
      return
    }

    scanFile(file)
  } catch (err: any) {
    console.error('Clipboard paste failed', err)
  }
}

onMounted(() => {
  window.addEventListener('paste', pasteFromClipboard)
})

const onQRDetected = (data: string) => {
  capturedData.value = data
  showCameraScanner.value = false
}

const onCameraScannerCancel = () => {
  showCameraScanner.value = false
}

const startCameraScanning = () => {
  errorMessage.value = null
  showCameraScanner.value = true
}

const resetCapture = () => {
  capturedData.value = ''
  errorMessage.value = null
  copySuccess.value = false
  showCameraScanner.value = false
}
// #endregion User Actions

// #region File Handling
const fileInput = ref<HTMLInputElement | null>(null)
const isLoading = ref(false)
const isDraggingOver = ref(false)

const catchScanFileError = async (err: Error, file: File) => {
  console.warn('Html5Qrcode failed, will try fallback to nimiq/qr-scanner:', err)

  const QrScanner = (await import('qr-scanner')).default

  // Fallback to nimiq/qr-scanner lib
  try {
    const result = await QrScanner.scanImage(file, { returnDetailedScanResult: true })
    capturedData.value = result.data
  } catch (err) {
    console.error('Fallback to nimiq/qr-scanner failed:', err)
    errorMessage.value = t('No QR code found in the image.')
  } finally {
    isLoading.value = false
  }
}

const scanFile = (file: File) => {
  isLoading.value = true
  errorMessage.value = null

  const html5QrCode = new Html5Qrcode('file-qr-reader')
  html5QrCode
    .scanFile(file, false)
    .then((decodedText) => {
      capturedData.value = decodedText
      isLoading.value = false
    })
    .catch((err) => catchScanFileError(err, file))
}

const handleFileUpload = (event: Event) => {
  let file: File | null = null

  // Handle drag and drop event
  if (event.type === 'drop') {
    const dt = (event as DragEvent).dataTransfer
    if (dt?.files && dt.files.length > 0) {
      file = dt.files[0]
    }
  }
  // Handle file input change event
  else {
    const target = event.target as HTMLInputElement
    if (target.files && target.files.length > 0) {
      file = target.files[0]
    }
  }

  if (!file) return

  scanFile(file)
}

const handleDragOver = (event: DragEvent) => {
  event.preventDefault()
  isDraggingOver.value = true
}

const handleDragLeave = () => {
  isDraggingOver.value = false
}

const handleDrop = (event: DragEvent) => {
  event.preventDefault()
  isDraggingOver.value = false
  handleFileUpload(event)
}
// #endregion File Handling

defineExpose({
  capturedData,
  isLoading,
  resetCapture,
  copyToClipboard
})
</script>

<template>
  <div class="relative mx-auto w-full max-w-[600px]">
    <!-- Success Result Card -->
    <div v-if="capturedData" class="surface-card p-8 text-center" style="border-radius: var(--radius-lg)">
      <!-- Success Icon -->
      <div
        class="mx-auto mb-6 flex h-20 w-20 items-center justify-center rounded-full"
        style="background: var(--bg-input)"
      >
        <svg width="32" height="32" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path d="M5 13l4 4L19 7" stroke-width="3" stroke-linecap="round" stroke-linejoin="round" />
        </svg>
      </div>

      <h2 class="mb-2 text-2xl font-semibold">{{ t('Successfully Scanned') }}</h2>
      <p class="mb-6 text-sm" style="color: var(--text-muted)">{{ t('Extracted content from the QR code') }}</p>

      <!-- QR Code Type Badge -->
      <div class="mb-4 flex items-center justify-center">
        <span
          class="inline-flex items-center gap-1 rounded-full px-3 py-1 text-sm"
          style="background: var(--bg-input)"
          v-html="qrCodeTypeIcon + ' ' + typeLabel"
        ></span>
      </div>

      <!-- Data Display -->
      <div
        class="mb-8 break-all rounded-xl p-4 text-left font-mono text-sm"
        style="background: var(--bg-input)"
      >
        <component
          :is="isActionable ? 'a' : 'span'"
          :href="isActionable ? formattedData : undefined"
          :target="qrCodeType === 'url' ? '_blank' : undefined"
          class="block"
        >
          {{ capturedData }}
        </component>
      </div>

      <!-- Action Buttons -->
      <div class="mx-auto grid max-w-md grid-cols-2 gap-3">
        <button
          v-if="isActionable"
          class="btn btn-primary col-span-2"
          @click="window.open(formattedData, '_blank')"
        >
          <svg width="18" height="18" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path d="M10 6H6a2 2 0 00-2 2v10a2 2 0 002 2h10a2 2 0 002-2v-4M14 4h6m0 0v6m0-6L10 14" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
          </svg>
          {{ qrCodeType === 'url' ? t('Open URL') : t('Open') }}
        </button>
        <button class="btn btn-secondary" @click="copyToClipboard">
          <svg width="18" height="18" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path d="M8 5H6a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2v-1M8 5a2 2 0 002 2h2a2 2 0 002-2M8 5a2 2 0 012-2h2a2 2 0 012 2" stroke-width="2" stroke-linecap="round" />
          </svg>
          {{ t('Copy') }}
        </button>
        <button class="btn btn-secondary" @click="$emit('create-qr', capturedData)">
          <svg width="18" height="18" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path d="M3 11h8V3H3zm2-6h4v4H5zM3 21h8v-8H3zm2-6h4v4H5zm8-12v8h8V3zm6 6h-4V5h4z" fill="currentColor" stroke="none" />
          </svg>
          {{ t('Create QR') }}
        </button>
      </div>

      <!-- Scan Another -->
      <button
        class="mt-8 text-sm transition-colors hover:opacity-70"
        style="color: var(--text-muted)"
        @click="resetCapture"
      >
        {{ t('Scan Another Code') }}
      </button>
    </div>

    <!-- Camera Scanner -->
    <div v-else-if="showCameraScanner" class="mb-4 w-full">
      <QRCodeCameraScanner @qr-detected="onQRDetected" @cancel="onCameraScannerCancel" />
    </div>

    <!-- Initial Scan Controls -->
    <div v-else class="capture-controls">
      <!-- Loading State -->
      <div v-if="isLoading" class="mb-4 flex flex-col items-center justify-center">
        <div
          class="mb-2 size-10 animate-spin rounded-full border-4 border-solid"
          style="border-color: var(--border-light); border-top-color: var(--accent-primary)"
        ></div>
        <p>{{ t('Processing...') }}</p>
      </div>

      <!-- Hidden div for file QR reader -->
      <div id="file-qr-reader" class="hidden"></div>

      <div class="flex flex-col items-center gap-4" v-if="!isLoading">
        <div class="surface-card w-full p-8 text-center" style="border-radius: var(--radius-lg)">
          <h3 class="mb-6 text-xl font-semibold">{{ t('Scan a QR Code') }}</h3>

          <!-- Upload Area -->
          <button
            :class="[
              'flex w-full cursor-pointer items-center justify-center rounded-2xl border-2 border-dashed p-8 text-center transition-all',
              isDraggingOver
                ? 'border-black bg-zinc-100 dark:border-white dark:bg-zinc-800'
                : 'border-zinc-300 hover:border-zinc-400 hover:bg-zinc-50 dark:border-zinc-700 dark:hover:border-zinc-600 dark:hover:bg-zinc-800/50'
            ]"
            @click="fileInput?.click()"
            @keyup.enter="fileInput?.click()"
            @keyup.space="fileInput?.click()"
            @dragover="handleDragOver"
            @dragleave="handleDragLeave"
            @drop="handleDrop"
          >
            <div class="flex flex-col items-center gap-3">
              <svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 24 24" style="color: var(--text-muted)">
                <path
                  fill="currentColor"
                  d="M11 16V7.85l-2.6 2.6L7 9l5-5l5 5l-1.4 1.45l-2.6-2.6V16h-2Zm-5 4q-.825 0-1.413-.588T4 18v-3h2v3h12v-3h2v3q0 .825-.588 1.413T18 20H6Z"
                />
              </svg>
              <p class="font-medium">{{ t('Upload QR Code Image') }}</p>
              <p class="text-sm" style="color: var(--text-muted)">{{ t('or drag and drop an image here') }}</p>
            </div>
          </button>
          <input
            ref="fileInput"
            type="file"
            accept="image/*"
            class="hidden"
            @change="handleFileUpload"
          />

          <!-- Error message -->
          <p v-if="errorMessage" class="mt-4 text-red-500">
            {{ errorMessage }}
          </p>

          <!-- Helpful tip -->
          <p class="mt-3 text-sm" style="color: var(--text-muted)">
            {{ t('Tip: For best results, use a clear image with good lighting.') }}
          </p>

          <div class="mt-6 flex flex-col items-center gap-3">
            <p class="text-sm" style="color: var(--text-muted)">{{ t('or') }}</p>

            <div class="flex flex-wrap items-center justify-center gap-3">
              <!-- Camera option -->
              <button
                class="btn btn-secondary"
                @click="startCameraScanning"
                type="button"
              >
                <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24">
                  <path
                    fill="currentColor"
                    d="M12 9a3 3 0 1 0 0 6a3 3 0 0 0 0-6m0 8a5 5 0 1 1 0-10a5 5 0 0 1 0 10m0-12a1 1 0 0 1 1 1a1 1 0 0 1-1 1a1 1 0 0 1-1-1a1 1 0 0 1 1-1m4.5 1.5a1.5 1.5 0 0 1 1.5 1.5a1.5 1.5 0 0 1-1.5 1.5a1.5 1.5 0 0 1-1.5-1.5a1.5 1.5 0 0 1 1.5-1.5M20 4h-3.17l-1.24-1.35A1.99 1.99 0 0 0 14.12 2H9.88c-.56 0-1.1.24-1.48.65L7.17 4H4a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V6a2 2 0 0 0-2-2"
                  />
                </svg>
                {{ t('Scan with Camera') }}
              </button>

              <!-- Paste from clipboard option -->
              <button
                class="btn btn-secondary"
                @click="pasteFromClipboard"
                type="button"
                v-if="IS_PASTE_IMAGE_FROM_CLIPBOARD_SUPPORTED"
              >
                <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24">
                  <path
                    fill="currentColor"
                    d="M19 2h-4.18C14.4.84 13.3 0 12 0S9.6.84 9.18 2H5c-1.1 0-2 .9-2 2v16c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V4c0-1.1-.9-2-2-2m-7 0c.55 0 1 .45 1 1s-.45 1-1 1s-1-.45-1-1s.45-1 1-1m7 18H5V4h2v3h10V4h2z"
                  />
                </svg>
                {{ t('Paste') }}
                <span v-html="KEY_COMBINATION_PASTE"></span>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
