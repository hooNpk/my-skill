# hand-data / clip-video

MD 파일에 정의된 시간대 기준으로 MP4 영상을 소리 없이 잘라 `clipped_video` 폴더에 저장하는 스킬입니다.

## 트리거

`/clip-video` 또는 "영상 잘라줘", "클립 만들어줘"

## 사전 요구사항

- ffmpeg 실행 파일 (`ffmpeg.exe`)
  - 기본 탐색 경로: 비디오 폴더의 상위 폴더
  - 없으면 경로를 직접 인수로 전달

## MD 파일 형식

MP4 파일과 같은 이름의 `.md` 파일에 클립 정보를 작성합니다.

```
- {시작시간} ~ {종료시간}, {출력파일이름}
```

예시 (`20260405_151828.md`):
```
- 00:00 ~ 00:34, 20260405_folding_socks_1
- 00:34 ~ 01:09, 20260405_folding_socks_2
- 01:56 ~ 02:27, 20260405_folding_socks_3
```

- 시간 형식: `MM:SS` 또는 `HH:MM:SS` 모두 지원
- 출력 파일명에 확장자 생략 가능 (`.mp4` 자동 추가)

## 사용 방법

```
/clip-video <비디오_폴더_경로>
```

예시:
```
/clip-video C:/project/hand-data/video
```

## 동작 순서

1. 비디오 폴더에서 `.md` 파일 목록 수집
2. 같은 이름의 `.mp4`가 없는 MD는 건너뜀
3. MD 파싱 → 시작/종료 시간 및 출력 파일명 추출
4. ffmpeg로 클립 생성 (`-an` 무음, `-c:v copy` 고속 복사)
5. 결과 보고

## 출력 구조

```
{비디오폴더 상위}/
└── clipped_video/
    ├── {파일이름1}.mp4
    ├── {파일이름2}.mp4
    └── ...
```

## 설치

```bash
cp -r my-skill/hand-data/ your-project/.claude/skills/clip-video/
```
