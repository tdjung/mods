# Claude Code Mods: 팀 추천 모음

Claude Code의 **mods** 기능을 소개하고, 팀(Linux 환경)에서 쓸 만한 mod를 정리한 저장소입니다.
설치와 사용법은 [USAGE.md](./USAGE.md), 코드 보안 점검 결과는 [SECURITY.md](./SECURITY.md)에 정리했습니다.

> 기준일: 2026-10-12 · 대상: Claude Code v2.1.287 이상, Linux 터미널(CLI)

## 5분 시작

패치 없이 바로 쓸 수 있는 것만 모았습니다. 컨테이너 안에서 실행하세요.

**1. 버전 확인**

```bash
claude --version   # 2.1.287 이상
```

**2. status line에 context % 붙이기 (mod 불필요)**

기존 게이트웨이 status line을 지우지 않고 뒤에 덧붙입니다. → [USAGE.md §4-12](./USAGE.md#4-12-mod-없이-status-line에-context-표시)

**3. mod 두 개 설치: context-bar(컨텍스트 막대) + blast-radius(위험 명령 확인)**

```bash
# context-bar
claude plugin marketplace add hamzafer/claude-code-mods
claude plugin install context-bar@claude-code-mods

# blast-radius (Anthropic 공식 샘플)
git clone https://github.com/anthropics/claude-code-playground.git
cd claude-code-playground/claude-code/mods
claude plugin marketplace add ./
claude plugin install blast-radius@claude-code-playground-mods --scope user
```

GitHub에 접속할 수 없으면 반입한 폴더로 설치합니다. → [USAGE.md §3](./USAGE.md#3-설치-폐쇄망)

**4. 확인**

`claude`를 다시 실행하면 프롬프트 위에 컨텍스트 막대가 보입니다. 버려도 되는 폴더에서 Claude에게 `rm -rf build`를 시켜 보면 blast-radius가 실행 전에 멈추고 확인을 요청합니다.

> mission-control(서브에이전트·툴 호출 트리)은 Linux에서 쓸모없는 모델 호출이 들어 있어 **한 줄을 지우고** 쓰는 걸 권장합니다. → [USAGE.md §4-1](./USAGE.md#4-1-mission-control-에이전트툴-호출-실시간-트리)

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

모두 **외부 네트워크 호출이 없고**, 공개 소스(MIT/Apache-2.0)이며, 코드를 직접 점검했습니다. 폐쇄망에서도 쓸 수 있습니다.
점검 결과의 근거와 권장 패치는 [SECURITY.md](./SECURITY.md)에 있습니다.

### Anthropic 공식 제공

| 분류 | Mod | 설명 | 명령 | 점검 결과 |
| --- | --- | --- | --- | --- |
| 내장 | `diff` | 커밋하지 않은 변경 사항을 파일별로 패널에 보여줍니다. **설치 불필요**(Claude Code에 내장). | `/diff` | 그대로 사용. 패널이 열려 있으면 Bash 호출마다 저장소 전체 `git diff`가 돌아감 |
| 내장 | `agents-md` | 프로젝트의 `AGENTS.md`를 `CLAUDE.md`처럼 지시 파일로 읽습니다. **설치 불필요**. | 자동 | **설정 권장.** `CLAUDE.md`가 없는 외부 저장소에서는 그 안의 `AGENTS.md`가 지시로 들어감 |
| 안전장치 | [blast-radius](https://claude-mods.com/blast-radius/) | `rm -rf`, `git reset --hard`, force push 등을 실행 전에 멈추고, 영향받는 파일을 보여준 뒤 진행/취소를 기다립니다. (DevRel 예제) | 자동 | 사용, 한계 숙지. 변수 경로는 피해를 작게 표시, 우회 명령 다수 |
| 비용·컨텍스트 | [token-weather](https://claude-mods.com/token-weather/) | 컨텍스트 사용률을 날씨 아이콘과 최근 12턴 추이로 프롬프트 위 한 줄에 보여줍니다. 한 턴에 급증한 순간을 잡기 좋습니다. (DevRel 예제) | 자동 | 그대로 사용. **다른 mod의 band를 가릴 수 있음** |
| 가시성 | [replay-theater](https://claude-mods.com/replay-theater/) | 직전 턴에서 Claude가 수정한 파일을 diff 하나씩 넘겨봅니다. (DevRel 예제) | `/replay` | 그대로 사용. **다른 mod의 band를 가릴 수 있음** |

`telemetry`, `sec-default`도 내장이지만 사용자가 쓰는 기능이 아니라 SECURITY.md에서만 다룹니다.

### 외부 개발자 제공

| 분류 | Mod | 설명 | 명령 | 점검 결과 |
| --- | --- | --- | --- | --- |
| 가시성 | [mission-control](https://claude-mods.com/mission-control/) | 메인 에이전트, 서브에이전트, 모든 툴 호출을 사이드 패널에 실시간 트리로 보여줍니다. 각 서브에이전트의 실행 시간, 툴 호출 수, 현재 상태(실행 중/실패)를 확인할 수 있습니다. *Linux에서는 Who 뷰만 지원.* | `/mission` | **패치 후 사용.** 수정 턴마다 Haiku로 코드 조각 전송(Linux에선 쓸모없음) |
| 가시성 | [change-ledger](https://claude-mods.com/change-ledger/) | 세션에서 수정한 모든 파일을 추가/삭제 줄 수, 어느 에이전트가 바꿨는지와 함께 보여주고, 아래에 git working tree를 표시합니다. | `/changes` | 사용, 패치 권장. 턴마다 git을 timeout 없이 실행 |
| 가시성 | [turn-timeline](https://claude-mods.com/turn-timeline/) | 현재 턴을 타임라인으로 그려 모델·툴·대기 시간 비율과 가장 느린 3단계를 보여줍니다. 시뮬레이션·빌드가 긴 작업의 병목 파악용. | `/timeline` | 그대로 사용 |
| 안전장치 | [Loop Breaker](https://claude-mods.com/loop-breaker/) | 같은 에러로 두 번 실패한 호출의 세 번째 시도와, 같은 파일을 두 버전 사이로 뒤집는 수정을 거부합니다. | 자동 (`/unstick`) | 사용. EDA 로그(시드 등)에서는 거의 동작 안 함 |
| 안전장치 | [Assertion Guardian](https://claude-mods.com/assertion-guardian/) | 테스트를 통과시키려고 assertion 삭제, skip 추가, 기대값 변경을 하는 수정을 거부합니다. *cocotb `test_*.py`는 인식.* | 자동 (`/allow-test-change`) | 사용, RTL엔 효과 제한. SV testbench 미인식 |
| 비용·컨텍스트 | [context-bar](https://claude-mods.com/context-bar/) | 컨텍스트 창을 카테고리별(시스템, MCP 툴, 메모리, 메시지) 막대로 보여주고, compaction 직전 70%/90%에서 노랑/빨강으로 바뀝니다. 컨텍스트를 많이 차지하는 MCP 서버를 찾는 데 유용합니다. | `/context-bar` | 그대로 사용 |
| 비용·컨텍스트 | [token-ledger](https://claude-mods.com/token-ledger/) | 프롬프트 아래에 세션 비용, 직전 턴 비용, 캐시 적중률을 표시하고, 턴별 비용 표를 보여줍니다. *금액은 정가 추정치.* | `/ledger` | 그대로 사용 |

> **band 겹침 주의:** 프롬프트 위 영역(band)은 여러 mod가 함께 씁니다. token-weather와 replay-theater는 자기 내용을 그릴 때 다른 mod의 내용을 이어 붙이지 않아서,
> 함께 켜면 context-bar나 blast-radius의 확인 버튼이 가려질 수 있습니다. 터미널을 144컬럼 이상으로 쓰면 blast-radius는 사이드 패널로 열려 영향을 받지 않습니다.

> context 사용률을 status line에 숫자로만 띄우고 싶다면 mod 없이 기본 status line으로도 됩니다. ([USAGE.md §4-12](./USAGE.md#4-12-mod-없이-status-line에-context-표시))

**처음엔 [5분 시작](#5분-시작)대로 `context-bar` + `blast-radius`로 시작하고, 필요하면 나머지를 추가하세요.**

## 참고 링크

- [Anthropic 공식 발표](https://claude.com/resources/articles/claude-code-mods)
- [Claude Code mods 공식 문서](https://code.claude.com/docs/en/plugins/mods/overview)
- [Claude Mods 디렉터리 (비공식, 수동 검증)](https://claude-mods.com/)
- [빌트인 mod 소스 (diff, agents-md, sec-default, telemetry)](https://github.com/anthropics/claude-code/tree/main/mods)
