<script setup lang="ts">
import BatchExportFieldsGuide from '@/components/BatchExportFieldsGuide.vue'
import CopyImageModal from '@/components/CopyImageModal.vue'
import QRCodeFrame from '@/components/QRCodeFrame.vue'
import StyledQRCode from '@/components/StyledQRCode.vue'
import {
  Accordion,
  AccordionContent,
  AccordionItem,
  AccordionTrigger
} from '@/components/ui/accordion'
import { Combobox } from '@/components/ui/Combobox'
import {
  Drawer,
  DrawerContent,
  DrawerHeader,
  DrawerTitle,
  DrawerTrigger
} from '@/components/ui/drawer'
import VCardPreview from '@/components/VCardPreview.vue'
import { IS_COPY_IMAGE_TO_CLIPBOARD_SUPPORTED } from '@/utils/clipboard'
import { createRandomColor, getRandomItemInArray } from '@/utils/color'
import {
  copyImageToClipboard,
  downloadJpgElement,
  downloadPngElement,
  downloadSvgElement,
  getJpgElement,
  getPngElement,
  getSvgString
} from '@/utils/convertToImage'
import { parseCSV, validateCSVData, type CSVParsingResult } from '@/utils/csv'
import {
  generateEmailData,
  generateEventData,
  generateLocationData,
  generatePhoneData,
  generateSmsData,
  generateVCardData,
  generateWifiData
} from '@/utils/dataEncoding'
import { generateBatchExportFilename, processCsvDataForBatch } from '@/utils/csvBatchProcessing'
import { getNumericCSSValue } from '@/utils/formatting'
import { allFramePresets, defaultFramePreset, type FramePreset } from '@/utils/framePresets'
import { allQrCodePresets, defaultPreset, type Preset } from '@/utils/qrCodePresets'
import { useMediaQuery } from '@vueuse/core'
import JSZip from 'jszip'
import {
  type CornerDotType,
  type CornerSquareType,
  type DotType,
  type ErrorCorrectionLevel,
  type Options as StyledQRCodeProps
} from 'qr-code-styling'
import { computed, nextTick, onMounted, ref, watch } from 'vue'
import 'vue-i18n'
import { useI18n } from 'vue-i18n'

interface FrameStyle {
  textColor: string
  backgroundColor: string
  borderColor: string
  borderWidth: string
  borderRadius: string
  padding: string
}

const props = defineProps<{
  initialData?: string
}>()

const mainContentContainer = ref<HTMLElement | null>(null)
const isLarge = useMediaQuery('(min-width: 768px)')
const isLikelyMobileDevice = computed(() => {
  return typeof navigator !== 'undefined' && navigator.maxTouchPoints > 0
})

//#region /** locale */
const { t, locale } = useI18n()
//#endregion

//#region /* QR code style settings */
const data = ref(props.initialData || import.meta.env.VITE_DEFAULT_DATA_TO_ENCODE || '')
const debouncedData = ref(data.value)
const previewData = computed(() =>
  debouncedData.value?.length > 0 ? debouncedData.value : defaultQRCodeText.value
)
let dataDebounceTimer: ReturnType<typeof setTimeout>

watch(
  data,
  (newVal) => {
    clearTimeout(dataDebounceTimer)
    dataDebounceTimer = setTimeout(() => {
      debouncedData.value = newVal
    }, 500)
  },
  { immediate: true }
)
const image = ref()
const width = ref()
const height = ref()
const margin = ref()
const imageMargin = ref()

watch(
  () => props.initialData,
  (newValue) => {
    if (newValue) {
      data.value = newValue
    }
  }
)

const dotsOptionsColor = ref()
const dotsOptionsType = ref()
const cornersSquareOptionsColor = ref()
const cornersSquareOptionsType = ref()
const cornersDotOptionsColor = ref()
const cornersDotOptionsType = ref()
const styleBorderRadius = ref()
const styledBorderRadiusFormatted = computed(() => `${styleBorderRadius.value}px`)
const styleBackground = ref(defaultPreset.style.background)
const lastBackground = ref(defaultPreset.style.background)
const includeBackground = ref(true)
watch(
  includeBackground,
  (newIncludeBackground) => {
    if (!newIncludeBackground) {
      lastBackground.value = styleBackground.value
      styleBackground.value = 'transparent'
    } else {
      styleBackground.value = lastBackground.value
    }
  },
  {
    immediate: true
  }
)

const dotsOptions = computed(() => ({
  color: dotsOptionsColor.value,
  type: dotsOptionsType.value
}))
const cornersSquareOptions = computed(() => ({
  color: cornersSquareOptionsColor.value,
  type: cornersSquareOptionsType.value
}))
const cornersDotOptions = computed(() => ({
  color: cornersDotOptionsColor.value,
  type: cornersDotOptionsType.value
}))
const style = computed(() => ({
  borderRadius: styledBorderRadiusFormatted.value,
  background: styleBackground.value
}))
const imageOptions = computed(() => ({
  margin: imageMargin.value
}))
const qrOptions = computed(() => ({
  errorCorrectionLevel: errorCorrectionLevel.value
}))

const qrCodeProps = computed<StyledQRCodeProps>(() => ({
  data: previewData.value,
  image: image.value,
  width: width.value,
  height: height.value,
  margin: margin.value,
  dotsOptions: dotsOptions.value,
  cornersSquareOptions: cornersSquareOptions.value,
  cornersDotOptions: cornersDotOptions.value,
  imageOptions: imageOptions.value,
  qrOptions: qrOptions.value
}))

function randomizeStyleSettings() {
  const dotTypes: DotType[] = [
    'dots',
    'rounded',
    'classy',
    'classy-rounded',
    'square',
    'extra-rounded'
  ]
  const cornerSquareTypes: CornerSquareType[] = ['dot', 'square', 'extra-rounded']
  const cornerDotTypes: CornerDotType[] = ['dot', 'square']

  dotsOptionsType.value = getRandomItemInArray(dotTypes)
  dotsOptionsColor.value = createRandomColor()

  cornersSquareOptionsType.value = getRandomItemInArray(cornerSquareTypes)
  cornersSquareOptionsColor.value = createRandomColor()

  cornersDotOptionsType.value = getRandomItemInArray(cornerDotTypes)
  cornersDotOptionsColor.value = createRandomColor()

  styleBackground.value = createRandomColor()
}

function uploadImage() {
  console.debug('Uploading image')
  const imageInput = document.createElement('input')
  imageInput.type = 'file'
  imageInput.accept = 'image/*'
  imageInput.onchange = (event: Event) => {
    const target = event.target as HTMLInputElement
    if (target.files) {
      const file = target.files[0]
      const reader = new FileReader()
      reader.onload = (event: ProgressEvent<FileReader>) => {
        const target = event.target as FileReader
        const result = target.result as string
        image.value = result
      }
      reader.readAsDataURL(file)
    }
  }
  imageInput.click()
}
// #endregion

// #region /* Preset settings */
const isPresetSelectOpen = ref(false)
const allPresetOptions = computed(() => {
  const options = lastCustomLoadedPreset.value
    ? [lastCustomLoadedPreset.value, ...allQrCodePresets]
    : allQrCodePresets
  return options.map((preset) => ({ value: preset.name, label: t(preset.name) }))
})
const selectedPreset = ref<
  Preset & { key?: string; qrOptions?: { errorCorrectionLevel: ErrorCorrectionLevel } }
>(defaultPreset)
watch(selectedPreset, () => {
  // Note: We no longer auto-fill data from presets. Users can keep their own data
  // while changing the visual style. The QR preview will show default text if empty.

  image.value = selectedPreset.value.image
  width.value = selectedPreset.value.width
  height.value = selectedPreset.value.height
  margin.value = selectedPreset.value.margin
  imageMargin.value = selectedPreset.value.imageOptions.margin
  dotsOptionsColor.value = selectedPreset.value.dotsOptions.color
  dotsOptionsType.value = selectedPreset.value.dotsOptions.type
  cornersSquareOptionsColor.value = selectedPreset.value.cornersSquareOptions.color
  cornersSquareOptionsType.value = selectedPreset.value.cornersSquareOptions.type
  cornersDotOptionsColor.value = selectedPreset.value.cornersDotOptions.color
  cornersDotOptionsType.value = selectedPreset.value.cornersDotOptions.type
  styleBorderRadius.value = getNumericCSSValue(selectedPreset.value.style.borderRadius as string)
  styleBackground.value = selectedPreset.value.style.background
  includeBackground.value = selectedPreset.value.style.background !== 'transparent'
  errorCorrectionLevel.value =
    selectedPreset.value.qrOptions && selectedPreset.value.qrOptions.errorCorrectionLevel
      ? selectedPreset.value.qrOptions.errorCorrectionLevel
      : 'Q'
  // Most presets don't have a frame, so we set it to false by default
})

const LAST_LOADED_LOCALLY_PRESET_KEY = 'Last saved locally'
const LOADED_FROM_FILE_PRESET_KEY = 'Loaded from file'
const CUSTOM_LOADED_PRESET_KEYS = [LAST_LOADED_LOCALLY_PRESET_KEY, LOADED_FROM_FILE_PRESET_KEY]
const selectedPresetKey = ref<string>(
  import.meta.env.VITE_DISABLE_LOCAL_STORAGE === 'true'
    ? defaultPreset.name
    : localStorage.getItem('qrCodeConfig')
      ? LAST_LOADED_LOCALLY_PRESET_KEY
      : defaultPreset.name
)
const lastCustomLoadedPreset = ref<Preset>()
watch(
  selectedPresetKey,
  (newKey, prevKey) => {
    if (newKey === prevKey || !newKey) return

    if (
      import.meta.env.VITE_DISABLE_LOCAL_STORAGE !== 'true' &&
      CUSTOM_LOADED_PRESET_KEYS.includes(newKey) &&
      lastCustomLoadedPreset.value
    ) {
      selectedPreset.value = lastCustomLoadedPreset.value
      return
    }

    const updatedPreset = allQrCodePresets.find((preset) => preset.name === newKey)
    if (updatedPreset) {
      selectedPreset.value = updatedPreset
    }
  },
  { immediate: true }
)
//#endregion

