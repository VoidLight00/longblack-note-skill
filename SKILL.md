---
name: longblack-note
description: 롱블랙(longblack.co) 오늘의 노트를 자동으로 Obsidian 마크다운+오디오로 저장하는 스킬. 매일 크론으로 실행되며 본문 전체, 오디오 MP3, 구조화된 노트를 생성한다. Use when setting up Longblack daily note automation, configuring the cron job, or troubleshooting Longblack note saving.
---

# Longblack Note Skill

롱블랙 구독 노트를 매일 자동으로 Obsidian 마크다운 + 오디오(MP3)로 저장한다.

## 설치 (Interactive Setup)

**처음 설치 시 아래 순서로 사용자에게 질문하여 설정을 완료한다.**

### Step 1: Obsidian Vault 경로
```
"롱블랙 노트를 저장할 Obsidian Vault 경로를 알려주세요."
"예: /Users/username/Documents/MyVault"
```
- 사용자 응답을 `VAULT_PATH`로 저장
- 노트 저장 경로: `{VAULT_PATH}/롱블랙/`
- 오디오 저장 경로: `{VAULT_PATH}/롱블랙/audio/`
- 두 폴더가 없으면 `mkdir -p`로 생성

### Step 2: 롱블랙 로그인
```
"롱블랙(longblack.co) 계정 이메일을 알려주세요."
```
- 브라우저(profile=openclaw)로 https://longblack.co 접속
- 로그인 상태 확인 → 로그인 안 되어 있으면:
  ```
  "롱블랙에 로그인이 필요합니다. 비밀번호를 알려주시면 자동 로그인합니다."
  "또는 직접 브라우저에서 로그인하셔도 됩니다."
  ```
- 로그인 완료 확인

### Step 3: 크론 스케줄
```
"매일 몇 시에 노트를 저장할까요? (기본: 07:00)"
```
- 기본값: `0 7 * * *` (Asia/Seoul)
- 사용자 타임존은 OpenClaw 설정에서 자동 감지

### Step 4: Obsidian 테마 확인
```
"Obsidian에서 사용 중인 테마가 있나요? (예: Minimal, Default 등)"
```
- Minimal 테마인 경우:
  - `{VAULT_PATH}/.obsidian/snippets/` 폴더 확인
  - `longblack-toc-fix.css` 스니펫 설치 (references/longblack-toc-fix.css 참고)
  - appearance.json의 `enabledCssSnippets`에 추가
  - accent color가 밝은색(#e0e0e0 이상)이면 경고 표시

### Step 5: 설정 저장 & 크론 등록
- 설정을 `TOOLS.md`에 기록:
  ```
  ## 📚 롱블랙 (Longblack)
  - **URL:** https://longblack.co
  - **계정:** {이메일}
  - **저장 경로:** {VAULT_PATH}/롱블랙/
  - **오디오 경로:** {VAULT_PATH}/롱블랙/audio/
  - **크론:** 매일 {시간} KST
  ```
- 크론잡 등록 (references/cron-message.md의 메시지 사용)
- 완료 메시지 출력

## 크론 실행 파이프라인

매일 크론 실행 시 아래 절차를 따른다. 상세 메시지는 `references/cron-message.md` 참고.

```
1. 중복 체크 (오늘 날짜 파일 있으면 스킵)
2. 브라우저(profile=openclaw) → longblack.co/note
3. 오늘의 노트 클릭 → 전문 읽기
4. 오디오: evaluate → audio source src URL → curl -H "Referer" 다운로드
5. 마크다운 생성 (references/template.md 구조)
6. 저장 + 보고
```

### 핵심 규칙
- **본문**: 생략 없이 전체 추출 (이미지 제외)
- **오디오**: curl에 `-H "Referer: https://longblack.co/note/{ID}"` 필수 (없으면 S3 403)
- **PDF**: 만들지 않음 (마크다운만)
- **목차**: `[[#위키링크]]` 사용 (Obsidian 내부 링크)

## 트러블슈팅

### 오디오 403 에러
S3 URL에 Referer 헤더 필요:
```bash
curl -L -H "Referer: https://longblack.co/note/{ID}" -o output.mp3 "S3_URL"
```

### 목차가 안 보임 (Minimal 테마)
`references/longblack-toc-fix.css` 스니펫 설치 → Obsidian 외관 설정에서 활성화

### 로그인 만료
브라우저(profile=openclaw)에서 longblack.co 재로그인 필요
