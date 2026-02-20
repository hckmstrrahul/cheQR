# cheQR — Architecture Guide

This document explains the technical architecture, data flow, and design decisions of cheQR.

---

## High-Level Architecture

```
┌──────────────────────────────────────────────────┐
│                    App.vue                        │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────┐ │
│  │ Desktop Hdr │  │ Mobile Hdr  │  │  Footer  │ │
│  └─────────────┘  └─────────────┘  └──────────┘ │
│  ┌──────────────────────────────────────────────┐│
│  │         Mode Toggle (Create / Scan)          ││
│  └──────────────────────────────────────────────┘│
│  ┌───────────────────┐  ┌───────────────────────┐│
│  │   QRCodeCreate    │  │     QRCodeScan        ││
│  │  ┌─────────────┐  │  │  ┌─────────────────┐  ││
│  │  │ StyledQRCode│  │  │  │ CameraScanner   │  ││
│  │  │ QRCodeFrame │  │  │  │ File Upload     │  ││
│  │  │ Settings    │  │  │  │ Clipboard Paste │  ││
│  │  │ Export      │  │  │  └─────────────────┘  ││
│  │  │ Batch       │  │  └───────────────────────┘│
│  │  └─────────────┘  │                           │
│  └───────────────────┘                           │
└──────────────────────────────────────────────────┘
```

---

## Data Flow

### QR Code Creation

```
User Input (text/URL/template)
  │
  ▼
data ref (debounced 500ms) ──→ previewData computed
  │                                    │
  ▼                                    ▼
Style Settings ──→ qrCodeProps ──→ StyledQRCode.vue
(preset / manual)     computed        │
                                      ▼
                                qr-code-styling lib
                                      │
                                      ▼
                               Rendered QR Code (SVG)
```

### Export Pipeline

```
DOM Element (#element-to-export)
  │
  ├──→ dom-to-image ──→ PNG / JPG (data URL)
  │                         │
  │                         ├──→ Download (file-saver)
  │                         ├──→ Clipboard (navigator.clipboard)
  │                         └──→ ZIP (jszip) for batch
  │
  └──→ dom-to-svg ──→ SVG string ──→ Download
```

### QR Code Scanning

```
Input Source
  ├──→ Camera (html5-qrcode)
  ├──→ File Upload
  └──→ Clipboard Paste
          │
          ▼
    QR Code Decoded
          │
          ▼
    Type Detection (dataEncoding.ts)
          │
          ▼
    ┌─────────────────────┐
    │ URL / Email / Phone │
    │ SMS / WiFi / vCard  │
    │ Calendar / Location │
    │ Text                │
    └─────────────────────┘
          │
          ▼
    Quick Actions (open, copy, create QR)
```

### Batch Export

```
CSV File Upload
  │
  ▼
parseCSV() ──→ validateCSVData()
                    │
                    ▼
          processCsvDataForBatch()
                    │
                    ▼
         ┌──────────────────────┐
         │ For each row:        │
         │  1. Set data + frame │
         │  2. Wait for render  │
         │  3. Capture to image │
         │  4. Add to ZIP       │
         └──────────────────────┘
                    │
                    ▼
            ZIP download (.zip)
```

---

## State Management

cheQR uses Vue 3 Composition API with `ref` and `computed` — no external store (Pinia/Vuex).

| State | Scope | Persistence |
|-------|-------|-------------|
| QR code style settings | `QRCodeCreate.vue` | localStorage (auto-save) |
| Selected preset | `QRCodeCreate.vue` | localStorage |
| Frame settings | `QRCodeCreate.vue` | localStorage |
| App mode (Create/Scan) | `App.vue` | None (session only) |
| Language preference | `LanguageSelector.vue` | localStorage |
| Scanned data | `QRCodeScan.vue` | None (passed via ref) |

---

## Key Libraries

| Library | Purpose | Used In |
|---------|---------|---------|
| `qr-code-styling` | QR code rendering with style options | `StyledQRCode.vue` |
| `html5-qrcode` | Camera-based QR scanning | `QRCodeCameraScanner.vue` |
| `qr-scanner` | Fallback file-based QR scanning | `QRCodeScan.vue` |
| `dom-to-image` | DOM element → PNG/JPG | `convertToImage.ts` |
| `dom-to-svg` | DOM element → SVG | `convertToImage.ts` |
| `jszip` | ZIP file creation for batch | `QRCodeCreate.vue` |
| `file-saver` | Trigger file downloads | `convertToImage.ts` |
| `vue-i18n` | Internationalization | Global (36 locales) |
| `radix-vue` / `reka-ui` | Headless UI primitives | `ui/` components |
| `vaul-vue` | Drawer component | Mobile bottom sheet |
| `@floating-ui/vue` | Tooltip/popover positioning | `MobileMenu.vue` |
| `marked` | Markdown → HTML | `AppFooter.vue` (changelog) |

---

## Presets System

### QR Code Presets (`qrCodePresets.ts`)

Each preset is a `Preset` object containing:
- `name` — Display name (i18n key)
- `image` — Logo image URL
- `width`, `height`, `margin` — Dimensions
- `dotsOptions` — Color + type for QR dots
- `cornersSquareOptions` — Color + type for corner squares
- `cornersDotOptions` — Color + type for corner dots
- `imageOptions` — Image margin
- `style` — Background color + border radius

Custom presets can be injected via `VITE_QR_CODE_PRESETS` environment variable (JSON string).

### Frame Presets (`framePresets.ts`)

Each frame preset contains:
- `name` — Display name
- `style` — textColor, backgroundColor, borderColor, borderWidth, borderRadius, padding
- `text` — Default frame text
- `position` — Text position (top/bottom/left/right)

Custom frame presets via `VITE_FRAME_PRESETS` environment variable.

---

## Internationalization

- 36 locales actively imported in `i18n.ts`
- Translation files in `locales/*.json`
- Default locale: `en`
- Language detection: browser language → localStorage preference
- All user-facing strings use `t('key')` from `vue-i18n`

---

## PWA Configuration

- Service worker: auto-update mode (`vite-plugin-pwa`)
- Caching: Workbox with network-first for pages, precaching for assets
- Manifest: standalone display, portrait orientation
- Icons: 192px and 512px (regular + maskable)

---

## Deployment Options

1. **Static hosting** — Build with `npm run build`, serve `dist/` from any static server
2. **Docker** — Multi-stage Dockerfile (Node build → Nginx serve)
3. **Subdirectory** — Set `BASE_PATH=/your-path` for non-root deployments
4. **Vercel/Netlify** — Zero-config deployment from git

---

## Adding a New Data Template

1. Add generator function in `src/utils/dataEncoding.ts`
2. Add detection logic in `detectDataType()` in the same file
3. Add form fields in `src/components/DataTemplatesModal.vue`
4. Add i18n keys in `locales/en.json` (and other locales)

## Adding a New Preset

1. Create a JSON file in `src/assets/presets/` with the preset configuration
2. Import and register it in `src/utils/qrCodePresets.ts`
3. The preset will automatically appear in the preset dropdown
