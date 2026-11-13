# work-memory-plugin

Claude Code와 Codex가 **같은 업무 기록을 이어 쓰게** 하는 로컬 메모리입니다.
어제 Claude와 한 분석을 오늘 Codex가 이어받아도, "지금 어디까지 했고 뭐가 남았는지"를 같은 기록에서 확인합니다.

- MCP 서버 + Claude Code 스킬 + 명령(`/wm-resume`, `/wm-log`)
- 기록은 내 컴퓨터의 SQLite 파일 하나(`~/.work-memory/memory.db`)에만 저장됩니다. 클라우드나 유료 API를 쓰지 않습니다.
- Node.js 24만 있으면 동작합니다(외부 패키지 없음).

## 왜 만들었나
여러 프로젝트를 Claude와 Codex로 번갈아 작업하다 보니, 새 대화를 열 때마다 "지난번에 뭘 정했더라"를 다시 설명해야 했습니다.
AI 각자의 메모리는 서로 공유되지 않고, 대화 기록은 길어서 다시 읽기 어렵습니다.
그래서 **업무에서 의미 있는 사실만** 정해진 형식으로 남기고, 다음 작업을 시작할 때 그 요약부터 보게 만들었습니다.

## 어떻게 동작하나
```
Claude Code ─┐                          ┌─ 프로젝트 현재 상태
             ├─ MCP ─▶ 업무 Event ──────┼─ 열린 이슈 / 다음 할 일 / 회신 대기
Codex ───────┘   (작업·결정·발견·문제…)  └─ 검색 가능한 Timeline
```

| 원칙 | 내용 |
|---|---|
| Event가 원본 | 작업(WORK)·결정(DECISION)·발견(FINDING)·문제(ISSUE)·다음 할 일·회신 대기·상태 변경을 Event로 기록하고, 상태는 Event에서 계산합니다. |
| 중복은 근거로 | 같은 날 같은 사실을 Claude와 Codex가 각각 기록하면 새 기록 대신 근거(evidence)로 합칩니다. |
| 완료는 사람이 | AI가 "끝났다"고 판단한 것은 *제안*으로만 남고, 사용자가 확인해야 항목이 닫힙니다. |
| 모르면 묻기 | 프로젝트가 불확실한 기록은 검토 대기로 두고, 지정한 뒤에 반영합니다. |
| 내용 대신 경로 | 산출물은 파일 경로만 남기고 파일 내용은 저장하지 않습니다. |

## 예시: 작업을 이어갈 때 보이는 화면
`/wm-resume cohort`를 실행하면 Claude가 아래 맥락을 읽고 표로 요약합니다. (가상 프로젝트)

```text
# Sample Cohort Study — 가상 코호트의 치료 지속성 분석
현재 상태: 1차 분석 중 (since 2026-10-08)
마지막 활동: 2026-10-08

## Open Issue
- [issue:1] 기관별로 추적 종료일 정의가 다름

## Waiting
- [waiting:1] 추적 종료일 정의 회신 ← 공동연구자 (2026-10-08)

## 최근 Work
- 2026-10-08 Table 1 초안 작성
- 2026-10-06 대상자 선정 흐름도 작성

## 최근 Decision
- 2026-10-06 Index date 는 첫 처방일로 정의

## Timeline
- 2026-10-08 [RESOLUTION/codex] 완료 제안: action:1        ← 사용자가 승인해서 "Table 1 초안 작성" 할 일이 닫힘
- 2026-10-08 [STATUS_CHANGE/codex] 상태: 1차 분석 중
- 2026-10-06 [WORK/claude, evidence 2] 대상자 선정 흐름도 작성   ← Claude와 Codex 기록이 하나로 합쳐짐
```

## 쓰는 법
| 상황 | 말하기 / 명령 |
|---|---|
| 작업 이어가기 | `/wm-resume Sample Cohort Study` 또는 "cohort 이어서 하자" |
| 오늘 작업 기록 | `/wm-log cohort` → 무엇을 기록할지 먼저 보여 주고 기록 |
| 과거 경위 찾기 | "censoring 기준 어떻게 정했었지?" |
| 완료 처리 | "issue:1 끝났어" → 바로 닫힘 / AI의 완료 제안 → 확인 후 승인 |

## MCP 도구 (10개)
`list_projects` · `add_project` · `get_project_context` · `get_recent_activity` · `search_work_memory` ·
`log_work_event` · `log_work_session` · `get_open_items` · `resolve_item` · `review_event`

## 구성
- `server/mcp.ts`: MCP stdio 서버 (외부 라이브러리 없이 필요한 부분만 구현)
- `server/engine.ts`: 기록·상태 계산·중복 병합·검색 (SQLite FTS5)
- Claude Code 플러그인: 스킬 + `/wm-resume`·`/wm-log` 명령, Codex는 같은 MCP 서버를 설정에 등록
- 테스트 10개: 상태 반영, 중복 병합, 검토 대기, 완료 승인, 프로젝트 간 오닫힘 방지, 검색, 입력 검증, MCP 통신

## 코드
이 저장소는 **소개용**입니다. 소스 코드는 비공개로 관리합니다.
채용·협업 등으로 코드를 보고 싶으시면 [프로필](https://github.com/pkjphj2)의 연락처로 알려 주세요.

## License
All rights reserved. 열람만 허용하며, 허락 없이 복제·수정·배포·사용할 수 없습니다. 자세한 내용은 [LICENSE](LICENSE).
