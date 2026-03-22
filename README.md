# My Skills

Claude Code에서 사용할 수 있는 커스텀 스킬 모음입니다.
웹사이트 개발/배포/운영 과정에서 반복되는 작업들을 스킬로 정리했습니다.

## Skills

### 1. cloudflare-pages-deploy

Cloudflare Pages에 정적 웹사이트를 배포합니다.

- **트리거**: "배포해줘", "cloudflare에 올려줘", "deploy", "wrangler"
- **기능**:
  - 첫 배포: wrangler CLI 설치, 프로젝트 생성, 첫 배포
  - 업데이트 배포: 변경된 파일만 자동 감지하여 빠르게 배포
  - API 토큰 관리 및 보안 (gitignore 처리)
  - 포터블 Node.js 환경 지원 (Windows)

### 2. google-sheets-reservation-system

Google Sheets를 백엔드로 사용하는 예약 시스템을 구축합니다.

- **트리거**: "예약 시스템 만들어줘", "Google Sheets 예약", "booking system"
- **기능**:
  - **예약 신청 폼**: 시작/종료 시간 선택, CAPTCHA, 개인정보 동의, 비밀글/비밀번호
  - **Apps Script 웹앱**: Google Forms를 우회하여 직접 스프레드시트에 기록 (fire-and-forget)
  - **예약 게시판**: 페이지네이션, 이름 마스킹(박정훈→박*훈), 상세 모달
  - **스케줄 캘린더**: '예약완료' 상태만 필터링, 데스크탑/모바일 뷰, 룸별 색상 구분
- **참고 파일**: `references/apps-script.md`

### 3. web-image-crawler

기존 웹사이트에서 이미지를 일괄 다운로드합니다.

- **트리거**: "이미지 다운로드해줘", "사진 크롤링", "scrape images"
- **기능**:
  - Python 스크립트 기반 이미지 크롤링
  - 인코딩 문제 처리 (UTF-8 errors='ignore')
  - 기존 파일 스킵 / 강제 재다운로드 옵션
  - 멀티페이지 갤러리 크롤링

### 4. gallery-board

페이지네이션이 있는 이미지 갤러리 게시판을 구축합니다.

- **트리거**: "갤러리 만들어줘", "photo grid", "portfolio page"
- **기능**:
  - 썸네일 그리드: 3열 반응형, object-fit: cover
  - 페이지네이션: 9개씩, 이전/번호/다음
  - 상세 페이지: 전체 크기 이미지
  - gallery-data.js 데이터 구조

## 사용 방법

프로젝트의 `.claude/skills/` 디렉토리에 스킬 폴더를 복사합니다:

```bash
git clone https://github.com/hooNpk/my-skill.git

# webpage 카테고리의 스킬을 프로젝트에 복사
cp -r my-skill/webpage/cloudflare-pages-deploy your-project/.claude/skills/
cp -r my-skill/webpage/google-sheets-reservation-system your-project/.claude/skills/

# 또는 webpage 스킬 전체 복사
cp -r my-skill/webpage/*/ your-project/.claude/skills/
```

## 디렉토리 구조

```
my-skill/
├── README.md
└── webpage/                                ← 웹페이지 개발/배포 관련 스킬
    ├── cloudflare-pages-deploy/
    │   └── SKILL.md
    ├── google-sheets-reservation-system/
    │   ├── SKILL.md
    │   └── references/
    │       └── apps-script.md
    ├── web-image-crawler/
    │   └── SKILL.md
    └── gallery-board/
        └── SKILL.md
```
