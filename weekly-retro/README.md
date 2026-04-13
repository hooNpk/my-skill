# weekly-retro

음성 녹음 파일을 STT로 변환하고 Claude가 주간 회고 마크다운으로 정리하여 Obsidian 폴더에 저장하는 스킬입니다.

## 트리거

`/weekly-retro` 또는 "주간 회고 작성해줘"

## 사전 요구사항

- Python 환경 및 STT 스크립트: `C:/project/weekly_retro/stt.py`
- 설정 파일: `C:/project/weekly_retro/config.json`
  ```json
  {
    "obsidian_retro_folder": "경로/to/obsidian/회고"
  }
  ```

## 사용 방법

```
/weekly-retro <오디오_파일_경로>
```

예시:
```
/weekly-retro C:/recordings/2026-W15.m4a
```

## 동작 순서

1. **STT 실행** — `stt.py`로 음성을 텍스트로 변환
2. **오류 교정** — 알려진 STT 오류 목록을 기반으로 텍스트 교정
3. **회고 마크다운 생성** — 아래 섹션 구조로 작성
   - Weekly Summary
   - Activity Log (회사/사이드 프로젝트별)
   - Networking & Insights
   - 운동·건강 점검
   - 원문 트랜스크립트 요약
4. **Obsidian 저장** — `config.json`의 경로에 `YYYY-W{주차}.md`로 저장

## 출력 파일명 규칙

`YYYY-W{주차}.md` — 오늘 날짜 기준 직전 주의 ISO 주차 사용

예: 2026-04-13(월) 실행 → `2026-W15.md`

## 설치

```bash
cp -r my-skill/weekly-retro/ your-project/.claude/skills/
```
