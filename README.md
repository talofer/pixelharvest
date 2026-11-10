# Image Downloader & Bulk WebP/AVIF to JPG Converter - PixelHarvest

[![Manifest V3](https://img.shields.io/badge/Chrome-Manifest%20V3-10b981?style=flat-square&logo=googlechrome&logoColor=white)](https://talofer.github.io/pixelharvest/)
[![100% Local Processing](https://img.shields.io/badge/Privacy-100%25%20Client--Side-38bdf8?style=flat-square)](https://talofer.github.io/pixelharvest/privacy.html)
[![License: MIT](https://img.shields.io/badge/License-MIT-8b5cf6?style=flat-square)](LICENSE)

**PixelHarvest** is a zero-server, privacy-first **Chrome Manifest V3 extension** and [interactive web micro-tool](https://talofer.github.io/pixelharvest/) built for e-commerce operators, UI/UX designers, frontend engineers, and researchers who need to **bulk download images from any webpage** and **automatically convert stubborn `.webp`, `.avif`, and `.svg` files into clean `.jpg` or `.png` archives** with a single click.

🔗 **Try the Free Browser Drag & Drop Converter:** [https://talofer.github.io/pixelharvest/](https://talofer.github.io/pixelharvest/)

---

## Why Designers & E-Commerce Teams Switch to PixelHarvest

Modern CDNs (Cloudflare Polish, Shopify, Next.js `next/image`, Amazon CloudFront) serve images dynamically as `image/webp` or `image/avif`—even when the URL ends in `.jpg`. Legacy desktop workflows, older Adobe tools, ERP systems, and marketplace uploaders frequently reject `.webp` and `.avif` files.

PixelHarvest solves both **deep DOM extraction** and **instant format transcoding** in one step:

1. **Deep DOM Harvester (`src/content/harvester.ts`):**
   - Extracts standard `<img>` tags (`currentSrc` & `src`).
   - Parses responsive `<picture>` and `srcset` attributes to automatically select the **highest-resolution source** (`1x`, `2x`, `2000w`, etc.).
   - Inspects computed CSS `background-image: url(...)` properties across all layout containers.
   - Harvests inline `<svg>` vector nodes, strips unsafe scripts/event attributes via `DOMParser('image/svg+xml')`, injects missing `xmlns` namespaces, and serializes clean vector files.
2. **Real-Time Client-Side WebP/AVIF/SVG → JPG/PNG Transcoder (`src/popup/converter.ts`):**
   - Uses hardware-accelerated `fetch()` → `createImageBitmap(blob)` → offscreen HTML5 `<canvas>` (`0.92` quality) with `#FFFFFF` alpha compositing for JPG exports.
   - Runs through a bounded 3-worker semaphore concurrency pool with deterministic memory cleanup (`bitmap.close()`, `canvas.width = 0`, `URL.revokeObjectURL()`), preventing tab OOM crashes even on 4K image galleries.
3. **1-Click Sanitized `.zip` Packaging:**
   - Bundles up to 25 assets free per batch into a single `pixelharvest-export.zip` archive with Zip-Slip-proof filenames (`pixelharvest-001-name.jpg`).

---

## Feature & Privacy Comparison: PixelHarvest vs. Legacy Image Downloaders

| Capability / Security Metric | PixelHarvest (MV3) | Legacy Image Downloader Extensions | Cloud WebP Converter Websites |
| :--- | :--- | :--- | :--- |
| **Chrome Manifest Version** | **Manifest V3** (Service Worker) | Often deprecated Manifest V2 | N/A (Web Upload) |
| **Host Permissions Required** | **`activeTab` + `scripting` only** (Zero `<all_urls>`) | Broad `<all_urls>` (reads all browsing tabs) | Full file upload to remote servers |
| **WebP & AVIF → JPG/PNG Auto-Convert** | **Built-in (`createImageBitmap` + Canvas)** | Downloads raw `.webp` files unchanged | Requires manual upload & re-download |
| **Responsive `srcset` Highest-Res Selection** | **Yes (Automatic width/density parser)** | Usually grabs low-res thumbnail `src` | No DOM extraction capability |
| **Inline `<svg>` Extraction & XSS Sanitization** | **Yes (`DOMParser` XML sanitizer)** | Rarely supported or raw innerHTML | No DOM extraction capability |
| **Data Privacy & Server Uploads** | **100% Local Browser Memory** | Varies (often includes ad/tracker SDKs) | Uploads proprietary assets to 3rd-party cloud |

---

## Security & Architecture Highlights

- **Minimal Permission Footprint:** PixelHarvest requests only `activeTab`, `scripting`, `downloads`, and `storage`. It cannot run in the background or access pages unless you explicitly click the extension icon.
- **Cross-Origin Safe Fetching:** Extension-context `fetch(url, { mode: 'cors', credentials: 'omit' })` prevents tainted-canvas security errors on CDN-hosted images while never leaking session cookies.
- **Multi-Language Global Support (`_locales/`):** Native Chrome Web Store localization in English (`en`), German (`de`), Spanish (`es`), French (`fr`), and Japanese (`ja`).

---

## Technical Guides & Articles

- [How to Bulk Download & Convert WebP and AVIF Product Images for E-Commerce Workflows](docs/articles/webp-avif-ecommerce-workflow.md)
- [How to Audit Chrome Image Downloader Extensions for Manifest V3 Security & Privacy](docs/articles/safe-image-downloader-extensions.md)

---

## Legal & Compliance

- **Interactive Web Converter & Landing Hub:** [https://talofer.github.io/pixelharvest/](https://talofer.github.io/pixelharvest/)
- **Privacy Policy:** [https://talofer.github.io/pixelharvest/privacy.html](https://talofer.github.io/pixelharvest/privacy.html)
- **Terms of Service:** [https://talofer.github.io/pixelharvest/terms.html](https://talofer.github.io/pixelharvest/terms.html)
- **License:** [MIT License](LICENSE)