//#region /* Error correction level */
const errorCorrectionLevels: ErrorCorrectionLevel[] = ['L', 'M', 'Q', 'H']
const errorCorrectionLevel = ref<ErrorCorrectionLevel>('Q')
const ERROR_CORRECTION_LEVEL_LABELS: Record<ErrorCorrectionLevel, string> = {
  L: `Low (7%)`,
  M: `Medium (15%)`,
  Q: `High (25%)`,
  H: `Highest (30%)`
}
const recommendedErrorCorrectionLevel = computed<ErrorCorrectionLevel | null>(() => {
  if (!data.value) return null
  if (data.value.length <= 50) {
    return 'H'
  } else if (data.value.length <= 150) {
    return 'Q'
  } else if (data.value.length <= 500) {
    return 'M'
  } else {
    return 'L'
  }
})

const errorCorrectionOptions = computed(() =>
  errorCorrectionLevels.map((level) => ({
    value: level,
    label: `${t(ERROR_CORRECTION_LEVEL_LABELS[level])}${level === recommendedErrorCorrectionLevel.value ? ' ✓' : ''}`
  }))
)
//#endregion

//#region /* Dropdown options for QR code styling */
const dotsTypeOptions = computed(() =>
  ['dots', 'rounded', 'classy', 'classy-rounded', 'square', 'extra-rounded'].map((type) => ({
    value: type,
    label: t(type)
  }))
)

const cornersSquareTypeOptions = computed(() =>
  ['dot', 'square', 'extra-rounded'].map((type) => ({
    value: type,
    label: t(type)
  }))
)

const cornersDotTypeOptions = computed(() =>
  ['dot', 'square'].map((type) => ({
    value: type,
    label: t(type)
  }))
)
//#endregion

//#region /* Data type selection for QR code content */
const selectedDataType = ref<string>('url')

const dataTypeOptions = computed(() => [
  { value: 'url', label: t('URL'), icon: 'link' },
  { value: 'text', label: t('Text'), icon: 'text' },
  { value: 'wifi', label: t('WiFi'), icon: 'wifi' },
  { value: 'vcard', label: t('vCard'), icon: 'user' },
  { value: 'email', label: t('Email'), icon: 'mail' },
  { value: 'phone', label: t('Phone'), icon: 'phone' },
  { value: 'sms', label: t('SMS'), icon: 'message' },
  { value: 'location', label: t('Location'), icon: 'map' },
  { value: 'event', label: t('Event'), icon: 'calendar' }
])

// Email refs
const emailAddress = ref('')
const emailSubject = ref('')
const emailBody = ref('')
const emailCc = ref('')
const emailBcc = ref('')

// Phone refs
const phoneNumber = ref('')

// SMS refs
const smsNumber = ref('')
const smsMessage = ref('')

// WiFi refs
const wifiSSID = ref('')
const wifiPassword = ref('')
const wifiEncryption = ref('nopass')
const wifiHidden = ref(false)

// vCard refs
const vcardFirstName = ref('')
const vcardLastName = ref('')
const vcardOrg = ref('')
const vcardPosition = ref('')
const vcardPhoneWork = ref('')
const vcardPhonePrivate = ref('')
const vcardPhoneMobile = ref('')
const vcardEmail = ref('')
const vcardWebsite = ref('')
const vcardStreet = ref('')
const vcardZipcode = ref('')
const vcardCity = ref('')
const vcardState = ref('')
const vcardCountry = ref('')
const vcardVersion = ref('2')

// Location refs
const locationLatitude = ref<number | string>('')
const locationLongitude = ref<number | string>('')

// Event refs
const eventTitle = ref('')
const eventLocation = ref('')
const eventStartTime = ref('')
const eventEndTime = ref('')

// WiFi encryption options
const wifiEncryptionOptions = computed(() => [
  { value: 'nopass', label: t('No encryption') },
  { value: 'WEP', label: 'WEP' },
  { value: 'WPA', label: 'WPA/WPA2' }
])

// vCard version options
const vcardVersionOptions = computed(() => [
  { value: '2', label: 'vCard 2.1' },
  { value: '3', label: 'vCard 3.0' },
  { value: '4', label: 'vCard 4.0' }
])

const handleDataTypeSelect = (type: string) => {
  selectedDataType.value = type
  data.value = ''
}

// Generate data string based on selected type
const generateDataFromFields = () => {
  let generatedString = ''
  switch (selectedDataType.value) {
    case 'email':
      generatedString = generateEmailData({
        address: emailAddress.value,
        subject: emailSubject.value,
        body: emailBody.value,
        cc: emailCc.value,
        bcc: emailBcc.value
      })
      break
    case 'phone':
      generatedString = generatePhoneData({ phone: phoneNumber.value })
      break
    case 'sms':
      generatedString = generateSmsData({ phone: smsNumber.value, message: smsMessage.value })
      break
    case 'wifi':
      generatedString = generateWifiData({
        ssid: wifiSSID.value,
        password: wifiPassword.value,
        encryption: wifiEncryption.value as 'nopass' | 'WEP' | 'WPA',
        hidden: wifiHidden.value
      })
      break
    case 'vcard':
      generatedString = generateVCardData({
        firstName: vcardFirstName.value,
        lastName: vcardLastName.value,
        org: vcardOrg.value,
        position: vcardPosition.value,
        phoneWork: vcardPhoneWork.value,
        phonePrivate: vcardPhonePrivate.value,
        phoneMobile: vcardPhoneMobile.value,
        email: vcardEmail.value,
        website: vcardWebsite.value,
        street: vcardStreet.value,
        zipcode: vcardZipcode.value,
        city: vcardCity.value,
        state: vcardState.value,
        country: vcardCountry.value,
        version: vcardVersion.value
      })
      break
    case 'location':
      generatedString = generateLocationData({
        latitude: locationLatitude.value,
        longitude: locationLongitude.value
      })
      break
    case 'event':
      generatedString = generateEventData({
        title: eventTitle.value,
        location: eventLocation.value,
        startTime: eventStartTime.value,
        endTime: eventEndTime.value
      })
      break
  }
  if (generatedString) {
    data.value = generatedString
  }
}

// Watch for changes in data type specific fields and auto-generate
watch([emailAddress, emailSubject, emailBody, emailCc, emailBcc], () => {
  if (selectedDataType.value === 'email' && emailAddress.value) {
    generateDataFromFields()
  }
})

watch(phoneNumber, () => {
  if (selectedDataType.value === 'phone' && phoneNumber.value) {
    generateDataFromFields()
  }
})

watch([smsNumber, smsMessage], () => {
  if (selectedDataType.value === 'sms' && smsNumber.value) {
    generateDataFromFields()
  }
})

watch([wifiSSID, wifiPassword, wifiEncryption, wifiHidden], () => {
  if (selectedDataType.value === 'wifi' && wifiSSID.value) {
    generateDataFromFields()
  }
})

watch([vcardFirstName, vcardLastName, vcardOrg, vcardPosition, vcardPhoneWork, vcardPhonePrivate, vcardPhoneMobile, vcardEmail, vcardWebsite, vcardStreet, vcardZipcode, vcardCity, vcardState, vcardCountry, vcardVersion], () => {
  if (selectedDataType.value === 'vcard') {
    generateDataFromFields()
  }
})

watch([locationLatitude, locationLongitude], () => {
  if (selectedDataType.value === 'location' && locationLatitude.value && locationLongitude.value) {
    generateDataFromFields()
  }
})

watch([eventTitle, eventLocation, eventStartTime, eventEndTime], () => {
  if (selectedDataType.value === 'event' && eventTitle.value) {
    generateDataFromFields()
  }
})

// Fill with example data
const fillWithExampleData = () => {
  const now = new Date()
  const oneHourLater = new Date(now.getTime() + 60 * 60 * 1000)
  const formatForInput = (date: Date) => {
    const year = date.getFullYear()
    const month = (date.getMonth() + 1).toString().padStart(2, '0')
    const day = date.getDate().toString().padStart(2, '0')
    const hours = date.getHours().toString().padStart(2, '0')
    const minutes = date.getMinutes().toString().padStart(2, '0')
    return `${year}-${month}-${day}T${hours}:${minutes}`
  }

  switch (selectedDataType.value) {
    case 'text':
      data.value = 'Have a wonderful day!'
      break
    case 'url':
      data.value = 'https://github.com/lyqht/mini-qr'
      break
    case 'email':
      emailAddress.value = 'test@example.com'
      emailSubject.value = 'QR Code Test'
      emailBody.value = 'This is a test email from cheQR!'
      emailCc.value = 'cc@example.com'
      emailBcc.value = 'bcc@example.com'
      break
    case 'phone':
      phoneNumber.value = '+19876543210'
      break
    case 'sms':
      smsNumber.value = '+19876543210'
      smsMessage.value = 'Hello from cheQR!'
      break
    case 'wifi':
      wifiSSID.value = 'MyWiFiNetwork'
      wifiEncryption.value = 'WPA'
      wifiPassword.value = 'Password123'
      wifiHidden.value = false
      break
    case 'vcard':
      vcardFirstName.value = 'Jane'
      vcardLastName.value = 'Smith'
      vcardOrg.value = 'Example Inc.'
      vcardPosition.value = 'Developer'
      vcardPhoneWork.value = '+1-555-000-1111'
      vcardPhonePrivate.value = ''
      vcardPhoneMobile.value = '+1-555-222-3333'
      vcardEmail.value = 'jane.smith@example.com'
      vcardWebsite.value = 'https://example.com'
      vcardStreet.value = '456 Example Ave'
      vcardCity.value = 'Othertown'
      vcardState.value = 'NY'
      vcardZipcode.value = '10001'
      vcardCountry.value = 'USA'
      break
    case 'location':
      locationLatitude.value = '37.7749'
      locationLongitude.value = '-122.4194'
      break
    case 'event':
      eventTitle.value = 'Project Sync'
      eventLocation.value = 'Online'
      eventStartTime.value = formatForInput(now)
      eventEndTime.value = formatForInput(oneHourLater)
      break
  }
}
//#endregion

