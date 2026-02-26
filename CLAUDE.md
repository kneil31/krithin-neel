# CLAUDE.md — Krithin Neel Family Memories

Subproject of rsquare-studios-dashboard. Family memories page hosted on GitHub Pages.

**Last Updated:** 2026-02-25

## IMPORTANT: No Secrets in This File

**This file is on a PUBLIC repo.** Never put passwords, phone numbers, emails, SmugMug node/image IDs, YouTube video IDs, file paths, or any personal information here. All secrets and identifiers live in gitignored local files (`.secret`, `cover_images.json`) or in the generator source.

## Overview

- **Live URL:** https://krithin.rsquarestudios.com/ (custom domain, CNAME → kneil31.github.io)
- **GitHub Repo:** kneil31/krithin-neel (public)
- **Generator:** `generate_krithin_page.py` → `output/index.html`
- **Password:** Read from `KRITHIN_PAGE_PASSWORD` env var or `.secret` file (not hardcoded)
- **Encryption:** AES-256-GCM, data-driven DOM (no innerHTML)

## Design

- Warm pastel baby/family theme (cream #FFF8F0 bg, peach #E8A87C accent)
- System font stack (no Google Fonts — zero 3rd-party calls)
- 3 tabs: Krithin | Reels | Monika. OTP users only see Krithin + Reels (Monika tab hidden)
- Mobile-first (shared via WhatsApp)
- WhatsApp number stored in encrypted blob (loaded after decryption)
- Gallery tiles: 3/4 aspect ratio, 2-column mobile, 3-column desktop
- Video tiles: 16/9 aspect ratio, always 2-column grid. Supports optional `"cover"` field
- Reel tiles: 9/16 portrait aspect ratio, 3-column grid, `<img>` tags. Supports optional `"cover"` field
- Gallery tiles: Support optional `"cover"` field to override `cover_images.json` highlight images

## Data

- Image counts from `../album_stats.json` (keyed by SmugMug node_id)
- Cover images from `cover_images.json` (SmugMug highlight images, medium/large/xlarge)
- Gallery URLs, video URLs, and metadata hardcoded in generator
- Custom covers: galleries, videos, and reels all support optional `"cover"` field (SmugMug XLarge URLs)

## Content

Galleries, videos, and reels are organized into 3 tabs. See `generate_krithin_page.py` for the full list of galleries, videos, and SmugMug node IDs.

- Video/reel/gallery tiles all support optional `"cover": "SmugMug URL"` field — use XLarge size for good quality

## Security

- **Encryption:** AES-256-GCM at build time, PBKDF2 key derivation
- **Password:** From env var or `.secret` file (never hardcoded)
- **Data-driven DOM:** JS builds DOM with `createElement`/`textContent` (no `innerHTML`)
- **URL allowlist:** JS validates all URLs against known hosts before setting `href`/`src`
- **Referrer protection:** `<meta name="referrer" content="no-referrer">` + `rel="noreferrer noopener"`
- **No 3rd-party calls:** System fonts, no external JS/CSS
- **Privacy:** `<meta name="robots" content="noindex, nofollow, noarchive">`
- **CSP:** Strict Content-Security-Policy via `<meta>` tag with nonce-based `script-src`
- **No inline onclick:** All event handlers use `addEventListener`
- **Permissions-Policy:** Camera, mic, geolocation disabled
- **3-strike lockout** with cooldown on wrong password
- **OTP:** Dual-blob encryption — master password always works + OTP expires after 48h. OTP blob excludes some content entirely.
- **Session persistence:** 30-min auto-unlock (survives external site visits)
- **Schema versioned** encrypted payload

## Workflow

```bash
# Regenerate (password from .secret file or env var)
cd krithin-neel
python3 generate_krithin_page.py

# Deploy to GitHub Pages
cp output/index.html /tmp/krithin-neel/
cd /tmp/krithin-neel && git add . && git commit -m "description" && git push

# Generate new OTP + deploy + Slack notify (all-in-one)
python3 generate_otp.py              # Regenerate + deploy + Slack notify
python3 generate_otp.py --no-push    # Regenerate only (no deploy, no Slack)
python3 generate_otp.py --no-slack   # Regenerate + deploy (no Slack)
```

## Notes

- Album names, node IDs, image keys, and cover mappings are in `generate_krithin_page.py` and `cover_images.json` — not documented here (public repo)
- Upload script (`upload_moniel_wedding.py`) filters macOS `._` resource fork files, uses `AutoRename: True` and `SortMethod: Name`
- OG image hosted on SmugMug (XLarge size). Local `og-image.jpg` was scrubbed from repo history via `filter-repo`.
- `generate_otp.py` — wrapper script: regenerates page (new OTP), copies to deploy dir, pushes, sends OTP to Slack
