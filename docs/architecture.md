🏗️ Photo Gallery Automation — Architecture Documentation
1. Overview

A lightweight, automated photo publishing pipeline.
You keep a local folder with your camera photos; a background script automatically resizes, optimizes, and uploads them to Cloudflare R2.
Your Astro site reads a JSON manifest from R2 to display the gallery dynamically — no redeploys required.

2. Goals

Instant publishing: drag a photo → it appears live.

Minimal cost: Cloudflare R2 = free egress, cheap storage.

Privacy: strip GPS, keep only useful metadata.

Performance: small thumbnails, cached full images.

Simplicity: static Astro frontend + small Node watcher.

3. System Architecture
[You drop photo]
        │
        ▼
  [Local Watcher Script]
        │
        ├─► Resize + optimize
        ├─► Extract EXIF (no GPS)
        ├─► Upload full + thumb → R2
        └─► Update manifest.json (atomic)
                │
                ▼
      [Cloudflare R2 Storage]
                │
                ▼
         [Astro Static Site]
                │
                ▼
       User sees new photo live

4. Components
4.1 Local Folder

~/Documents/personal/photo-gallery-automation/originals/

You drop camera photos here.

Source files stay private (RAW or large JPEG).

4.2 Watcher Script

Monitors the folder for new images.

Generates:

Thumbs (480 px) — small previews, no EXIF.

Full (1600 px) — optimized display size, EXIF kept (minus GPS).

Uploads both to R2 and regenerates manifest.json.

Writes the new manifest atomically (temporary + rename).

Optionally calls Cloudflare API to purge cache for instant updates.

4.3 Manifest

A JSON file stored in R2 that acts as a lightweight “database.”

Contains metadata and public URLs for all photos.

Example:

{
  "items": [
    {
      "id": "forest-path",
      "title": "Forest Path",
      "thumb": "https://img.yourname.com/thumb/forest-path.jpg",
      "full": "https://img.yourname.com/full/forest-path.jpg",
      "exif": {
        "Camera": "Fujifilm X-T5",
        "Lens": "23mm f/2",
        "ISO": 200,
        "Exposure": "1/125s",
        "Date": "2024-03-15"
      }
    }
  ]
}

4.4 Cloudflare R2

Stores all processed photos and the manifest.

Public read access (CDN-cached).

Cache policy:

Images: public, max-age=31536000, immutable

Manifest: no-cache or max-age=60

4.5 Astro Frontend

Static site (e.g., on Cloudflare Pages).

Fetches live manifest.json from R2.

Displays thumbnails in a responsive grid.

Opens full-size image in a lightbox.

Optionally shows EXIF info overlay.

5. Data & Metadata Flow
Stage	Action	Metadata	Storage
Original	Photo from camera	Full EXIF	Local only
Processing	Resize + extract	EXIF read, GPS removed	Temp local
Output (Full)	1600 px JPEG	EXIF copied (no GPS)	R2 /full/
Output (Thumb)	480 px JPEG	None	R2 /thumb/
Manifest	Write/update JSON	Camera, Lens, Date, etc.	R2 root
6. Privacy Strategy

GPS removed before uploading.

Originals never leave your machine.

EXIF fields kept: camera, lens, ISO, aperture, date.

Optional: store EXIF only in manifest (not embedded).

7. Deployment Flow
Step	Description
1	Create Cloudflare R2 bucket (photos-site)
2	Configure public read + versioning
3	Deploy Astro site (Cloudflare Pages / Vercel)
4	Site fetches manifest from https://img.yourname.com/manifest.json
5	Drop new photo locally → watcher uploads → manifest updates
6	New photo visible live within seconds
8. Future Extensions
Feature	Description
🧠 AI Captions	Auto-generate titles or tags using OpenAI API
🔍 Search	Filter by camera, lens, tag, or date
🗺️ Map View	Optional GPS plotting
🔒 Private Albums	Signed URLs or access keys
📱 Mobile Upload	Sync from phone to /originals/ via cloud folder
9. Performance Notes

Immutable URLs: image caching is effortless.

Manifest caching: light and fast; only this changes.

Responsive design: use <img srcset> and lazy loading.

Build-free updates: no need to redeploy Astro.

10. Cost Estimate (Cloudflare R2)
Resource	Usage	Approx. Cost
Storage	~250 MB	<$0.01/month
Egress	Unlimited	$0
Requests	Low	Free tier covers it
11. Security Considerations

Use R2 access keys in a local .env, never commit them.

Enable object versioning (rollback protection).

Optional: restrict write access via scoped API token.

12. Summary

💾 Local Folder: you drop photos.

⚙️ Watcher: resizes, extracts EXIF, uploads to R2.

☁️ Cloudflare R2: serves optimized images + manifest.

🌐 Astro Site: renders live gallery, no rebuilds.

🔒 Privacy: EXIF cleaned, GPS stripped.

💸 Cost: practically free.