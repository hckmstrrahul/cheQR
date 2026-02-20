# cheQR

**The world's simplest, most stylish custom QR code generator.**

Create beautiful, fully customizable QR codes in seconds. Scan any QR code instantly. Free forever.

---

## Features

### QR Code Creation
- **Customizable Styles** — Choose dot types (dots, rounded, classy, classy-rounded, square, extra-rounded), corner square types, and corner dot types
- **Full Color Control** — Background, dots, corner squares, and corner dots all have independent color pickers
- **Logo/Image Embedding** — Upload any image or paste a URL to embed a logo in your QR code
- **Presets** — Pre-built style presets for one-click beautiful QR codes (Padlet, Vercel, Supabase, VueJS, and more)
- **Randomize** — One-click random style generation for creative exploration
- **Error Correction Levels** — L (7%), M (15%), Q (25%), H (30%) with intelligent suggestions based on data size
- **Adjustable Dimensions** — Width, height, margin, image margin, and border radius controls
- **Transparent Background Support** — Toggle backgrounds on/off for overlaying QR codes on any design

### Frame Customization
- **Text Labels** — Add custom text around your QR code (top, bottom, left, right positions)
- **Frame Styling** — Text color, background color, border color, border width, border radius, and padding
- **Frame Presets** — Default, dark, and borderless frame presets

### Data Templates
Structured input forms for generating valid QR code data:
- **Text** — Plain text encoding
- **URL** — Web links with validation
- **Email** — `mailto:` links with subject, body, CC, and BCC
- **Phone** — `tel:` links
- **SMS** — `smsto:` links with message body
- **WiFi** — Network credentials (SSID, password, encryption type, hidden network)
- **vCard** — Digital contact cards (name, phone, email, organization, address, website, notes)
- **Location** — Geographic coordinates
- **Calendar Event** — iCal events with title, description, location, start/end times

### Export Options
- **PNG** — High-quality raster export
- **JPG** — Compressed raster export
- **SVG** — Vector export (limited support)
- **Clipboard** — Copy QR code image directly to clipboard
- **Config Save/Load** — Save QR code configurations as JSON, load them later or share with others
- **Custom Filenames** — Name your exported files

### Batch Export
- **CSV Import** — Upload a CSV file with multiple data entries
- **Supports vCard Batch** — Batch create contact QR codes from CSV
- **Frame Text Per Row** — Custom frame text for each QR code in the batch
- **ZIP Download** — All QR codes packaged in a single ZIP file
- **Progress Tracking** — Real-time progress during batch generation

### QR Code Scanning
- **Camera Scanning** — Scan QR codes with your device camera (front/back switching)
- **File Upload** — Upload or drag-and-drop QR code images
- **Clipboard Paste** — Paste images from clipboard (Ctrl/Cmd+V)
- **Type Detection** — Automatically detects URLs, emails, phone numbers, SMS, WiFi credentials, vCards, calendar events, and more
- **Quick Actions** — Open links, send emails, call numbers, copy text based on detected type

### General
- **30+ Languages** — Internationalized UI with support for English, German, French, Japanese, Chinese, Korean, Spanish, and many more
- **PWA Support** — Install as a desktop or mobile app
- **Responsive Design** — Optimized for desktop and mobile with adaptive layouts
- **LocalStorage Persistence** — Your last configuration is automatically saved and restored
- **Accessible** — WCAG-compliant with keyboard navigation, ARIA labels, and focus management

---

## Tech Stack

| Category | Technology |
|----------|-----------|
| Framework | Vue 3 (Composition API) |
| Language | TypeScript |
| Build Tool | Vite 5 |
| Styling | Tailwind CSS 3 + shadcn/vue (Radix Vue) |
| QR Generation | qr-code-styling |
| QR Scanning | html5-qrcode + qr-scanner |
| i18n | vue-i18n |
| Export | dom-to-image, dom-to-svg, jszip, file-saver |
| PWA | vite-plugin-pwa |
| UI Components | Radix Vue, Vaul Vue, Lucide Icons |

---

## Getting Started

### Prerequisites
- Node.js 18+
- npm (or pnpm/yarn)

### Installation

```bash
git clone <your-repo-url>
cd cheQR
npm install
```

### Development

```bash
npm run dev
```

The app will be available at `http://localhost:5173`.

### Build

```bash
npm run build
```

Production files are output to `dist/`.

### Preview Production Build

```bash
npm run preview
```

---

## Project Structure

