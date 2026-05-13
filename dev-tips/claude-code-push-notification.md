# Claude Code 작업 완료 푸시 알림 설정 가이드

Claude Code의 **Hooks** 기능을 활용해 작업이 끝나면 핸드폰으로 알림을 받는 방법.

## 동작 원리

```
Claude Code 작업 완료 → Stop hook 발화 → curl로 푸시 서비스 호출 → 폰 알림
```

Hook은 Claude Code 라이프사이클의 특정 시점에 실행되는 셸 명령어. 별도 데몬·백그라운드 프로세스 없이 동작.

---

## 추천 방식: ntfy.sh

- **무료**, 계정 생성 불필요
- 토픽(topic) 이름만 정하면 즉시 사용 가능
- 설정 5분, 외부 의존성 최소

### 1단계 — 폰에 ntfy 앱 설치 및 토픽 구독

- Play Store / App Store에서 **"ntfy"** 검색 후 설치
- 앱 실행 → `+` 버튼 → 토픽 이름 입력 후 구독
- 예시: `weppy-claude-9x7k2m`

> ⚠️ **주의:** ntfy.sh 공개 토픽은 누구나 이름을 알면 구독 가능. 추측이 어려운 무작위 문자열을 포함시킬 것.

### 2단계 — `~/.claude/settings.json`에 Stop hook 등록

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "curl -s -H 'Title: Claude Code 작업 완료' -H 'Tags: white_check_mark' -d \"$(basename $(pwd)) 작업이 끝났습니다\" https://ntfy.sh/weppy-claude-9x7k2m"
          }
        ]
      }
    ]
  }
}
```

- 토픽 URL 부분만 본인 토픽으로 변경
- `$(basename $(pwd))`는 현재 디렉토리명을 메시지에 포함 → 폰에서 `cloud-wms-be`, `class-checkin-be` 등 어떤 프로젝트인지 즉시 식별
- `Tags: white_check_mark`는 알림 아이콘으로 ✅ 표시

### 3단계 — 동작 확인

수동 테스트:

```bash
curl -d "테스트 알림" https://ntfy.sh/weppy-claude-9x7k2m
```

폰에 알림이 즉시 뜨면 성공. 이후 `claude` 실행하고 임의 작업을 시킨 뒤 종료 시점에 알림 수신 확인.

---

## Hook 이벤트 선택

| Hook 이벤트 | 발화 시점 | 용도 |
|---|---|---|
| **Stop** | Claude의 턴이 종료될 때 (응답 완료) | "작업 끝났음" 알림 |
| **Notification** | 권한 요청·입력 대기 등 사용자 개입 필요 시 | "내 입력 필요" 알림 |

- "지시한 작업이 끝나면 알림" 목적은 **Stop**이 적합
- 단, Stop은 **매 턴마다** 발화 → 짧은 대화형 질문에도 알림이 와서 시끄러울 수 있음

### 권장 패턴 1: 자율 실행 시에만 알림 활성화

`--dangerously-skip-permissions` 모드로 장시간 작업을 돌릴 때만 알림을 받도록 환경변수로 토글.

```json
"command": "if [ \"$NOTIFY\" = \"1\" ]; then curl -s -d \"$(basename $(pwd)) 완료\" https://ntfy.sh/weppy-claude-9x7k2m; fi"
```

실행 방법:

```bash
# 알림 받기
NOTIFY=1 claude --dangerously-skip-permissions

# 알림 없이 일반 사용
claude
```

### 권장 패턴 2: Notification hook 병행

입력 대기 상태에도 알림이 와야 하면 두 이벤트 모두 등록.

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          { "type": "command", "command": "curl -s -d \"$(basename $(pwd)) 완료\" https://ntfy.sh/weppy-claude-9x7k2m" }
        ]
      }
    ],
    "Notification": [
      {
        "hooks": [
          { "type": "command", "command": "curl -s -H 'Priority: high' -d \"$(basename $(pwd)) 입력 대기 중\" https://ntfy.sh/weppy-claude-9x7k2m" }
        ]
      }
    ]
  }
}
```

---

## 대안 비교

| 서비스 | 비용 | 장점 | 단점 |
|---|---|---|---|
| **ntfy.sh** | 무료 | 계정 불필요, 즉시 시작 | 공개 토픽은 보안 약함 (셀프호스팅 가능) |
| **Pushover** | 일회성 $5 | 안정적, 우선순위·소리 커스터마이징 풍부 | 유료 |
| **Telegram Bot** | 무료 | 이미 텔레그램 쓰면 편함, 이력 보존 | 봇 생성·chat_id 발급 절차 필요 |
| **Slack Webhook** | 무료 (워크스페이스 보유 시) | 회사 워크플로우와 통합 가능 | 워크스페이스·채널 권한 필요 |

### Pushover 예시

```bash
curl -s \
  --form-string "token=$PUSHOVER_APP_TOKEN" \
  --form-string "user=$PUSHOVER_USER_KEY" \
  --form-string "message=$(basename $(pwd)) 작업 완료" \
  https://api.pushover.net/1/messages.json
```

### Telegram Bot 예시

```bash
curl -s -X POST \
  "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage" \
  -d "chat_id=$TELEGRAM_CHAT_ID" \
  -d "text=$(basename $(pwd)) 작업 완료"
```

### Slack Incoming Webhook 예시

```bash
curl -s -X POST \
  -H 'Content-Type: application/json' \
  -d "{\"text\":\"$(basename $(pwd)) 작업 완료\"}" \
  $SLACK_WEBHOOK_URL
```

---

## 운영 팁

1. **시작은 ntfy로** — 토픽만 정하면 끝나므로 5분 안에 검증 가능
2. **익숙해지면 Pushover 또는 Telegram으로 이전** — 회사 환경에서 안정성·이력 관리 측면 우위
3. **민감 정보 알림 금지** — ntfy 공개 토픽은 평문 전송. 코드·인증정보·고객 데이터는 메시지에 넣지 말 것
4. **프로젝트별 토픽 분리** — 필요하면 `weppy-wms-...`, `weppy-side-...`처럼 분리해 알림 우선순위 차등 가능
5. **자율 실행과 짝지어 사용** — `--dangerously-skip-permissions` + 알림 조합이 가장 효과적. 장시간 자리를 비울 때 유용

---

## 참고

- Claude Code Hooks 공식 문서: https://docs.claude.com/en/docs/claude-code/hooks
- ntfy.sh 공식 문서: https://docs.ntfy.sh/
- Pushover API: https://pushover.net/api
