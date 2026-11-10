# How to Bulk Download & Convert WebP and AVIF Product Images for E-Commerce Workflows

Modern e-commerce platforms—including Shopify, WooCommerce, Magento, and custom headless storefronts powered by Next.js or Cloudflare Polish—automatically serve product photography as **WebP (`.webp`)** or **AVIF (`.avif`)**. While these next-generation formats reduce page weight by 30–50%, they frequently create a major bottleneck for merchandising teams, catalog managers, and digital designers.

When you right-click and choose *"Save image as…"* on a modern storefront, Chrome saves a `.webp` or `.avif` file—even if the image URL ends in `.jpg` or `.png`. Worse yet:
- Many legacy ERP systems, print-on-demand portals, and wholesale marketplaces strictly require baseline `.jpg` or `.png` uploads.
- Product galleries hide high-resolution zoom assets inside complex `<picture>` tags, multi-density `srcset` attributes, or CSS `background-image` containers.
- Manually saving and converting 20+ SKU angles one by one wastes hours every week.

---

## Why Websites Serve `.webp` and `.avif` Even When the URL Says `.jpg`

Content Delivery Networks (CDNs) inspect the browser's HTTP `Accept` request header:

```http
Accept: image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8
```

When Chrome sends `image/avif,image/webp`, the CDN performs transparent content negotiation on the edge, returning an AVIF or WebP binary payload with `Content-Type: image/webp` while keeping the original URL path (`/products/shoe-hero.jpg`). When you save the image directly from the browser tab, Chrome writes the raw WebP/AVIF stream to disk.

---

## Step-by-Step: Bulk Extracting Highest-Resolution Product Photos as Clean `.jpg` or `.png`

Using **[PixelHarvest](https://talofer.github.io/pixelharvest/)**, you can extract every high-resolution product photo on a page and transcode them into clean `.jpg` or `.png` files in a single `.zip` archive without uploading confidential catalog photography to third-party servers.

### 1. Open the Product or Collection Page
Scroll through the gallery or open the product page in Google Chrome. If the storefront uses lazy-loaded carousels, briefly scroll through the page so the DOM attaches the gallery nodes.

### 2. Launch PixelHarvest with 1 Click
Click the **PixelHarvest** icon in your Chrome toolbar. Because PixelHarvest uses Chrome Manifest V3's `activeTab` permission rather than invasive `<all_urls>` background permissions, its DOM harvester runs only when you explicitly invoke it.

During the scan, PixelHarvest automatically:
- Evaluates every `<picture>` and `img[srcset]` descriptor (`800w`, `1600w`, `2x`) and selects the **highest-resolution source**.
- Extracts CSS `background-image: url(...)` hero banners and swatches.
- Captures inline `<svg>` size charts, badges, and brand vector logos.

### 3. Filter by Dimensions & Format
Use the interactive toolbar in the popup:
- Set **Min Width (`px`)** to `300` or `500` to immediately hide tiny tracking pixels, UI icons, and payment badges.
- Filter by format pills (`All`, `WebP`, `AVIF`, `JPG`, `PNG`, `SVG`) to isolate product shots or vector graphics.

### 4. Select Output Format (`Convert All to JPG` or `Convert All to PNG`)
Choose **Convert All to JPG** (which automatically composites transparent pixels over a clean `#FFFFFF` studio-white backdrop at `0.92` quality) or **Convert All to PNG** (which preserves full alpha transparency).

### 5. Export Your Sanitized `.zip` Archive
Click **Download Selected as ZIP**. PixelHarvest decodes each asset locally in browser memory via `createImageBitmap()` and packages clean, sequentially numbered files (`pixelharvest-001-shoe-hero.jpg`, `pixelharvest-002-shoe-side.jpg`) into `pixelharvest-export.zip`.

---

## Already Have `.webp` or `.avif` Files on Your Desktop?

If you already downloaded `.webp`, `.avif`, or `.svg` files to your computer and need to convert them immediately without installing anything, use the **[PixelHarvest Free Client-Side Drag & Drop Converter](https://talofer.github.io/pixelharvest/)**. Every conversion runs 100% locally inside your browser tab via HTML5 Canvas.
