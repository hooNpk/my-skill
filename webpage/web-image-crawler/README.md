# Web Image Crawler

기존 웹사이트에서 이미지를 일괄 다운로드하는 Python 스크립트를 생성하는 스킬입니다. 사이트 마이그레이션, 갤러리 이미지 수집, 백업 등에 활용합니다.

## 이 스킬이 하는 일

- 웹페이지에서 모든 이미지 URL을 추출하여 다운로드
- 갤러리 목록 → 상세 페이지 → 이미지의 멀티레벨 크롤링
- 인코딩 문제 (UTF-8, EUC-KR) 자동 처리
- 이미지 깨짐 없이 바이너리 모드로 저장
- 기존 파일 스킵 / 강제 재다운로드 옵션

## 언제 사용하나요?

- "이 사이트에서 이미지 전부 다운로드해줘"
- "기존 홈페이지 갤러리 사진을 로컬로 받아줘"
- "웹사이트 이미지를 마이그레이션 해야 해"
- 특정 사이트의 룸 사진, 갤러리 사진 등을 일괄 수집할 때

## 사용 방법

### 기본 구조

스크립트를 프로젝트 디렉토리에 생성하고 실행합니다:

```bash
python download_images.py
```

### 핵심 함수들

**1. 페이지 가져오기 (인코딩 처리)**

```python
def fetch_page(url):
    resp = requests.get(url, headers=HEADERS, timeout=30)
    return resp.content.decode('utf-8', errors='ignore')
```

`errors='ignore'`를 사용하면 잘못된 바이트가 있어도 에러 없이 진행됩니다. 터미널에서 한글이 깨져 보일 수 있지만, 파일에 저장된 내용은 정상입니다.

**2. 이미지 다운로드 (바이너리 모드)**

```python
def download_image(url, save_path, force=False):
    if os.path.exists(save_path) and os.path.getsize(save_path) > 0 and not force:
        print(f"  [SKIP] {save_path}")
        return

    resp = requests.get(url, headers=HEADERS, timeout=30)
    with open(save_path, 'wb') as f:   # 반드시 'wb' (바이너리 쓰기)
        f.write(resp.content)           # .content (bytes), .text (str) 아님
```

이미지는 반드시 바이너리 모드(`'wb'`)로 저장해야 합니다. `.text` 대신 `.content`를 사용합니다.

**3. 멀티페이지 갤러리 크롤링**

```python
# 1단계: 목록 페이지에서 상세 페이지 링크 추출
for page in range(1, 3):
    html = fetch_page(f"{BASE_URL}/gallery?page={page}")
    links = re.findall(r'href="(/gallery/view/(\d+))"', html)

    # 2단계: 각 상세 페이지에서 이미지 URL 추출
    for href, wr_id in links:
        detail_html = fetch_page(urljoin(BASE_URL, href))
        images = re.findall(r'src="([^"]+)"', detail_html)

        # 3단계: 이미지 다운로드
        for idx, img_url in enumerate(images):
            download_image(img_url, f"images/gallery_{wr_id}_{idx:02d}.jpg")
```

### 기존 파일 처리

**스킵 모드 (기본):** 이미 존재하는 파일은 건너뜁니다.

**강제 재다운로드:** 플레이스홀더 이미지를 실제 이미지로 교체할 때:

```python
# 기존 파일 삭제 후 다운로드
import glob
for f in glob.glob("images/gallery/gallery_*.jpg"):
    os.remove(f)
# 이후 다운로드 실행
```

### 파일 명명 규칙

용도에 따라 일관된 이름을 사용합니다:

```
images/
├── rooms/
│   ├── room-a_01.jpg, room-a_02.jpg, room-a_03.jpg
│   ├── room-b_01.jpg, room-b_02.jpg, room-b_03.jpg
│   └── room-c_01.jpg, room-c_02.jpg
└── gallery/
    ├── gallery_71_00.jpg, gallery_71_01.jpg   (wr_id=71의 이미지들)
    ├── gallery_67_00.jpg
    └── ...
```

## 자주 발생하는 문제

| 문제 | 원인 | 해결 |
|------|------|------|
| `UnicodeDecodeError` | 페이지 인코딩 불일치 | `decode('utf-8', errors='ignore')` 사용 |
| 0바이트 이미지 | 사이트가 쿠키/리퍼러 요구 | `Referer` 헤더 추가 |
| 403 Forbidden | User-Agent 차단 | 브라우저 User-Agent 사용 |
| 터미널 한글 깨짐 | Windows 터미널 인코딩 | 정상 — 파일 내용은 올바름 |
| 기존 파일이 교체 안 됨 | 스킵 로직 동작 | 기존 파일 삭제 후 재실행 |
| 이미지 깨짐 | 텍스트 모드로 저장 | `'wb'` 모드 + `.content` 사용 |
| `python3` 실행 안 됨 | Windows Store 스텁 | `python` 명령어 사용 |

## 필수 헤더

사이트 차단을 피하기 위해 User-Agent를 설정합니다:

```python
HEADERS = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
}
```

일부 사이트는 `Referer` 헤더도 필요합니다:

```python
HEADERS = {
    'User-Agent': 'Mozilla/5.0 ...',
    'Referer': 'http://example.com/'
}
```

## Windows 참고사항

- `python3` 대신 `python` 사용 (`python3`은 Windows Store 스텁일 수 있음)
- 스크립트를 `/tmp/`가 아닌 프로젝트 디렉토리에 저장
- 파일 경로에 forward slash (`/`) 사용 가능 (Python이 자동 변환)
