---
name: web-image-crawler
description: |
  Crawl and download all images from an existing website using a Python script. Use this skill when the user wants to scrape, download, or migrate images from a website, save images locally from web pages, or bulk-download photos from a gallery site. Handles encoding issues, corrupted downloads, and provides skip/force-redownload options. Also triggers when the user says "download images from this site" or "grab all photos from this URL".
---

# Web Image Crawler

Download all images from a website using Python. Handles common pitfalls: encoding issues, incomplete downloads, existing file detection, and multi-page galleries.

## Core Script Structure

```python
import requests
import os
import re
from urllib.parse import urljoin

HEADERS = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
}

def download_image(url, save_path, force=False):
    """Download a single image. Skip if exists unless force=True."""
    if os.path.exists(save_path) and os.path.getsize(save_path) > 0 and not force:
        print(f"  [SKIP] {save_path} (already exists)")
        return True

    try:
        resp = requests.get(url, headers=HEADERS, timeout=30)
        resp.raise_for_status()
        os.makedirs(os.path.dirname(save_path), exist_ok=True)

        with open(save_path, 'wb') as f:
            f.write(resp.content)

        size_kb = len(resp.content) / 1024
        print(f"  [OK] {save_path} ({size_kb:.1f} KB)")
        return True
    except Exception as e:
        print(f"  [FAIL] {url}: {e}")
        return False
```

## Handling Encoding Issues

Websites may serve HTML in various encodings (UTF-8, EUC-KR, etc.). Some pages have invalid bytes even within their declared encoding. Always use error-tolerant decoding:

```python
def fetch_page(url):
    """Fetch page HTML with encoding fallback."""
    resp = requests.get(url, headers=HEADERS, timeout=30)

    # Try UTF-8 first (most common), ignore invalid bytes
    try:
        return resp.content.decode('utf-8', errors='ignore')
    except:
        pass

    # Fallback to declared encoding
    return resp.text
```

The `errors='ignore'` parameter silently drops invalid bytes. This may lose a few characters but prevents crashes. Terminal output of Korean text may appear garbled on Windows, but the saved file content will be correct.

## Extracting Image URLs

### From `<img>` tags
```python
def extract_images(html, base_url):
    """Extract all image URLs from HTML."""
    # Match src attributes in img tags
    pattern = r'<img[^>]+src=["\']([^"\']+)["\']'
    matches = re.findall(pattern, html, re.IGNORECASE)
    return [urljoin(base_url, m) for m in matches]
```

### From gallery/detail pages (multi-level crawl)
```python
def crawl_gallery(list_url, detail_pattern):
    """Crawl a gallery: list page → detail pages → images."""
    html = fetch_page(list_url)

    # Extract detail page links
    detail_links = re.findall(detail_pattern, html)

    all_images = []
    for link in detail_links:
        detail_url = urljoin(list_url, link)
        detail_html = fetch_page(detail_url)
        images = extract_images(detail_html, detail_url)
        all_images.extend(images)

    return all_images
```

## File Naming Conventions

Organize downloaded images with clear naming:

```python
# Room images: room-{name}_{number}.jpg
save_path = f"images/rooms/room-a_{idx:02d}.jpg"

# Gallery images: gallery_{wr_id}_{idx}.jpg
save_path = f"images/gallery/gallery_{wr_id}_{idx:02d}.jpg"

# Thumbnails: gallery_{number}.jpg
save_path = f"images/gallery/gallery_{num:02d}.jpg"
```

## Handling Existing Placeholder Images

If the target directory already has placeholder/demo images, delete them before downloading real images:

```python
import glob

def clean_placeholders(directory, pattern="*.jpg"):
    """Remove existing placeholder images before re-downloading."""
    existing = glob.glob(os.path.join(directory, pattern))
    for f in existing:
        os.remove(f)
        print(f"  [DEL] {f}")
```

This prevents the skip-existing logic from keeping old placeholder files instead of downloading real ones.

## Complete Example: Gallery Crawler

```python
import requests, os, re
from urllib.parse import urljoin

HEADERS = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)'}
BASE_URL = "http://example.com"
SAVE_DIR = "images/gallery"

def main():
    os.makedirs(SAVE_DIR, exist_ok=True)

    # Crawl multiple pages
    for page in range(1, 3):  # Pages 1 and 2
        url = f"{BASE_URL}/gallery?page={page}"
        html = requests.get(url, headers=HEADERS).content.decode('utf-8', 'ignore')

        # Extract detail page links
        links = re.findall(r'href="(/gallery/view/(\d+))"', html)

        for href, wr_id in links:
            detail_url = urljoin(BASE_URL, href)
            detail_html = requests.get(detail_url, headers=HEADERS).content.decode('utf-8', 'ignore')

            # Extract title
            title_match = re.search(r'<title>([^<]+)</title>', detail_html)
            title = title_match.group(1).split(' > ')[-1].strip() if title_match else f"gallery_{wr_id}"

            # Extract images
            images = re.findall(r'<img[^>]+src=["\']([^"\']+)["\']', detail_html)
            images = [urljoin(detail_url, img) for img in images if 'data/editor' in img]

            for idx, img_url in enumerate(images):
                save_path = os.path.join(SAVE_DIR, f"gallery_{wr_id}_{idx:02d}.jpg")
                if os.path.exists(save_path):
                    continue
                try:
                    data = requests.get(img_url, headers=HEADERS, timeout=30).content
                    with open(save_path, 'wb') as f:
                        f.write(data)
                    print(f"  [OK] {save_path}")
                except Exception as e:
                    print(f"  [FAIL] {img_url}: {e}")

if __name__ == '__main__':
    main()
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `UnicodeDecodeError` | Use `decode('utf-8', errors='ignore')` |
| Images download as 0 bytes | Check if site requires cookies or referrer header |
| 403 Forbidden | Add proper `User-Agent` and `Referer` headers |
| Terminal shows garbled Korean | Normal on Windows — file content is correct |
| Existing files not replaced | Delete old files first or use `force=True` |
| Images appear corrupted | Write in binary mode (`'wb'`), don't decode image data |

## Platform Notes (Windows)

- Use `python` not `python3` (python3 may map to Windows Store stub)
- Save scripts to the project directory, not `/tmp/` (Windows Python may not find `/tmp/`)
- File paths use forward slashes in Python (`images/gallery/`) — works on Windows