//#region /* Frame settings */ Start empty, default is set intelligently */
const defaultFrameText = computed(() => t('Scan for more info'))
const frameText = ref<string>('')
const frameTextPosition = ref<'top' | 'bottom' | 'left' | 'right'>('bottom')
const showFrame = ref(false)

//#region /* Default QR code text */
const defaultQRCodeText = computed(() => t('Have nice day!'))

const frameStyle = ref<FrameStyle>({
  textColor: '#000000',
  backgroundColor: '#ffffff',
  borderColor: '#000000',
  borderWidth: '1px',
  borderRadius: '8px',
  padding: '16px'
})

const selectedFramePresetKey = ref<string>(defaultFramePreset.name)
const lastCustomLoadedFramePreset = ref<FramePreset>()
const CUSTOM_LOADED_FRAME_PRESET_KEYS = [
  LAST_LOADED_LOCALLY_PRESET_KEY,
  LOADED_FROM_FILE_PRESET_KEY
]

const allFramePresetOptions = computed(() => {
  const options = lastCustomLoadedFramePreset.value
    ? [lastCustomLoadedFramePreset.value, ...allFramePresets]
    : allFramePresets
  return options.map((preset) => ({ value: preset.name, label: t(preset.name) }))
})

function applyFramePreset(preset: FramePreset) {
  if (preset.style) {
    frameStyle.value = { ...frameStyle.value, ...preset.style }
  }
  if (preset.text) frameText.value = preset.text
  if (preset.position) frameTextPosition.value = preset.position
}

watch(
  selectedFramePresetKey,
  (newKey, prevKey) => {
    if (newKey === prevKey || !newKey) return

    if (
      import.meta.env.VITE_DISABLE_LOCAL_STORAGE !== 'true' &&
      CUSTOM_LOADED_FRAME_PRESET_KEYS.includes(newKey) &&
      lastCustomLoadedFramePreset.value
    ) {
      applyFramePreset(lastCustomLoadedFramePreset.value)
      return
    }

    const preset = allFramePresets.find((p) => p.name === newKey)
    if (preset) {
      applyFramePreset(preset)
    }
  },
  { immediate: true }
)

const frameSettings = computed(() => ({
  text: frameText.value,
  position: frameTextPosition.value,
  style: frameStyle.value
}))
//#endregion

//#region /* Frame text autofill */ Fill if empty */
watch(locale, () => {
  if (frameText.value.trim() === '') {
    frameText.value = defaultFrameText.value
  }
})

watch(defaultFrameText, (now, prev) => {
  const untouched = frameText.value.trim() === '' || frameText.value === prev
  if (untouched) {
    frameText.value = now
  }
})

watch(showFrame, (on) => {
  if (on && frameText.value.trim() === '') {
    frameText.value = defaultFrameText.value
  }
})

onMounted(() => {
  if (frameText.value.trim() === '') {
    frameText.value = defaultFrameText.value
  }
})
//#endregion

//#region /* QR code text autofill -  Fill if empty */
watch(locale, () => {
  if (!props.initialData && data.value.trim() === '') {
    data.value = defaultQRCodeText.value
  }
})

watch(defaultQRCodeText, (now, prev) => {
  const untouched = !props.initialData && (data.value.trim() === '' || data.value === prev)
  if (untouched) {
    data.value = now
  }
})

//#endregion

//#region /* General Export - download qr code and copy to clipboard */
const isExportButtonDisabled = computed(() => {
  if (exportMode.value === ExportMode.Single) {
    return !data.value
  }
  return dataStringsFromCsv.value.length === 0
})

const PREVIEW_QRCODE_DIM_UNIT = 200

/**
 * Calculates the dimensions for QR code export
 * When frame is enabled (showFrame = true), dimensions are calculated from the actual rendered element
 * to include the frame's size. Otherwise, uses the configured width and height values.
 */
function getExportDimensions() {
  if (!showFrame.value) {
    return {
      width: width.value,
      height: height.value
    }
  }
  const el = document.getElementById('element-to-export')
  if (!el) {
    return {
      width: width.value,
      height: height.value
    }
  }

  // Calculate the scale factor based on the preview size
  const scaleFactor = width.value / PREVIEW_QRCODE_DIM_UNIT

  const elWidth = el.offsetWidth
  const elHeight = el.offsetHeight

  // Get the actual dimensions including the frame and apply the scale factor
  return {
    width: elWidth * scaleFactor,
    height: elHeight * scaleFactor
  }
}

// #region Copy image modal (Safari fallback)
const showSafariCopyImageModal = ref(false)
const copyModalIsLoading = ref(false)
const copyModalImageSrc = ref<string | null>(null)

async function openCopyModal() {
  const el = document.getElementById('element-to-export')
  if (!el) return
  copyModalIsLoading.value = true
  try {
    copyModalImageSrc.value = await getPngElement(
      el,
      getExportDimensions(),
      styledBorderRadiusFormatted.value
    )
    showSafariCopyImageModal.value = true
  } catch (error) {
    console.error('Error preparing image for copy modal:', error)
  } finally {
    copyModalIsLoading.value = false
  }
}

function closeCopyModal() {
  showSafariCopyImageModal.value = false
  copyModalImageSrc.value = null
}
// #endregion

function copyQRToClipboard() {
  const el = document.getElementById('element-to-export')
  if (!el) {
    return
  }
  if (IS_COPY_IMAGE_TO_CLIPBOARD_SUPPORTED) {
    copyImageToClipboard(el, getExportDimensions(), styledBorderRadiusFormatted.value)
  } else if (!isLikelyMobileDevice.value) {
    // for now we only open the copy image modal on safari desktop because
    // this modal will be hidden behind the export image modal on mobile viewport.
    openCopyModal()
  }
}

/**
 * Downloads QR code in specified format, handling both single and batch exports
 * @param format The format to download: 'png', 'svg', or 'jpg'
 */
function downloadQRImage(format: 'png' | 'svg' | 'jpg') {
  if (exportMode.value === ExportMode.Single) {
    // Sanitize filename to remove invalid characters
    const sanitizedFilename = (exportFilename.value || 'qr-code').replace(/[^a-zA-Z0-9_-]/g, '_')

    const formatConfig = {
      png: { fn: downloadPngElement, filename: `${sanitizedFilename}.png` },
      svg: { fn: downloadSvgElement, filename: `${sanitizedFilename}.svg` },
      jpg: {
        fn: downloadJpgElement,
        filename: `${sanitizedFilename}.jpg`,
        extraOptions: { bgcolor: 'white' }
      }
    }[format]

    const el = document.getElementById('element-to-export')
    if (!el) {
      return
    }

    formatConfig.fn(
      el,
      formatConfig.filename,
      { ...getExportDimensions(), ...formatConfig.extraOptions },
      styledBorderRadiusFormatted.value
    )
  } else {
    generateBatchQRCodes(format)
  }
}
//#endregion

//#region /* QR Config Utils - Saving, Loading and Downloading */
interface QRCodeConfig {
  props: StyledQRCodeProps & {
    name?: string
  }
  style: {
    borderRadius: string
    background?: string
  }
  frame?: {
    text: string
    position: 'top' | 'bottom' | 'left' | 'right'
    style: FrameStyle
  } | null
}

function createQrConfig(): QRCodeConfig {
  return {
    props: qrCodeProps.value,
    style: style.value,
    frame: showFrame.value ? frameSettings.value : null
  }
}

function downloadQRConfig() {
  console.debug('Downloading QR code config')
  const qrCodeConfig = createQrConfig()
  const qrCodeConfigString = JSON.stringify(qrCodeConfig)
  const qrCodeConfigBlob = new Blob([qrCodeConfigString], { type: 'application/json' })
  const qrCodeConfigUrl = URL.createObjectURL(qrCodeConfigBlob)
  const qrCodeConfigLink = document.createElement('a')
  qrCodeConfigLink.href = qrCodeConfigUrl
  qrCodeConfigLink.download = 'qr-code-config.json'
  qrCodeConfigLink.click()
}

function saveQRConfigToLocalStorage() {
  const qrCodeConfig = createQrConfig()
  const qrCodeConfigString = JSON.stringify(qrCodeConfig)
  localStorage.setItem('qrCodeConfig', qrCodeConfigString)
}

function loadQRConfig(jsonString: string, key?: string) {
  const qrCodeConfig = JSON.parse(jsonString) as QRCodeConfig
  const qrCodeProps = qrCodeConfig.props
  const qrCodeStyle = qrCodeConfig.style
  const frameConfig = qrCodeConfig.frame

  const preset = {
    ...qrCodeProps,
    style: qrCodeStyle
  } as Preset

  if (key) {
    preset.name = key
    lastCustomLoadedPreset.value = preset
    selectedPresetKey.value = key
  }

  let framePreset: FramePreset | undefined

  selectedPreset.value = preset

  if (frameConfig) {
    showFrame.value = true
    frameText.value = frameConfig.text || defaultFrameText.value
    frameTextPosition.value = frameConfig.position || 'bottom'
    frameStyle.value = {
      ...frameStyle.value,
      ...frameConfig.style
    }
    framePreset = {
      name: key || LAST_LOADED_LOCALLY_PRESET_KEY,
      style: frameConfig.style,
      text: frameConfig.text,
      position: frameConfig.position
    }
  }

  if (framePreset && key) {
    lastCustomLoadedFramePreset.value = framePreset
    selectedFramePresetKey.value = key
  }
}

