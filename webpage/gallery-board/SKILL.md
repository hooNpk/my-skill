---
name: gallery-board
description: |
  Build a gallery board page with pagination, thumbnail grid, and detail views. Use this skill when the user wants to create a photo gallery, portfolio page, image board, or any grid-based media display with pagination. Covers gallery data management, thumbnail previews with object-fit, pagination UI, and detail pages with full-size images. Also triggers for "gallery page", "photo grid", "image board", or "portfolio gallery".
---

# Gallery Board

A paginated image gallery with thumbnail grid, detail page, and data-driven architecture.

## Architecture

```
gallery-data.js  ← Data source (title, date, thumbnail, detail images)
gallery.html     ← Thumbnail grid + pagination
gallery-view.html ← Detail page with full-size images
```

## 1. Gallery Data Structure

Define gallery items in a JavaScript data file:

```javascript
// gallery-data.js
const GALLERY_DATA = [
  {
    title: "브랜드 룩북 촬영",
    date: "2024-11",
    thumb: "../images/gallery/gallery_71_00.jpg",
    photos: [
      "../images/gallery/gallery_71_00.jpg",
      "../images/gallery/gallery_71_01.jpg",
      "../images/gallery/gallery_71_02.jpg",
    ]
  },
  {
    title: "제품 촬영",
    date: "2024-10",
    thumb: "../images/gallery/gallery_67_00.jpg",
    photos: [
      "../images/gallery/gallery_67_00.jpg",
      "../images/gallery/gallery_67_01.jpg",
    ]
  },
  // ... more items
];
```

Each item has:
- `title` — display name
- `date` — YYYY-MM format for sorting/display
- `thumb` — thumbnail image path (typically the first photo)
- `photos` — array of all full-size image paths for the detail page

## 2. Thumbnail Grid

### HTML Structure

```html
<div class="gallery-grid" id="galleryGrid"></div>
<div class="pagination" id="galleryPagination"></div>
```

### CSS — Responsive Grid with Cover Thumbnails

```css
.gallery-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
  padding: 20px;
}

.gallery-grid__item {
  display: block;
  text-decoration: none;
  border-radius: 8px;
  overflow: hidden;
  background: #f5f5f5;
}

.gallery-grid__item img {
  width: 100%;
  aspect-ratio: 3 / 4;
  object-fit: cover;      /* Fill container, crop excess */
  display: block;
  transition: transform 0.3s;
}

.gallery-grid__item:hover img {
  transform: scale(1.05);
}

.gallery-grid__item .gallery-grid__title {
  padding: 10px;
  font-size: 14px;
  text-align: center;
}

@media (max-width: 768px) {
  .gallery-grid { grid-template-columns: repeat(2, 1fr); gap: 10px; }
}
```

`object-fit: cover` is the key property — it fills the container while maintaining aspect ratio, cropping the overflow. This eliminates white space around images of different proportions. Use `aspect-ratio: 3/4` to keep uniform card heights.

### JavaScript — Render Grid

```javascript
const PAGE_SIZE = 9;
let currentPage = 1;

function renderPage(page) {
  currentPage = page;
  const start = (page - 1) * PAGE_SIZE;
  const pageItems = GALLERY_DATA.slice(start, start + PAGE_SIZE);

  grid.innerHTML = pageItems.map((item, i) => {
    const idx = start + i;
    return `
      <a href="gallery-view.html?id=${idx}" class="gallery-grid__item">
        <img src="${item.thumb}" alt="${item.title}" loading="lazy">
        <div class="gallery-grid__title">${item.title}</div>
      </a>`;
  }).join('');

  renderPagination();
  window.scrollTo({ top: 0, behavior: 'smooth' });
}
```

## 3. Pagination UI

```javascript
function renderPagination() {
  const total = Math.ceil(GALLERY_DATA.length / PAGE_SIZE);
  if (total <= 1) { pagination.innerHTML = ''; return; }

  let html = '';

  // Previous button
  html += currentPage > 1
    ? `<a href="#" onclick="renderPage(${currentPage - 1});return false;">◀ 이전</a>`
    : `<span class="disabled">◀ 이전</span>`;

  // Page numbers
  for (let i = 1; i <= total; i++) {
    html += i === currentPage
      ? `<span class="active">${i}</span>`
      : `<a href="#" onclick="renderPage(${i});return false;">${i}</a>`;
  }

  // Next button
  html += currentPage < total
    ? `<a href="#" onclick="renderPage(${currentPage + 1});return false;">다음 ▶</a>`
    : `<span class="disabled">다음 ▶</span>`;

  pagination.innerHTML = html;
}
```

### Pagination CSS

```css
.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 6px;
  margin: 30px 0;
  flex-wrap: wrap;
}

.pagination a,
.pagination span {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 40px;
  height: 40px;
  padding: 0 10px;
  font-size: 15px;
  border: 1px solid #ddd;
  border-radius: 6px;
  text-decoration: none;
  color: #555;
  transition: all 0.2s;
}

.pagination a:hover {
  background-color: #f5e6e8;
  border-color: #c9636a;
  color: #c9636a;
}

.pagination .active {
  background-color: #c9636a;
  color: white;
  border-color: #c9636a;
  font-weight: 700;
}

.pagination .disabled {
  color: #ccc;
  cursor: default;
  border-color: #eee;
}
```

Make buttons at least 40×40px for comfortable touch targets. Use visible text ("◀ 이전" / "다음 ▶") instead of tiny icons.

## 4. Detail Page

### HTML Structure

```html
<!-- gallery-view.html -->
<div class="gallery-detail">
  <h2 id="galleryTitle"></h2>
  <p id="galleryDate"></p>
  <div class="gallery-detail__images" id="galleryImages"></div>
  <a href="gallery.html" class="btn-back">목록으로</a>
</div>
```

### JavaScript — Load Detail

```javascript
const params = new URLSearchParams(location.search);
const id = parseInt(params.get('id'));
const item = GALLERY_DATA[id];

if (item) {
  document.getElementById('galleryTitle').textContent = item.title;
  document.getElementById('galleryDate').textContent = item.date;
  document.getElementById('galleryImages').innerHTML = item.photos.map(p =>
    `<img src="${p}" alt="${item.title}" loading="lazy">`
  ).join('');
}
```

### Detail Image CSS

```css
.gallery-detail__images img {
  width: 100%;
  max-width: 860px;
  margin: 0 auto 16px;
  display: block;
  border-radius: 8px;
}
```

Display detail images full-width (up to 860px), one per row, with the original aspect ratio preserved.

## 5. Room Pages (Variant)

For room/portfolio pages that show a fixed set of images (not a grid), use a modifier class:

```css
.gallery-grid--room {
  grid-template-columns: 1fr;   /* Single column */
  gap: 16px;
  max-width: 860px;
  margin: 0 auto;
}

.gallery-grid--room .gallery-grid__item img {
  aspect-ratio: unset;          /* Use natural aspect ratio */
  object-fit: contain;          /* Show full image, no crop */
  height: auto;
  background: #f8f5f5;
}
```

This overrides the gallery grid to show images one per row at full size, preserving the original proportions — ideal for portfolio or room showcase pages.
