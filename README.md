# Photo Gallery Automation

🏗️ Architecture Overview

Photo Gallery Automation is a minimal pipeline that turns your local photo folder into a live, auto-updating web gallery.

You drop a photo → it’s resized, optimized, uploaded to Cloudflare R2, and appears instantly on your Astro site — no manual deploys.

[You drop photo]
       │
       ▼
 Local Watcher Script
 (resize + extract EXIF)
       │
       ▼
 Uploads → Cloudflare R2
 (thumb + full + manifest.json)
       │
       ▼
   Astro Site (static)
 (fetches manifest.json from R2)
       │
       ▼
  Live Photo Gallery 🌍

⚙️ Components
Component	Purpose
Local Folder	Where you drop new camera photos.
Watcher Script	Detects new photos, resizes, extracts EXIF, uploads to R2.
Cloudflare R2	Stores processed photos and a manifest.json file.
Astro Frontend	Static site that reads the manifest and displays the gallery.
🔒 Privacy & Performance

Strips GPS but keeps camera/lens/date metadata.

Thumbnails (≈480 px) load fast; full-size (≈1600 px) only when clicked.

R2 serves images via CDN with zero egress fees.

The manifest refreshes automatically—no rebuilds needed.

🌐 Live Workflow

Add photo → watcher processes it.

Image + manifest uploaded to Cloudflare R2.

Astro site fetches updated manifest.

New photo visible globally within seconds.

💡 Detailed technical architecture available in docs/ARCHITECTURE.md