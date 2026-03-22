# Cloudflare Pages Deploy

Cloudflare Pages에 정적 웹사이트를 배포하는 스킬입니다. wrangler CLI를 사용하여 첫 배포부터 업데이트 배포까지 전 과정을 안내합니다.

## 이 스킬이 하는 일

- Cloudflare Pages 프로젝트 생성 및 첫 배포
- 코드 변경 후 업데이트 배포 (변경 파일만 자동 감지)
- API 토큰 보안 관리 (gitignore 처리)
- 포터블 Node.js 환경에서의 배포 (글로벌 설치 없이)

## 언제 사용하나요?

- "cloudflare에 배포해줘"
- "사이트 변경사항 반영해줘"
- "wrangler로 deploy 해줘"
- 코드 수정 후 라이브 사이트에 반영이 필요할 때

## 사전 준비

### 1. Cloudflare API 토큰 발급

1. [Cloudflare Dashboard](https://dash.cloudflare.com) 접속
2. My Profile → API Tokens → Create Token
3. **Cloudflare Pages** 편집 권한 포함하여 생성
4. 생성된 토큰을 안전한 곳에 보관

### 2. Node.js & wrangler 설치

**방법 A: 글로벌 설치 (권장)**
```bash
npm install -g wrangler
```

**방법 B: 포터블 설치 (Node.js 글로벌 설치가 안 되는 환경)**
```bash
# Node.js 다운로드 및 압축 해제
curl -o /tmp/node.zip https://nodejs.org/dist/v22.14.0/node-v22.14.0-win-x64.zip
unzip -o /tmp/node.zip "node-v22.14.0-win-x64/node.exe" -d /tmp/

# wrangler 로컬 설치
cd /tmp/node-v22.14.0-win-x64
./node.exe -e "require('child_process').execSync('npm install wrangler', {stdio:'inherit'})"
```

## 사용 방법

### 첫 배포

```bash
# 토큰을 gitignore 처리된 파일에 저장
echo "YOUR_TOKEN" > .cloudflare-token
echo ".cloudflare-token" >> .gitignore

# 배포 (프로젝트가 없으면 자동 생성)
CLOUDFLARE_API_TOKEN="$(cat .cloudflare-token)" \
  wrangler pages deploy . \
  --project-name=my-site \
  --branch=main
```

### 업데이트 배포

코드 수정 후 동일한 명령어 실행:
```bash
CLOUDFLARE_API_TOKEN="$(cat .cloudflare-token)" \
  wrangler pages deploy . \
  --project-name=my-site \
  --branch=main
```

wrangler가 변경된 파일만 자동 감지하여 업로드합니다. 보통 1~3초 내에 완료됩니다.

### 포터블 Node.js 환경에서 배포

```bash
# node.exe가 없으면 재압축 해제
test -f /tmp/node-v22.14.0-win-x64/node.exe || \
  unzip -o /tmp/node.zip "node-v22.14.0-win-x64/node.exe" -d /tmp/

# 배포
CLOUDFLARE_API_TOKEN="$(cat .cloudflare-token)" \
  /tmp/node-v22.14.0-win-x64/node.exe \
  /tmp/node-v22.14.0-win-x64/node_modules/wrangler/bin/wrangler.js \
  pages deploy . --project-name=my-site --branch=main
```

## 배포 결과 예시

```
⛅️ wrangler 4.75.0
Uploading... (130/130)
✨ Success! Uploaded 2 files (128 already uploaded) (1.01 sec)
🌎 Deploying...
✨ Deployment complete! Take a peek over at https://abc123.my-site.pages.dev
```

## 문제 해결

| 문제 | 해결 방법 |
|------|----------|
| 토큰 만료 | Cloudflare Dashboard에서 새 토큰 생성 |
| `node.exe` not found | `/tmp/node.zip`에서 재압축 해제 |
| 403 Forbidden | 토큰에 Pages 편집 권한이 있는지 확인 |
| 프로젝트를 찾을 수 없음 | `--project-name` 철자 확인 (첫 배포 시 자동 생성) |

## 보안 주의사항

- API 토큰은 절대 git에 커밋하지 마세요
- `.cloudflare-token` 파일은 반드시 `.gitignore`에 추가
- 토큰이 노출되었다면 즉시 Cloudflare에서 폐기하고 재발급
