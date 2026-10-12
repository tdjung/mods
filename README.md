# Claude Code Mods: 팀 추천 모음

Claude Code의 **mods** 기능을 소개하고, 팀(Linux 환경)에서 쓸 만한 mod를 정리한 저장소입니다.
설치와 사용법은 [USAGE.md](./USAGE.md), 코드 보안 점검 결과는 [SECURITY.md](./SECURITY.md)에 정리했습니다.

> 기준일: 2026-10-12 · 대상: Claude Code v2.1.287 이상, Linux 터미널(CLI)

## Mods란?

mod는 **Claude Code 프로세스 안에서 실행되는 작은 TypeScript/JavaScript 플러그인**입니다.
2026년 10월 1일 v2.1.287에서 정식 출시됐고, 기본으로 켜져 있습니다.

Claude Code는 동작할 때마다 이벤트(툴 호출, 프롬프트 제출, 화면 그리기 등)를 발생시킵니다.
mod는 이 이벤트에 함수를 걸어두고, 이벤트를 **관찰**하거나, **바꾸거나**, 아예 **대신 응답**할 수 있습니다.
Express 미들웨어와 비슷한 구조입니다.

## 기존 방식(hooks, skills, MCP)과 다른 점

기존 확장들은 모두 Claude Code **바깥**에서 동작했습니다. mod는 **안**에서 동작합니다.

| | 기존 settings hook / skill / MCP | mod |
| --- | --- | --- |
| 실행 위치 | 별도 프로세스(셸 명령, HTTP), 또는 Claude가 읽는 텍스트 | Claude Code 프로세스 내부 |
| 화면(UI) | 바꿀 수 없음 | 사이드 패널, 프롬프트 위 band, 기존 UI 일부를 다시 그림 |
| 툴 호출 | 막거나 입력을 조정하는 정도 | 결과를 직접 반환하거나 최종 권한 판단을 덮어씀 |
| 모델 요청 | 개입 불가 | 요청을 가로채 수정·라우팅·직접 응답 |
| 슬래시 명령 | 결국 Claude에게 프롬프트를 보냄 | Claude 턴 없이 즉시 실행(토큰 0) |
| 상태 유지 | 호출마다 새 프로세스 | 세션 내내 메모리에 상태 유지 |

핵심은 **결정론적 코드**라는 점입니다. Claude에게 "매번 이렇게 해줘"라고 부탁하는 것과 달리,
mod는 토큰을 쓰지 않고 항상 같은 방식으로 동작합니다.

> **주의:** mod는 샌드박스 없이 내 권한으로 실행됩니다. 파일·환경변수 읽기, 모든 프롬프트와 툴 호출 열람,
> 툴 호출 대신 승인까지 가능합니다. 설치 전 반드시 `claude plugin validate`로 검토하세요. ([USAGE.md §5](./USAGE.md#5-보안-체크리스트))

## 추천 mods

8개 모두 **외부 네트워크 호출이 없고**, 공개 소스(MIT/Apache-2.0)이며, 코드를 직접 점검했습니다([SECURITY.md](./SECURITY.md)). 폐쇄망에서도 쓸 수 있습니다.

### 가시성: Claude가 뭘 하고 있는지 보기

| Mod | 설명 | 명령 |
| --- | --- | --- |
| [mission-control](https://claude-mods.com/mission-control/) | 메인 에이전트, 서브에이전트, 모든 툴 호출을 사이드 패널에 실시간 트리로 보여줍니다. 각 서브에이전트의 실행 시간, 툴 호출 수, 현재 상태(실행 중/실패)를 확인할 수 있습니다. *Linux에서는 Who 뷰만 지원(파일 맵 Code 뷰는 macOS 전용).* | `/mission` |
| [change-ledger](https://claude-mods.com/change-ledger/) | 세션에서 수정한 모든 파일을 추가/삭제 줄 수, 어느 에이전트가 바꿨는지와 함께 보여주고, 아래에 git working tree를 표시합니다. | `/changes` |
| [turn-timeline](https://claude-mods.com/turn-timeline/) | 현재 턴을 타임라인으로 그려 모델·툴·대기 시간 비율과 가장 느린 3단계를 보여줍니다. 시뮬레이션·빌드가 긴 작업의 병목 파악용. | `/timeline` |

### 안전장치: 사고 방지

| Mod | 설명 | 명령 |
| --- | --- | --- |
| [blast-radius](https://claude-mods.com/blast-radius/) | Anthropic 공식 샘플. `rm -rf`, `git reset --hard`, force push 등을 실행 전에 멈추고, 영향받는 파일을 보여준 뒤 진행/취소를 기다립니다. | 자동 |
| [Loop Breaker](https://claude-mods.com/loop-breaker/) | 같은 에러로 두 번 실패한 호출의 세 번째 시도와, 같은 파일을 두 버전 사이로 뒤집는 수정을 거부합니다. 합성·컴파일 에러를 같은 방식으로 반복해서 고치는 상황을 막아줍니다. | 자동 (`/unstick`) |
| [Assertion Guardian](https://claude-mods.com/assertion-guardian/) | 테스트를 통과시키려고 assertion 삭제, skip 추가, 기대값 변경을 하는 수정을 거부합니다. *SystemVerilog testbench는 인식하지 못함(cocotb `test_*.py`는 인식).* | 자동 (`/allow-test-change`) |

### 비용·컨텍스트: 사용량이 새는 지점 보기

| Mod | 설명 | 명령 |
| --- | --- | --- |
| [context-bar](https://claude-mods.com/context-bar/) | 컨텍스트 창을 카테고리별(시스템, MCP 툴, 메모리, 메시지) 막대로 프롬프트 위에 보여주고, compaction 직전 70%/90%에서 노랑/빨강으로 바뀝니다. 컨텍스트를 많이 차지하는 MCP 서버를 찾는 데 유용합니다. | `/context-bar` (숨기기/보이기) |
| [token-ledger](https://claude-mods.com/token-ledger/) | 프롬프트 아래에 세션 비용, 직전 턴 비용, 캐시 적중률을 표시하고, 턴별 비용 표를 보여줍니다. *금액은 `modelPricing` 설정이 없으면 정가 추정치.* | `/ledger` |

> context 사용률을 status line에 숫자로만 띄우고 싶다면 mod 없이 기본 status line으로도 됩니다. ([USAGE.md §4-9](./USAGE.md#4-9-mod-없이-status-line에-context-표시))

**처음엔 `mission-control` + `blast-radius` + `context-bar`로 시작하는 걸 권장합니다.**

## 참고 링크

- [Anthropic 공식 발표](https://claude.com/resources/articles/claude-code-mods)
- [Claude Code mods 공식 문서](https://code.claude.com/docs/en/plugins/mods/overview)
- [Claude Mods 디렉터리 (비공식, 수동 검증)](https://claude-mods.com/)
- [빌트인 mod 소스 (diff, agents-md, sec-default, telemetry)](https://github.com/anthropics/claude-code/tree/main/mods)