function loadQrConfigFromFile() {
  console.debug('Loading QR code config')
  const qrCodeConfigInput = document.createElement('input')
  qrCodeConfigInput.type = 'file'
  qrCodeConfigInput.accept = 'application/json'
  qrCodeConfigInput.onchange = (event: Event) => {
    const target = event.target as HTMLInputElement
    if (target.files) {
      const file = target.files[0]
      const reader = new FileReader()
      reader.onload = (event: ProgressEvent<FileReader>) => {
        const target = event.target as FileReader
        const result = target.result as string
        loadQRConfig(result, LOADED_FROM_FILE_PRESET_KEY)
      }
      reader.readAsText(file)
    }
  }
  qrCodeConfigInput.click()
}

watch(
  [qrCodeProps, style, showFrame, frameSettings],
  () => {
    saveQRConfigToLocalStorage()
  },
  {
    deep: true
  }
)

onMounted(() => {
  if (import.meta.env.VITE_DISABLE_LOCAL_STORAGE !== 'true') {
    const qrCodeConfigString = localStorage.getItem('qrCodeConfig')
    if (qrCodeConfigString) {
      loadQRConfig(qrCodeConfigString, LAST_LOADED_LOCALLY_PRESET_KEY)
    } else {
      // No localStorage data found, use the environment variable default preset
      selectedPreset.value = { ...defaultPreset }
      selectedPresetKey.value = defaultPreset.name
    }
    // No separate frameConfig loading from localStorage noted,
    // assuming selectedFramePresetKey watcher handles it if lastCustomLoadedFramePreset was populated by loadQRConfig
  }

  // Set initial data if provided through props
  if (props.initialData) {
    data.value = props.initialData
  } else if (data.value.trim() === '') {
    data.value = defaultQRCodeText.value
  }
})
//#endregion

//#region /* Batch QR Code Generation */
enum ExportMode {
  Single = 'single',
  Batch = 'batch'
}

const exportFilename = ref('qr-code')
const exportMode = ref(ExportMode.Single)
const dataStringsFromCsv = ref<string[]>([])
const frameTextsFromCsv = ref<string[]>([])
const fileNamesFromCsv = ref<string[]>([])

const inputFileForBatchEncoding = ref<File | null>(null)
const fileInput = ref<HTMLInputElement | null>(null)
const isValidCsv = ref(true)

const isExportingBatchQRs = ref(false)
const isBatchExportSuccess = ref(false)
const currentExportedQrCodeIndex = ref<number | null>(null)

const parsedCsvResult = ref<{ data: CSVParsingResult } | null>(null)
const previewRowIndex = ref(0)
const previewRow = computed(() => {
  const idx = previewRowIndex.value
  if (dataStringsFromCsv.value.length === 0) return null
  if (idx < 0 || idx >= dataStringsFromCsv.value.length) return null
  if (
    parsedCsvResult.value?.data &&
    Array.isArray(parsedCsvResult.value.data) &&
    parsedCsvResult.value.data.length > idx
  ) {
    return parsedCsvResult.value.data[idx]
  }
  return null
})

const resetBatchExportProgress = () => {
  isExportingBatchQRs.value = false
  currentExportedQrCodeIndex.value = null
  usedFilenames.clear()
}

const resetData = () => {
  data.value = ''
  inputFileForBatchEncoding.value = null
  dataStringsFromCsv.value = []
  frameTextsFromCsv.value = []
  fileNamesFromCsv.value = []
  isValidCsv.value = true
  resetBatchExportProgress()
  isBatchExportSuccess.value = false
}

watch(exportMode, () => {
  resetData()
})

watch(previewRowIndex, (newIndex) => {
  if (
    exportMode.value === ExportMode.Batch &&
    dataStringsFromCsv.value.length > 0 &&
    newIndex >= 0 &&
    newIndex < dataStringsFromCsv.value.length
  ) {
    data.value = dataStringsFromCsv.value[newIndex]
    frameText.value = frameTextsFromCsv.value[newIndex] || defaultFrameText.value
  }
})

const getFileFromInputEvent = (event: InputEvent) => {
  const inputElement = event.target as HTMLInputElement
  if (inputElement.files && inputElement.files.length > 0) {
    return inputElement.files[0]
  }
  return null
}

const onBatchInputFileUpload = (event: Event) => {
  isBatchExportSuccess.value = false
  let file: File | null = getFileFromInputEvent(event as InputEvent)

  // If it is not input event, then it might be a drag and drop event
  if (file == null) {
    const dt = (event as DragEvent).dataTransfer
    if (!dt || !dt.files || dt.files.length === 0) {
      return
    }
    file = dt.files[0]
  }

  inputFileForBatchEncoding.value = file
  const reader = new FileReader()
  reader.onload = (e) => {
    const content = e.target?.result
    if (typeof content !== 'string') {
      isValidCsv.value = false
      return
    }

    const result = parseCSV(content)
    parsedCsvResult.value = result
    if (!result.isValid) {
      isValidCsv.value = false
      return
    }

    if (!validateCSVData(result.data)) {
      isValidCsv.value = false
      return
    }

    // Process CSV data using the utility function
    const batchResult = processCsvDataForBatch(result.data)

    dataStringsFromCsv.value = batchResult.urls
    frameTextsFromCsv.value = batchResult.frameTexts
    fileNamesFromCsv.value = batchResult.fileNames
    showFrame.value = batchResult.hasCustomFrameText
    isValidCsv.value = true
    previewRowIndex.value = 0 // Reset preview to first row on new upload

    // Update the QR code preview with the first row's data
    if (batchResult.urls.length > 0) {
      data.value = batchResult.urls[0]
      frameText.value = batchResult.frameTexts[0] || defaultFrameText.value
    }
  }

  reader.readAsText(file)
}

const sleep = (ms: number) => new Promise((resolve) => setTimeout(resolve, ms))

const usedFilenames = new Set<string>()
const createZipFile = (
  zip: typeof JSZip,
  dataUrl: string,
  index: number,
  format: 'png' | 'svg' | 'jpg'
) => {
  const dataString = dataStringsFromCsv.value[index]
  const frameText = frameTextsFromCsv.value[index]
  const customFileName = fileNamesFromCsv.value[index]

  const fileName = generateBatchExportFilename(
    dataString,
    frameText,
    customFileName,
    index,
    usedFilenames
  )

  // Sanitize filename to remove invalid characters
  const sanitizedFileName = fileName.replace(/[^a-zA-Z0-9_-]/g, '_')

  if (format === 'png' || format === 'jpg') {
    zip.file(`${sanitizedFileName}.${format}`, dataUrl.split(',')[1], { base64: true })
  } else {
    // For SVG, we don't need to split and use base64
    zip.file(`${sanitizedFileName}.${format}`, dataUrl)
  }
}
async function generateBatchQRCodes(format: 'png' | 'svg' | 'jpg') {
  isExportingBatchQRs.value = true
  const zip = new JSZip()
  let numQrCodesCreated = 0
  const el = document.getElementById('element-to-export')
  if (!el) {
    return
  }

  try {
    for (let index = 0; index < dataStringsFromCsv.value.length; index++) {
      currentExportedQrCodeIndex.value = index
      const url = dataStringsFromCsv.value[index]
      const currentFrameText = frameTextsFromCsv.value[index]
      data.value = url
      frameText.value = currentFrameText
      await sleep(1000)
      let dataUrl: string = ''
      if (format === 'png') {
        dataUrl = await getPngElement(el, getExportDimensions(), styledBorderRadiusFormatted.value)
      } else if (format === 'jpg') {
        dataUrl = await getJpgElement(el, getExportDimensions(), styledBorderRadiusFormatted.value)
      } else {
        dataUrl = await getSvgString(el, getExportDimensions(), styledBorderRadiusFormatted.value)
      }
      createZipFile(zip, dataUrl, index, format)
      numQrCodesCreated++
    }

    while (numQrCodesCreated !== dataStringsFromCsv.value.length) {
      await sleep(100)
    }

    zip.generateAsync({ type: 'blob' }).then((content) => {
      const link = document.createElement('a')
      link.href = URL.createObjectURL(content)
      link.download = `qr-codes.zip`
      link.click()
      isBatchExportSuccess.value = true
    })
  } catch (error) {
    console.error('Error generating batch QR codes', error)
    isBatchExportSuccess.value = false
  } finally {
    resetBatchExportProgress()
  }
}
// #endregion
</script>

