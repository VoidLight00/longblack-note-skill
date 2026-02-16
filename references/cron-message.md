# 크론잡 메시지 템플릿

아래 메시지를 크론잡 payload로 등록한다.
`{SAVE_PATH}`, `{AUDIO_PATH}`는 설치 시 설정한 경로로 치환.

```
롱블랙 오늘의 노트를 마크다운으로 저장해줘. 아래 절차를 따라:

## 절차
1. 브라우저(profile=openclaw)로 https://longblack.co/note 접속
2. 오늘 날짜의 최신 노트 클릭
3. 노트 전문을 읽고 마크다운 템플릿에 맞춰 정리
4. 오디오: evaluate로 페이지에서 audio source의 src URL 추출 → curl -L -H "Referer: https://longblack.co/note/{노트ID}" -o 파일경로 "S3_URL" 로 다운로드 → audio/ 폴더 저장. 실패시 "오디오 추출 실패" 표시
5. 마크다운 파일 저장 (PDF 만들지 않음!)

## 경로
- MD: {SAVE_PATH}/YYYY-MM-DD-제목요약.md
- Audio: {AUDIO_PATH}/YYYY-MM-DD-제목요약.mp3

## 중복 체크
ls {SAVE_PATH} | grep $TODAY → 있으면 '이미 저장됨' 보고 후 종료

## 마크다운 템플릿
references/template.md 구조를 정확히 따를 것.

## 주의사항
- PDF 만들지 않음
- 본문 생략 없이 전문 포함
- 오디오 curl에 반드시 -H "Referer: https://longblack.co/note/{ID}" 포함
- 완료 후 제목, 저자, 핵심문장 1개, 오디오 상태만 간단 보고
```

## 크론 등록 파라미터

```json
{
  "name": "롱블랙 오늘의 노트 저장",
  "schedule": { "kind": "cron", "expr": "0 {HOUR} * * *", "tz": "{TIMEZONE}" },
  "sessionTarget": "isolated",
  "payload": { "kind": "agentTurn", "message": "(위 메시지)", "timeoutSeconds": 300 },
  "delivery": { "mode": "announce", "bestEffort": true }
}
```
