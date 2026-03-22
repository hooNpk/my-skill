# Gallery Board

페이지네이션이 있는 이미지 갤러리 게시판을 구축하는 스킬입니다. 썸네일 그리드, 상세 페이지, 데이터 기반 갤러리 관리를 포함합니다.

## 이 스킬이 하는 일

- 반응형 썸네일 그리드 (3열 → 2열 모바일 적응)
- 페이지네이션 (9개씩, 이전/번호/다음 네비게이션)
- 상세 페이지 (전체 크기 이미지 목록)
- JavaScript 데이터 파일로 갤러리 항목 관리
- 룸/포트폴리오 페이지 변형 (1열 풀사이즈)

## 언제 사용하나요?

- "갤러리 페이지 만들어줘"
- "사진 게시판을 만들고 싶어"
- "포트폴리오 페이지를 만들어줘"
- "이미지를 그리드로 보여주고 클릭하면 상세 보기"

## 파일 구조

```
├── gallery-data.js       ← 갤러리 데이터 (제목, 날짜, 이미지 경로)
├── pages/
│   ├── gallery.html      ← 썸네일 그리드 + 페이지네이션
│   └── gallery-view.html ← 상세 페이지
├── css/
│   └── style.css         ← 그리드, 페이지네이션, 상세 스타일
└── images/
    └── gallery/          ← 갤러리 이미지 파일들
```

## 사용 방법

### 1. 갤러리 데이터 정의

`gallery-data.js`에 갤러리 항목을 배열로 정의합니다:

```javascript
const GALLERY_DATA = [
  {
    title: "브랜드 룩북 촬영",          // 표시 제목
    date: "2024-11",                    // 날짜 (정렬/표시용)
    thumb: "../images/gallery/img_01.jpg",  // 썸네일 이미지
    photos: [                           // 상세 페이지 이미지들
      "../images/gallery/img_01.jpg",
      "../images/gallery/img_02.jpg",
      "../images/gallery/img_03.jpg",
    ]
  },
  // ... 더 많은 항목
];
```

- `thumb`: 보통 `photos` 배열의 첫 번째 이미지를 사용
- `photos`: 상세 페이지에서 보여줄 모든 이미지
- 최신 항목을 배열 앞에 배치하면 최신순 표시

### 2. 썸네일 그리드

**핵심 CSS — `object-fit: cover`:**

```css
.gallery-grid__item img {
  width: 100%;
  aspect-ratio: 3 / 4;
  object-fit: cover;
}
```

- `object-fit: cover` — 컨테이너를 꽉 채우고 넘치는 부분은 잘라냄
- `aspect-ratio: 3/4` — 모든 카드의 높이를 균일하게 유지
- 다양한 비율의 이미지도 흰 여백 없이 깔끔하게 표시

**반응형 그리드:**

```css
.gallery-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);  /* 데스크탑: 3열 */
  gap: 16px;
}

@media (max-width: 768px) {
  .gallery-grid {
    grid-template-columns: repeat(2, 1fr);  /* 모바일: 2열 */
  }
}
```

### 3. 페이지네이션

```javascript
const PAGE_SIZE = 9;
let currentPage = 1;

function renderPage(page) {
  currentPage = page;
  const start = (page - 1) * PAGE_SIZE;
  const pageItems = GALLERY_DATA.slice(start, start + PAGE_SIZE);

  // 썸네일 렌더링
  grid.innerHTML = pageItems.map((item, i) => `
    <a href="gallery-view.html?id=${start + i}" class="gallery-grid__item">
      <img src="${item.thumb}" alt="${item.title}" loading="lazy">
      <div class="gallery-grid__title">${item.title}</div>
    </a>
  `).join('');

  // 페이지네이션 컨트롤 렌더링
  renderPagination();

  // 페이지 상단으로 스크롤
  window.scrollTo({ top: 0, behavior: 'smooth' });
}
```

**페이지네이션 UI 가이드:**

- 버튼 최소 크기: 40×40px (모바일 터치 대응)
- 텍스트 사용: "◀ 이전" / "다음 ▶" (아이콘만 사용하면 너무 작음)
- 현재 페이지: 강조 색상 배경 + 볼드
- 비활성 버튼: 회색 처리

### 4. 상세 페이지

URL 파라미터로 갤러리 항목 ID를 전달합니다:

```
gallery-view.html?id=5
```

```javascript
const id = parseInt(new URLSearchParams(location.search).get('id'));
const item = GALLERY_DATA[id];

document.getElementById('galleryTitle').textContent = item.title;
document.getElementById('galleryImages').innerHTML = item.photos.map(p =>
  `<img src="${p}" alt="${item.title}" loading="lazy">`
).join('');
```

상세 이미지는 한 줄에 하나씩, 원본 비율 유지:

```css
.gallery-detail__images img {
  width: 100%;
  max-width: 860px;
  margin: 0 auto 16px;
  display: block;
}
```

### 5. 룸/포트폴리오 변형

고정된 이미지 세트를 1열로 보여주는 레이아웃:

```html
<div class="gallery-grid gallery-grid--room">
  <div class="gallery-grid__item">
    <img src="../images/rooms/room-a_01.jpg" alt="A Room">
  </div>
  <!-- ... -->
</div>
```

```css
.gallery-grid--room {
  grid-template-columns: 1fr;    /* 1열 */
  max-width: 860px;
  margin: 0 auto;
}

.gallery-grid--room .gallery-grid__item img {
  aspect-ratio: unset;           /* 원본 비율 유지 */
  object-fit: contain;           /* 전체 이미지 표시, 잘라내지 않음 */
  height: auto;
}
```

- 갤러리 (썸네일): `object-fit: cover` + `aspect-ratio: 3/4` → 균일한 그리드
- 룸/포트폴리오: `object-fit: contain` + `aspect-ratio: unset` → 원본 비율

## object-fit 정리

| 값 | 동작 | 용도 |
|----|------|------|
| `cover` | 컨테이너 채움, 넘치면 자름 | 썸네일 그리드 (균일한 카드) |
| `contain` | 전체 이미지 표시, 여백 발생 가능 | 상세/룸 페이지 (원본 보존) |
| `fill` | 비율 무시하고 채움 | 거의 사용 안 함 |

## 커스터마이징

| 설정 | 위치 | 기본값 |
|------|------|--------|
| 페이지당 항목 수 | `PAGE_SIZE` | 9 |
| 그리드 열 수 | `.gallery-grid { grid-template-columns }` | 3열 |
| 카드 비율 | `.gallery-grid__item img { aspect-ratio }` | 3/4 |
| 상세 이미지 최대 폭 | `.gallery-detail__images img { max-width }` | 860px |
| 모바일 전환점 | `@media (max-width: ...)` | 768px |