<template>
  <div
    class="create-page-layout flex items-start justify-center gap-6 overflow-x-hidden md:flex-row lg:pb-0"
  >
    <!-- Preview Panel (Left side on large screens) -->
    <div
      v-if="isLarge"
      ref="mainContentContainer"
      id="main-content-container"
      class="preview-panel sticky top-6 flex w-[420px] shrink-0 flex-col p-8 overflow-y-auto"
      style="height: calc(100vh - 48px); max-height: 1000px;"
    ></div>
    <!-- Bottom sheet on small screens -->
    <Drawer v-else>
      <DrawerTrigger
        id="drawer-preview-container"
        class="fixed inset-x-0 bottom-0 z-10 rounded-t-[32px] border-t outline-none backdrop-blur-xl focus-visible:ring-2 focus-visible:ring-black/20"
        style="background: var(--bg-surface); border-color: var(--border-light); box-shadow: 0 -10px 40px rgba(0,0,0,0.1);"
      >
        <div class="flex flex-col items-center">
          <!-- Handle indicator for bottom sheet -->
          <div class="mt-3 h-1.5 w-12 rounded-full" style="background: var(--border-light);"></div>
          <div :class="['w-full', '-my-8']">
            <div class="flex origin-center scale-[0.7] items-center justify-center md:scale-100">
              <QRCodeFrame
                v-if="showFrame"
                :frame-text="frameText"
                :text-position="frameTextPosition"
                :frame-style="frameStyle"
              >
                <template #qr-code>
                  <div id="qr-code-container" class="grid place-items-center">
                    <div
                      class="grid place-items-center overflow-hidden"
                      :style="[
                        style,
                        {
                          width: `${PREVIEW_QRCODE_DIM_UNIT}px`,
                          height: `${PREVIEW_QRCODE_DIM_UNIT}px`
                        }
                      ]"
                    >
                      <StyledQRCode
                        v-bind="{
                          ...qrCodeProps,
                          width: PREVIEW_QRCODE_DIM_UNIT,
                          height: PREVIEW_QRCODE_DIM_UNIT
                        }"
                        role="img"
                        aria-label="QR code"
                      />
                    </div>
                  </div>
                </template>
              </QRCodeFrame>
              <template v-else>
                <div class="grid place-items-center">
                  <div
                    class="grid place-items-center overflow-hidden"
                    :style="[
                      style,
                      {
                        width: `${PREVIEW_QRCODE_DIM_UNIT}px`,
                        height: `${PREVIEW_QRCODE_DIM_UNIT}px`
                      }
                    ]"
                  >
                    <StyledQRCode
                      v-bind="{
                        ...qrCodeProps,
                        width: PREVIEW_QRCODE_DIM_UNIT,
                        height: PREVIEW_QRCODE_DIM_UNIT
                      }"
                      role="img"
                      aria-label="QR code preview"
                    />
                  </div>
                </div>
              </template>
            </div>
          </div>
          <div
            class="flex items-center gap-1 py-2 text-center text-sm font-medium"
            style="color: var(--text-muted);"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="16"
              height="16"
              viewBox="0 0 24 24"
              class="inline"
            >
              <path fill="currentColor" d="M12 8l-6 6l1.41 1.41L12 10.83l4.59 4.58L18 14z" />
            </svg>
            {{ t('Export') }}
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="16"
              height="16"
              viewBox="0 0 24 24"
              class="inline"
            >
              <path fill="currentColor" d="M12 8l-6 6l1.41 1.41L12 10.83l4.59 4.58L18 14z" />
            </svg>
          </div>
        </div>
      </DrawerTrigger>
      <DrawerContent>
        <DrawerHeader>
          <DrawerTitle class="text-lg font-bold" style="color: var(--text-main); letter-spacing: -0.03em;">{{ t('Export QR code') }}</DrawerTitle>
        </DrawerHeader>
        <div class="overflow-y-auto overflow-x-hidden px-6 pb-6">
          <div ref="mainContentContainer" id="main-content-container" class="w-full"></div>
        </div>
      </DrawerContent>
    </Drawer>

    <!-- Main content -->
    <Teleport to="#main-content-container" v-if="mainContentContainer != null">
      <div id="main-content">
        <div class="flex items-center justify-between mb-6">
          <h2 class="text-2xl font-extrabold" style="color: var(--text-main); letter-spacing: -1px;">{{ t('Preview') }}</h2>
          <button 
            id="load-qr-code-config-button"
            class="btn btn-secondary shrink-0 gap-2" 
            style="height: 52px;"
            @click="loadQrConfigFromFile"
            :aria-label="t('Upload JSON Config')"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="20"
              height="20"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
            >
              <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4" />
              <polyline points="17 8 12 3 7 8" />
              <line x1="12" y1="3" x2="12" y2="15" />
            </svg>
            {{ t('Upload JSON Config') }}
          </button>
        </div>
        <div
          id="qr-code-container"
          class="qr-stage mb-6"
          :class="[
            showFrame && ['left', 'right'].includes(frameTextPosition) && 'scale-[0.7] md:scale-100'
          ]"
        >
          <div v-if="showFrame" id="element-to-export">
            <QRCodeFrame
              :frame-text="frameText"
              :text-position="frameTextPosition"
              :frame-style="frameStyle"
            >
              <template #qr-code>
                <div id="qr-code-container" class="grid place-items-center">
                  <div
                    class="grid place-items-center overflow-hidden"
                    :style="[
                      style,
                      {
                        width: `${PREVIEW_QRCODE_DIM_UNIT}px`,
                        height: `${PREVIEW_QRCODE_DIM_UNIT}px`
                      }
                    ]"
                  >
                    <StyledQRCode
                      v-bind="{
                        ...qrCodeProps,
                        width: PREVIEW_QRCODE_DIM_UNIT,
                        height: PREVIEW_QRCODE_DIM_UNIT
                      }"
                      role="img"
                      aria-label="QR code"
                    />
                  </div>
                </div>
              </template>
            </QRCodeFrame>
          </div>
          <div
            v-else
            id="element-to-export"
            class="grid place-items-center overflow-hidden"
            :style="[
              style,
              {
                width: `${PREVIEW_QRCODE_DIM_UNIT}px`,
                height: `${PREVIEW_QRCODE_DIM_UNIT}px`
              }
            ]"
          >
            <StyledQRCode
              v-bind="{
                ...qrCodeProps,
                data: data?.length > 0 ? data : t('Have nice day!'),
                width: PREVIEW_QRCODE_DIM_UNIT,
                height: PREVIEW_QRCODE_DIM_UNIT
              }"
              role="img"
              aria-label="QR code"
            />
          </div>
        </div>
        <!-- Export Card -->
        <div class="export-card">
          <h3 class="text-lg font-semibold mb-4" style="color: var(--text-main);">{{ t('Export QR Code') }}</h3>
          <div v-if="exportMode === ExportMode.Single" class="mb-4">
            <label for="export-filename" class="input-label">{{ t('File name') }}</label>
            <input
              id="export-filename"
              type="text"
              class="text-input"
              style="background: white;"
              v-model="exportFilename"
              placeholder="my-qr-code"
            />
          </div>
          <!-- All export buttons in one grid -->
          <div class="grid grid-cols-5 gap-2 mb-4">
            <button
              id="download-qr-image-button-png"
              class="btn btn-primary"
              style="height: 40px;"
              @click="() => downloadQRImage('png')"
              :disabled="isExportButtonDisabled"
              :aria-label="t('Download QR Code as PNG')"
            >
              PNG
            </button>
            <button
              id="download-qr-image-button-jpg"
              class="btn btn-secondary"
              style="height: 40px; background: white;"
              @click="() => downloadQRImage('jpg')"
              :disabled="isExportButtonDisabled"
              :aria-label="t('Download QR Code as JPG')"
            >
              JPG
            </button>
            <button
              id="download-qr-image-button-svg"
              class="btn btn-secondary"
              style="height: 40px; background: white;"
              @click="() => downloadQRImage('svg')"
              :disabled="isExportButtonDisabled"
              :aria-label="t('Download QR Code as SVG')"
            >
              SVG
            </button>
            <button
              v-if="exportMode !== ExportMode.Batch"
              id="copy-qr-image-button"
              class="btn btn-secondary"
              style="height: 40px; background: white;"
              @click="copyQRToClipboard"
              :disabled="isExportButtonDisabled"
              :aria-label="t('Copy QR Code to clipboard')"
            >
              {{ t('Copy') }}
            </button>
            <button
              id="save-qr-code-config-button"
              class="btn btn-secondary"
              style="height: 40px; background: white;"
              @click="downloadQRConfig"
              :aria-label="t('Save QR Code configuration')"
            >
              JSON
            </button>
          </div>
        </div>
      </div>
    </Teleport>

    <section id="settings" class="settings-panel flex w-full grow flex-col items-start gap-4 text-start p-8">
      <h2 class="sr-only">{{ t('Settings to customize your QR code') }}</h2>
      <Accordion
        type="multiple"
        collapsible
        class="flex w-full flex-col gap-3"
        :default-value="['data-settings', 'qr-code-settings']"
      >
        <AccordionItem value="frame-settings" class="accordion-item">
          <AccordionTrigger
            class="accordion-header"
            ><span id="frame-settings-title" class="text-lg font-semibold" style="color: var(--text-main);">{{ t('Frame settings') }}</span
            ></AccordionTrigger
          >
          <AccordionContent class="accordion-content">
            <section class="w-full space-y-4" aria-labelledby="frame-settings-title">
              <label class="checkbox-wrapper flex items-center gap-3 cursor-pointer" style="margin-bottom: var(--space-3);">
                <input id="show-frame" type="checkbox" v-model="showFrame" />
                <span class="text-base font-medium" style="color: var(--text-main);">{{ t('Add decorative frame') }}</span>
              </label>

              <template v-if="showFrame">
                <div class="input-group">
                  <label for="frame-text">{{ t('Frame Text') }}</label>
                  <textarea
                    name="frame-text"
                    class="text-input"
                    id="frame-text"
                    rows="2"
                    :placeholder="defaultFrameText"
                    v-model="frameText"
                  />
                </div>

                <div class="grid grid-cols-2 gap-3">
                  <label class="color-picker-wrapper cursor-pointer" for="frame-text-color">
                    <div class="color-swatch" :style="{ background: frameStyle.textColor, border: frameStyle.textColor === '#ffffff' ? '1px solid #ddd' : 'none' }"></div>
                    <span class="text-sm font-semibold" style="color: var(--text-main);">{{ t('Text Color') }}</span>
                    <input
                      id="frame-text-color"
                      type="color"
                      v-model="frameStyle.textColor"
                    />
                  </label>
                  <label class="color-picker-wrapper cursor-pointer" for="frame-bg-color">
                    <div class="color-swatch" :style="{ background: frameStyle.backgroundColor }"></div>
                    <span class="text-sm font-semibold" style="color: var(--text-main);">{{ t('Background') }}</span>
                    <input
                      id="frame-bg-color"
                      type="color"
                      v-model="frameStyle.backgroundColor"
                    />
                  </label>
                </div>
              </template>
            </section>
          </AccordionContent>
        </AccordionItem>
        <AccordionItem value="data-settings" class="accordion-item">
          <AccordionTrigger
            class="accordion-header"
            ><span id="data-settings-title" class="text-lg font-semibold" style="color: var(--text-main);">{{ t('Data to encode') }}</span></AccordionTrigger
          >
          <AccordionContent class="accordion-content">
            <section class="w-full space-y-4" aria-labelledby="data-settings-title">
              <div class="w-full">
                <div class="flex w-full flex-col flex-wrap gap-4 sm:flex-row sm:gap-x-8">
                  <div class="w-full sm:grow">
                    <!-- Mode Toggle + Use Example -->
                    <div class="flex items-center gap-3 flex-wrap mb-4">
                      <!-- Mode Toggle - Segmented Control -->
                      <div class="segmented-control" style="width: fit-content;">
                        <button
                          :class="['segmented-btn', exportMode === ExportMode.Single ? 'active' : '']"
                          @click="exportMode = ExportMode.Single"
                        >
                          {{ $t('Single') }}
                        </button>
                        <button
                          :class="['segmented-btn', exportMode === ExportMode.Batch ? 'active' : '']"
                          @click="exportMode = ExportMode.Batch"
                        >
                          {{ $t('Batch') }}
                        </button>
                      </div>
                      <!-- Use Example Button -->
                      <button
                        v-if="exportMode === ExportMode.Single"
                        @click="fillWithExampleData"
                        class="btn btn-secondary gap-2"
                        style="height: 36px; font-size: 13px; background: white; border: 1px solid var(--border-light);"
                      >
                        <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                          <path d="M8 10h8m-8 4h6m4-9H4a1 1 0 0 0-1 1v14a1 1 0 0 0 1 1h16a1 1 0 0 0 1-1V6a1 1 0 0 0-1-1Z"/>
                        </svg>
                        {{ t('Use example') }}
                      </button>
                    </div>
                    <!-- Single Mode Input -->
                    <div v-if="exportMode === ExportMode.Single" class="flex flex-col items-start w-full">
                      <!-- Data Type Pills with Icons -->
                      <div class="data-type-pills mb-4">
                        <button
                          v-for="dataType in dataTypeOptions"
                          :key="dataType.value"
                          :class="['data-type-pill', selectedDataType === dataType.value ? 'active' : '']"
                          @click="handleDataTypeSelect(dataType.value)"
                        >
                          <!-- URL Icon -->
                          <svg v-if="dataType.icon === 'link'" xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                            <path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"/>
                            <path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"/>
                          </svg>
                          <!-- Text Icon -->
                          <svg v-if="dataType.icon === 'text'" xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                            <polyline points="4 7 4 4 20 4 20 7"/>
                            <line x1="9" y1="20" x2="15" y2="20"/>
                            <line x1="12" y1="4" x2="12" y2="20"/>
                          </svg>
                          <!-- WiFi Icon -->
                          <svg v-if="dataType.icon === 'wifi'" xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                            <path d="M5 12.55a11 11 0 0 1 14.08 0"/>
                            <path d="M1.42 9a16 16 0 0 1 21.16 0"/>
                            <path d="M8.53 16.11a6 6 0 0 1 6.95 0"/>
                            <line x1="12" y1="20" x2="12.01" y2="20"/>
                          </svg>
                          <!-- User/vCard Icon -->
                          <svg v-if="dataType.icon === 'user'" xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                            <path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/>
                            <circle cx="12" cy="7" r="4"/>
                          </svg>
                          <!-- Mail Icon -->
                          <svg v-if="dataType.icon === 'mail'" xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                            <path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/>
                            <polyline points="22,6 12,13 2,6"/>
                          </svg>
                          <!-- Phone Icon -->
                          <svg v-if="dataType.icon === 'phone'" xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                            <path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z"/>
                          </svg>
                          <!-- SMS/Message Icon -->
                          <svg v-if="dataType.icon === 'message'" xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                            <path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/>
                          </svg>
                          <!-- Map/Location Icon -->
                          <svg v-if="dataType.icon === 'map'" xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                            <path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"/>
                            <circle cx="12" cy="10" r="3"/>
                          </svg>
                          <!-- Calendar/Event Icon -->
                          <svg v-if="dataType.icon === 'calendar'" xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                            <rect x="3" y="4" width="18" height="18" rx="2" ry="2"/>
                            <line x1="16" y1="2" x2="16" y2="6"/>
                            <line x1="8" y1="2" x2="8" y2="6"/>
                            <line x1="3" y1="10" x2="21" y2="10"/>
                          </svg>
                          {{ dataType.label }}
                        </button>
                      </div>

                      <!-- URL/Text Input (simple textarea) -->
                      <div v-if="selectedDataType === 'url' || selectedDataType === 'text'" class="w-full">
                        <textarea
                          id="data"
                          v-model="data"
                          class="w-full text-input"
                          rows="3"
                          :placeholder="selectedDataType === 'url' ? 'https://example.com' : t('Enter your text here')"
                        ></textarea>
                      </div>

                      <!-- Email Input Fields -->
                      <div v-if="selectedDataType === 'email'" class="w-full space-y-3">
                        <div>
                          <label class="input-label">{{ t('Email Address') }} <span class="text-red-500">*</span></label>
                          <input type="email" v-model="emailAddress" class="text-input w-full" placeholder="recipient@example.com" />
                        </div>
                        <div>
                          <label class="input-label">{{ t('CC') }}</label>
                          <input type="email" v-model="emailCc" class="text-input w-full" :placeholder="t('Optional CC')" />
                        </div>
                        <div>
                          <label class="input-label">{{ t('BCC') }}</label>
                          <input type="email" v-model="emailBcc" class="text-input w-full" :placeholder="t('Optional BCC')" />
                        </div>
                        <div>
                          <label class="input-label">{{ t('Subject') }}</label>
                          <input type="text" v-model="emailSubject" class="text-input w-full" :placeholder="t('Optional subject line')" />
                        </div>
                        <div>
                          <label class="input-label">{{ t('Message') }}</label>
                          <textarea v-model="emailBody" class="text-input w-full" rows="2" :placeholder="t('Optional email body')"></textarea>
                        </div>
                      </div>

                      <!-- Phone Input Fields -->
                      <div v-if="selectedDataType === 'phone'" class="w-full space-y-3">
                        <div>
                          <label class="input-label">{{ t('Phone Number') }} <span class="text-red-500">*</span></label>
                          <input type="tel" v-model="phoneNumber" class="text-input w-full" placeholder="+1234567890" />
                        </div>
                      </div>

                      <!-- SMS Input Fields -->
                      <div v-if="selectedDataType === 'sms'" class="w-full space-y-3">
                        <div>
                          <label class="input-label">{{ t('Phone Number') }} <span class="text-red-500">*</span></label>
                          <input type="tel" v-model="smsNumber" class="text-input w-full" placeholder="+1234567890" />
                        </div>
                        <div>
                          <label class="input-label">{{ t('Message') }}</label>
                          <textarea v-model="smsMessage" class="text-input w-full" rows="2" :placeholder="t('Optional SMS message')"></textarea>
                        </div>
                      </div>

                      <!-- WiFi Input Fields -->
                      <div v-if="selectedDataType === 'wifi'" class="w-full space-y-3">
                        <div>
                          <label class="input-label">{{ t('Encryption') }}</label>
                          <Combobox
                            :items="wifiEncryptionOptions"
                            v-model:value="wifiEncryption"
                            :button-label="t('Select encryption type')"
                          />
                        </div>
                        <div>
                          <label class="input-label">{{ t('Wireless SSID') }} <span class="text-red-500">*</span></label>
                          <input type="text" v-model="wifiSSID" class="text-input w-full" :placeholder="t('Your network name')" />
                        </div>
                        <div>
                          <label class="input-label">{{ t('Password') }} <span v-if="wifiEncryption !== 'nopass'" class="text-red-500">*</span></label>
                          <input type="password" v-model="wifiPassword" class="text-input w-full" :disabled="wifiEncryption === 'nopass'" :placeholder="t('Network password')" />
                        </div>
                        <label class="checkbox-wrapper flex items-center gap-3 cursor-pointer">
                          <input type="checkbox" v-model="wifiHidden" />
                          <span class="text-sm" style="color: var(--text-main);">{{ t('Hidden SSID') }}</span>
                        </label>
                      </div>

                      <!-- vCard Input Fields -->
                      <div v-if="selectedDataType === 'vcard'" class="w-full space-y-3">
                        <div>
                          <label class="input-label">{{ t('vCard Version') }}</label>
                          <Combobox
                            :items="vcardVersionOptions"
                            v-model:value="vcardVersion"
                            :button-label="t('Select vCard version')"
                          />
                        </div>
                        <div class="grid grid-cols-2 gap-3">
                          <div>
                            <label class="input-label">{{ t('First Name') }}</label>
                            <input type="text" v-model="vcardFirstName" class="text-input w-full" :placeholder="t('First Name')" />
                          </div>
                          <div>
                            <label class="input-label">{{ t('Last Name') }}</label>
                            <input type="text" v-model="vcardLastName" class="text-input w-full" :placeholder="t('Last Name')" />
                          </div>
                        </div>
                        <div class="grid grid-cols-2 gap-3">
                          <div>
                            <label class="input-label">{{ t('Organization') }}</label>
                            <input type="text" v-model="vcardOrg" class="text-input w-full" :placeholder="t('Company')" />
                          </div>
                          <div>
                            <label class="input-label">{{ t('Position') }}</label>
                            <input type="text" v-model="vcardPosition" class="text-input w-full" :placeholder="t('Job Title')" />
                          </div>
                        </div>
                        <div class="grid grid-cols-3 gap-3">
                          <div>
                            <label class="input-label">{{ t('Work Phone') }}</label>
                            <input type="tel" v-model="vcardPhoneWork" class="text-input w-full" placeholder="+1-555-000-0000" />
                          </div>
                          <div>
                            <label class="input-label">{{ t('Mobile') }}</label>
                            <input type="tel" v-model="vcardPhoneMobile" class="text-input w-full" placeholder="+1-555-000-0000" />
                          </div>
                          <div>
                            <label class="input-label">{{ t('Private') }}</label>
                            <input type="tel" v-model="vcardPhonePrivate" class="text-input w-full" placeholder="+1-555-000-0000" />
                          </div>
                        </div>
                        <div class="grid grid-cols-2 gap-3">
                          <div>
                            <label class="input-label">{{ t('Email') }}</label>
                            <input type="email" v-model="vcardEmail" class="text-input w-full" placeholder="email@example.com" />
                          </div>
                          <div>
                            <label class="input-label">{{ t('Website') }}</label>
                            <input type="url" v-model="vcardWebsite" class="text-input w-full" placeholder="https://example.com" />
                          </div>
                        </div>
                        <div>
                          <label class="input-label">{{ t('Street') }}</label>
                          <input type="text" v-model="vcardStreet" class="text-input w-full" :placeholder="t('Street Address')" />
                        </div>
                        <div class="grid grid-cols-2 gap-3">
                          <div>
                            <label class="input-label">{{ t('City') }}</label>
                            <input type="text" v-model="vcardCity" class="text-input w-full" :placeholder="t('City')" />
                          </div>
                          <div>
                            <label class="input-label">{{ t('State') }}</label>
                            <input type="text" v-model="vcardState" class="text-input w-full" :placeholder="t('State/Province')" />
                          </div>
                        </div>
                        <div class="grid grid-cols-2 gap-3">
                          <div>
                            <label class="input-label">{{ t('Zip Code') }}</label>
                            <input type="text" v-model="vcardZipcode" class="text-input w-full" :placeholder="t('Postal Code')" />
                          </div>
                          <div>
                            <label class="input-label">{{ t('Country') }}</label>
                            <input type="text" v-model="vcardCountry" class="text-input w-full" :placeholder="t('Country')" />
                          </div>
                        </div>
                      </div>

                      <!-- Location Input Fields -->
                      <div v-if="selectedDataType === 'location'" class="w-full space-y-3">
                        <div class="grid grid-cols-2 gap-3">
                          <div>
                            <label class="input-label">{{ t('Latitude') }} <span class="text-red-500">*</span></label>
                            <input type="number" step="any" v-model="locationLatitude" class="text-input w-full" placeholder="37.7749" />
                          </div>
                          <div>
                            <label class="input-label">{{ t('Longitude') }} <span class="text-red-500">*</span></label>
                            <input type="number" step="any" v-model="locationLongitude" class="text-input w-full" placeholder="-122.4194" />
                          </div>
                        </div>
                      </div>

                      <!-- Event Input Fields -->
                      <div v-if="selectedDataType === 'event'" class="w-full space-y-3">
                        <div>
                          <label class="input-label">{{ t('Event Title') }} <span class="text-red-500">*</span></label>
                          <input type="text" v-model="eventTitle" class="text-input w-full" :placeholder="t('Event name')" />
                        </div>
                        <div>
                          <label class="input-label">{{ t('Location') }}</label>
                          <input type="text" v-model="eventLocation" class="text-input w-full" :placeholder="t('Event location')" />
                        </div>
                        <div class="grid grid-cols-2 gap-3">
                          <div>
                            <label class="input-label">{{ t('Start Time') }} <span class="text-red-500">*</span></label>
                            <input type="datetime-local" v-model="eventStartTime" class="text-input w-full" />
                          </div>
                          <div>
                            <label class="input-label">{{ t('End Time') }} <span class="text-red-500">*</span></label>
                            <input type="datetime-local" v-model="eventEndTime" class="text-input w-full" />
                          </div>
                        </div>
                      </div>
                    </div>
                    <template v-if="exportMode === ExportMode.Batch">
                      <template v-if="!inputFileForBatchEncoding">
                        <BatchExportFieldsGuide />
                        <button
                          class="!ms-0 mt-4 flex items-center justify-center rounded-lg border-2 border-dashed border-gray-300 p-1 py-4 text-center text-input"
                          :aria-label="t('Choose a CSV file containing data to encode')"
                          @click="fileInput?.click()"
                          @keyup.enter="fileInput?.click()"
                          @keyup.space="fileInput?.click()"
                          @dragover.prevent
                          @drop.prevent="onBatchInputFileUpload"
                        >
                          <div class="flex flex-col items-center">
                            <svg
                              xmlns="http://www.w3.org/2000/svg"
                              width="48"
                              height="48"
                              viewBox="0 0 24 24"
                              class="mb-2 text-gray-400"
                            >
                              <path
                                fill="currentColor"
                                d="M11 16V7.85l-2.6 2.6L7 9l5-5l5 5l-1.4 1.45l-2.6-2.6V16h-2Zm-5 4q-.825 0-1.413-.588T4 18v-3h2v3h12v-3h2v3q0 .825-.588 1.413T18 20H6Z"
                              />
                            </svg>
                            <p aria-hidden="true" class="text-sm">
                              {{ $t('Upload a CSV file') }}
                            </p>
                          </div>
                          <input
                            ref="fileInput"
                            type="file"
                            accept=".csv,.txt"
                            class="hidden"
                            @change="onBatchInputFileUpload"
                          />
                        </button>
                      </template>
                      <div v-else-if="isValidCsv" class="p-4 text-center">
                        <div v-if="isBatchExportSuccess">
                          <p>{{ $t('QR codes have been successfully exported.') }}</p>
                          <button class="button mt-4" @click="inputFileForBatchEncoding = null">
                            {{ $t('Start new batch export') }}
                          </button>
                        </div>
                        <div v-else-if="currentExportedQrCodeIndex == null && !isExportingBatchQRs">
                          <div v-if="dataStringsFromCsv.length > 0" class="mt-4">
                            <div
                              class="flex flex-col gap-2 rounded-lg border border-gray-200 bg-gray-50 p-4 dark:border-gray-700 dark:bg-gray-800"
                            >
                              <div v-if="previewRow && 'firstName' in previewRow">
                                <VCardPreview :vCard="previewRow" />
                              </div>
                              <div v-else>
                                <div class="space-y-2">
                                  <div class="flex flex-col gap-1">
                                    <span
                                      class="text-xs font-medium text-gray-500 dark:text-gray-400"
                                      >{{ $t('Data') }}</span
                                    >
                                    <code
                                      class="rounded bg-white px-2 py-1 font-mono text-sm dark:bg-gray-900"
                                    >
                                      {{ dataStringsFromCsv[previewRowIndex] }}
                                    </code>
                                  </div>
                                  <div v-if="frameTextsFromCsv[previewRowIndex]">
                                    <span
                                      class="text-xs font-medium text-gray-500 dark:text-gray-400"
                                      >{{ $t('Frame text') }}</span
                                    >
                                    <code
                                      class="rounded bg-white px-2 py-1 font-mono text-sm dark:bg-gray-900"
                                    >
                                      {{ frameTextsFromCsv[previewRowIndex] }}
                                    </code>
                                  </div>
                                  <div v-if="fileNamesFromCsv[previewRowIndex]">
                                    <span
                                      class="text-xs font-medium text-gray-500 dark:text-gray-400"
                                      >{{ $t('File name') }}</span
                                    >
                                    <code
                                      class="rounded bg-white px-2 py-1 font-mono text-sm dark:bg-gray-900"
                                    >
                                      {{ fileNamesFromCsv[previewRowIndex] }}
                                    </code>
                                  </div>
                                </div>
                              </div>
                              <div class="mt-2 flex items-center justify-between">
                                <button
                                  class="rounded bg-gray-200 px-2 py-1 text-gray-700 disabled:opacity-50 dark:bg-gray-700 dark:text-gray-200 dark:disabled:opacity-60"
                                  :disabled="previewRowIndex === 0"
                                  @click="previewRowIndex--"
                                >
                                  &lt;
                                </button>
                                <span class="text-xs text-gray-500 dark:text-gray-400"
                                  >{{ previewRowIndex + 1 }} / {{ dataStringsFromCsv.length }}</span
                                >
                                <button
                                  class="rounded bg-gray-200 px-2 py-1 text-gray-700 disabled:opacity-50 dark:bg-gray-700 dark:text-gray-200 dark:disabled:opacity-60"
                                  :disabled="previewRowIndex === dataStringsFromCsv.length - 1"
                                  @click="previewRowIndex++"
                                >
                                  &gt;
                                </button>
                              </div>
                            </div>
                          </div>
                        </div>
                        <div v-else-if="currentExportedQrCodeIndex != null">
                          <p>{{ $t('Creating QR codes... This may take a while.') }}</p>
                          <p>
                            {{
                              $t('{index} / {count} QR codes have been created.', {
                                index: currentExportedQrCodeIndex + 1,
                                count: dataStringsFromCsv.length
                              })
                            }}
                          </p>
                        </div>
                      </div>
                      <div v-else class="p-4 text-center text-red-500">
                        <p>{{ $t('Invalid CSV') }}</p>
                      </div>
                    </template>
                  </div>
                </div>
              </div>
            </section>
          </AccordionContent>
        </AccordionItem>
        <AccordionItem value="qr-code-settings" class="accordion-item">
          <AccordionTrigger
            class="accordion-header"
            ><span id="qr-code-settings-title" class="text-lg font-semibold" style="color: var(--text-main);">{{ t('QR code settings') }}</span></AccordionTrigger
          >
          <AccordionContent class="accordion-content">
            <section class="w-full space-y-4" aria-labelledby="qr-code-settings-title">
              <div>
                <label>{{ t('Preset') }}</label>
                <div class="flex flex-row items-center justify-start gap-2">
                  <Combobox
                    :items="allPresetOptions"
                    v-model:value="selectedPresetKey"
                    v-model:open="isPresetSelectOpen"
                    :button-label="t('Select QR code preset')"
                    :insert-divider-at-indexes="[0, 2]"
                  />
                  <button
                    class="button grid size-10 place-items-center"
                    @click="randomizeStyleSettings"
                    :aria-label="t('Randomize style')"
                  >
                    <svg
                      xmlns="http://www.w3.org/2000/svg"
                      width="24"
                      height="24"
                      viewBox="0 0 640 512"
                    >
                      <path
                        fill="#888888"
                        d="M274.9 34.3c-28.1-28.1-73.7-28.1-101.8 0L34.3 173.1c-28.1 28.1-28.1 73.7 0 101.8l138.8 138.8c28.1 28.1 73.7 28.1 101.8 0l138.8-138.8c28.1-28.1 28.1-73.7 0-101.8L274.9 34.3zM200 224a24 24 0 1 1 48 0a24 24 0 1 1-48 0zM96 200a24 24 0 1 1 0 48a24 24 0 1 1 0-48zm128 176a24 24 0 1 1 0-48a24 24 0 1 1 0 48zm128-176a24 24 0 1 1 0 48a24 24 0 1 1 0-48zm-128-80a24 24 0 1 1 0-48a24 24 0 1 1 0 48zm96 328c0 35.3 28.7 64 64 64h192c35.3 0 64-28.7 64-64V256c0-35.3-28.7-64-64-64H461.7c11.6 36 3.1 77-25.4 105.5L320 413.8V448zm160-120a24 24 0 1 1 0 48a24 24 0 1 1 0-48z"
                      />
                    </svg>
                  </button>
                </div>
              </div>
              <div class="w-full">
                <label for="image-url" class="mb-2 block">{{ t('Logo image') }}</label>
                <div class="flex gap-2">
                  <input
                    name="image-url"
                    class="text-input flex-1"
                    id="image-url"
                    type="text"
                    :placeholder="t('Paste image URL or upload')"
                    v-model="image"
                  />
                  <button 
                    class="btn btn-secondary shrink-0 gap-2" 
                    style="height: 52px;"
                    @click="uploadImage"
                    :aria-label="t('Upload image')"
                  >
                    <svg
                      xmlns="http://www.w3.org/2000/svg"
                      width="20"
                      height="20"
                      viewBox="0 0 24 24"
                      fill="none"
                      stroke="currentColor"
                      stroke-width="2"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                    >
                      <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4" />
                      <polyline points="17 8 12 3 7 8" />
                      <line x1="12" y1="3" x2="12" y2="15" />
                    </svg>
                    {{ t('Upload') }}
                  </button>
                </div>
              </div>
              <div class="flex flex-row items-center gap-2">
                <label for="with-background">
                  {{ t('With background') }}
                </label>
                <input id="with-background" type="checkbox" v-model="includeBackground" />
              </div>
              <div>
                <label class="mb-3 block">{{ t('Colors') }}</label>
                <div id="color-settings" class="flex flex-wrap gap-3">
                  <label
                    :inert="!includeBackground"
                    :class="[!includeBackground && 'opacity-30', 'color-picker-wrapper cursor-pointer']"
                    for="background-color"
                  >
                    <div class="color-swatch" :style="{ background: styleBackground, border: styleBackground === '#ffffff' ? '1px solid #ddd' : 'none' }"></div>
                    <span class="text-sm font-semibold" style="color: var(--text-main);">{{ t('Bg') }}</span>
                    <input
                      id="background-color"
                      type="color"
                      v-model="styleBackground"
                    />
                  </label>
                  <label class="color-picker-wrapper cursor-pointer" for="dots-color">
                    <div class="color-swatch" :style="{ background: dotsOptionsColor }"></div>
                    <span class="text-sm font-semibold" style="color: var(--text-main);">{{ t('Dots') }}</span>
                    <input
                      id="dots-color"
                      type="color"
                      v-model="dotsOptionsColor"
                    />
                  </label>
                  <label class="color-picker-wrapper cursor-pointer" for="corners-square-color">
                    <div class="color-swatch" :style="{ background: cornersSquareOptionsColor }"></div>
                    <span class="text-sm font-semibold" style="color: var(--text-main);">{{ t('Corners') }}</span>
                    <input
                      id="corners-square-color"
                      type="color"
                      v-model="cornersSquareOptionsColor"
                    />
                  </label>
                </div>
              </div>
              <div class="grid grid-cols-3 gap-4">
                <div class="input-group">
                  <label for="width">{{ t('Width') }}</label>
                  <input
                    class="text-input"
                    id="width"
                    type="number"
                    placeholder="1080"
                    v-model="width"
                  />
                </div>
                <div class="input-group">
                  <label for="height">{{ t('Height') }}</label>
                  <input
                    class="text-input"
                    id="height"
                    type="number"
                    placeholder="1080"
                    v-model="height"
                  />
                </div>
                <div class="input-group">
                  <label for="border-radius">{{ t('Radius') }}</label>
                  <input
                    class="text-input"
                    id="border-radius"
                    type="number"
                    placeholder="16"
                    v-model="styleBorderRadius"
                  />
                </div>
              </div>
              <div class="flex w-full flex-col gap-4 sm:flex-row sm:gap-8">
                <div class="w-full sm:w-1/2">
                  <label for="margin">
                    {{ t('Margin (px)') }}
                  </label>
                  <input
                    class="text-input"
                    id="margin"
                    type="number"
                    placeholder="0"
                    v-model="margin"
                  />
                </div>
                <div class="w-full sm:w-1/2">
                  <label for="image-margin">
                    {{ t('Image margin (px)') }}
                  </label>
                  <input
                    class="text-input"
                    id="image-margin"
                    type="number"
                    placeholder="0"
                    v-model="imageMargin"
                  />
                </div>
              </div>
              <div
                id="dots-squares-settings"
                class="grid grid-cols-2 gap-4"
              >
                <div class="input-group">
                  <label for="dots-type">{{ t('Dots type') }}</label>
                  <Combobox
                    :items="dotsTypeOptions"
                    v-model:value="dotsOptionsType"
                    :button-label="t('Select dots type')"
                  />
                </div>
                <div class="input-group">
                  <label for="corners-square-type">{{ t('Corners Square') }}</label>
                  <Combobox
                    :items="cornersSquareTypeOptions"
                    v-model:value="cornersSquareOptionsType"
                    :button-label="t('Select corners square type')"
                  />
                </div>
                <div class="input-group">
                  <label for="corners-dot-type">{{ t('Corners Dot') }}</label>
                  <Combobox
                    :items="cornersDotTypeOptions"
                    v-model:value="cornersDotOptionsType"
                    :button-label="t('Select corners dot type')"
                  />
                </div>
                <div class="input-group">
                  <div class="flex items-center gap-2 mb-2">
                    <label for="error-correction" class="!mb-0">{{ t('Error Correction') }}</label>
                    <a
                      href="https://docs.uniqode.com/en/articles/7219782-what-is-the-recommended-error-correction-level-for-printing-a-qr-code"
                      target="_blank"
                      class="inline-flex items-center justify-center w-5 h-5 rounded-full"
                      style="background: var(--bg-input);"
                      :aria-label="t('What is error correction level?')"
                    >
                      <svg
                        xmlns="http://www.w3.org/2000/svg"
                        width="14"
                        height="14"
                        viewBox="0 0 24 24"
                      >
                        <path
                          fill="var(--text-muted)"
                          d="M11.95 18q.525 0 .888-.363t.362-.887t-.362-.888t-.888-.362t-.887.363t-.363.887t.363.888t.887.362m.05 4q-2.075 0-3.9-.788t-3.175-2.137T2.788 15.9T2 12t.788-3.9t2.137-3.175T8.1 2.788T12 2t3.9.788t3.175 2.137T21.213 8.1T22 12t-.788 3.9t-2.137 3.175t-3.175 2.138T12 22m0-2q3.35 0 5.675-2.325T20 12t-2.325-5.675T12 4T6.325 6.325T4 12t2.325 5.675T12 20m.1-12.3q.625 0 1.088.4t.462 1q0 .55-.337.975t-.763.8q-.575.5-1.012 1.1t-.438 1.35q0 .35.263.588t.612.237q.375 0 .638-.25t.337-.625q.1-.525.45-.937t.75-.788q.575-.55.988-1.2t.412-1.45q0-1.275-1.037-2.087T12.1 6q-.95 0-1.812.4T8.975 7.625q-.175.3-.112.638t.337.512q.35.2.725.125t.625-.425q.275-.375.688-.575t.862-.2"
                        />
                      </svg>
                    </a>
                  </div>
                  <Combobox
                    :items="errorCorrectionOptions"
                    v-model:value="errorCorrectionLevel"
                    :button-label="t('Select error correction level')"
                  />
                </div>
              </div>
            </section>
          </AccordionContent>
        </AccordionItem>
      </Accordion>
    </section>
  </div>

  <!-- Fallback modal for manual copy in Safari -->
  <CopyImageModal
    v-if="showSafariCopyImageModal"
    :is-loading="copyModalIsLoading"
    :image-src="copyModalImageSrc"
    @close="closeCopyModal"
  />
</template>