```
cheQR/
├── src/
│   ├── App.vue                          # Root component (header, mode toggle, layout)
│   ├── main.js                          # App entry point (Vue, i18n, PWA registration)
│   ├── index.css                        # Tailwind + shadcn/ui CSS variables
│   ├── style.css                        # Global styles (buttons, inputs, typography)
│   ├── components/
│   │   ├── QRCodeCreate.vue             # Main QR code creator (settings, export, batch)
│   │   ├── QRCodeScan.vue               # QR code scanner (camera, file, clipboard)
│   │   ├── StyledQRCode.vue             # QR code renderer (qr-code-styling wrapper)
│   │   ├── QRCodeFrame.vue              # Frame wrapper with text labels
│   │   ├── QRCodeCameraScanner.vue      # Live camera scanning
│   │   ├── DataTemplatesModal.vue       # Data type input forms
│   │   ├── BatchExportFieldsGuide.vue   # CSV format guide for batch export
│   │   ├── VCardPreview.vue             # vCard contact preview card
│   │   ├── CopyImageModal.vue           # Safari clipboard fallback
│   │   ├── LanguageSelector.vue         # Language picker dropdown
│   │   ├── MobileMenu.vue              # Mobile hamburger menu
│   │   ├── AppFooter.vue               # Desktop footer
│   │   └── ui/                          # shadcn/vue UI primitives
│   │       ├── accordion/
│   │       ├── button/
│   │       ├── Combobox/
│   │       ├── command/
│   │       ├── dialog/
│   │       ├── drawer/
│   │       └── popover/
│   ├── utils/
│   │   ├── qrCodePresets.ts             # QR code style preset definitions
│   │   ├── framePresets.ts              # Frame style preset definitions
│   │   ├── convertToImage.ts            # DOM-to-image export (PNG, JPG, SVG)
│   │   ├── clipboard.ts                # Clipboard API utilities
│   │   ├── csv.ts                       # CSV parsing & validation
│   │   ├── csvBatchProcessing.ts        # Batch QR code processing
│   │   ├── dataEncoding.ts             # QR data format generators (WiFi, vCard, etc.)
│   │   ├── color.ts                     # Color utilities (random generation)
│   │   ├── i18n.ts                      # Vue i18n configuration (36 locales)
│   │   ├── language.ts                  # Language map & sorted locale list
│   │   ├── useDarkModePreference.ts     # Dark mode composable
│   │   ├── formatting.ts               # CSS value parsing
│   │   └── basePath.ts                 # Base path utilities for deployment
│   ├── lib/
│   │   └── utils.ts                     # cn() helper (clsx + tailwind-merge)
│   └── assets/
│       ├── presets/                      # Preset JSON configurations
│       └── placeholder_image.png
├── locales/                             # 36+ translation files (JSON)
├── public/                              # Static assets (icons, images)
├── index.html                           # HTML entry point
├── vite.config.js                       # Vite configuration
├── tailwind.config.js                   # Tailwind CSS configuration
├── tsconfig.json                        # TypeScript configuration
├── components.json                      # shadcn/ui configuration
├── postcss.config.js                    # PostCSS configuration
├── Dockerfile                           # Multi-stage Docker build
├── docker-compose.yml                   # Docker Compose setup
└── nginx.conf                           # Nginx configuration for Docker
```

---

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `BASE_PATH` | Base path for deployment (e.g., `/cheqr/`) | `/` |
| `VITE_HIDE_CREDITS` | Hide footer credits (`"true"` to hide) | `"false"` |
| `VITE_DEFAULT_PRESET` | Default QR code preset name | `""` |
| `VITE_DEFAULT_DATA_TO_ENCODE` | Pre-filled data on app load | `""` |
| `VITE_QR_CODE_PRESETS` | Custom presets JSON string | `"[]"` |
| `VITE_FRAME_PRESET` | Default frame preset name | `""` |
| `VITE_FRAME_PRESETS` | Custom frame presets JSON string | `"[]"` |
| `VITE_DISABLE_LOCAL_STORAGE` | Disable localStorage config loading | `"false"` |

Copy `.env.example` to `.env` and customize as needed.

---

## Docker Deployment

### Quick Start (prebuilt image)

```bash
docker compose up -d
```

App available at `http://localhost:8081`.

### Custom Build

```bash
docker compose up -d --build
```

### Deploy at Subdirectory

```bash
BASE_PATH=/cheqr docker compose up -d
```

---

## Scripts

| Script | Description |
|--------|-------------|
| `npm run dev` | Start development server |
| `npm run build` | Production build |
| `npm run preview` | Preview production build |
| `npm run vitest` | Run unit tests |
| `npm run test:e2e` | Run Playwright E2E tests |
| `npm run type-check` | TypeScript type checking |
| `npm run lint` | ESLint fix |
| `npm run format` | Prettier formatting |

---

## Credits

cheQR is built on top of [Mini QR](https://github.com/lyqht/mini-qr) by [Estee Tey](https://github.com/lyqht), licensed under GPL-3.0.

---

## License

[GPL-3.0](LICENSE)
