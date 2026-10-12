# 추천 Mods 보안 점검 결과

추천한 mod(Anthropic 공식 5개, 외부 7개)와 내장 mod의 소스 코드를 **읽기만 해서** 점검한 결과입니다. 실행이나 설치는 하지 않았습니다.
기준은 팀 환경인 **Linux + 폐쇄망 + 엔터프라이즈 요금제**입니다.

> 점검일: 2026-10-12. 아래 커밋 기준이며, 다른 버전을 반입하면 다시 점검해야 합니다.
> 이 커밋들은 팀 카탈로그(`plugins/`)에 그대로 들어 있고, mission-control과 change-ledger에는 아래 권장 패치가 **적용돼 있습니다**([patches/](./patches)).
> 줄 번호는 해당 커밋의 파일 기준입니다.

| 저장소 | 커밋 | 점검한 mod |
| --- | --- | --- |
| [hamzafer/claude-code-mods](https://github.com/hamzafer/claude-code-mods) | `11abba8` | mission-control, context-bar |
| [arasovic/claude-code-mods](https://github.com/arasovic/claude-code-mods) | `9f1bef6` | change-ledger, turn-timeline |
| [anthropics/claude-code-playground](https://github.com/anthropics/claude-code-playground) | `569c528` | blast-radius, token-weather, replay-theater |
| [ccdwyer/loop-breaker](https://github.com/ccdwyer/loop-breaker) | `1eeaaba` | Loop Breaker |
| [ccdwyer/assertion-guardian](https://github.com/ccdwyer/assertion-guardian) | `ae1de89` | Assertion Guardian |
| [Arunjay4213/claude-mods](https://github.com/Arunjay4213/claude-mods) | `d4fffd7` | token-ledger |
| [anthropics/claude-code](https://github.com/anthropics/claude-code/tree/main/mods) | `3deb873` | 내장 mod: diff, agents-md, telemetry, sec-default ([§4](#4-빌트인-mods)) |
| [anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official) | `f359d43` | code-modernization (외부 통신 여부만, [§4](#4-빌트인-mods)) |

## 요약

추천 mod는 모두 **외부 네트워크 호출, 런타임 코드 다운로드(dynamic import, eval), 환경변수의 비밀값 읽기가 없습니다.**
명령어 주입 경로도 찾지 못했습니다. 그대로 써도 되는 것과 패치하고 쓰면 좋은 것은 아래와 같습니다.

### Anthropic 공식 제공

| Mod | 판정 | 핵심 | 상세 |
| --- | --- | --- | --- |
| diff (내장) | 그대로 사용 | 외부 접근 없음. git만 5초 timeout으로 실행. 패널이 열려 있을 때 Bash 호출마다 저장소 전체 diff | [§4](#diff) |
| agents-md (내장) | **설정 권장** | 외부 접근 없음. 단 `CLAUDE.md`가 없는 프로젝트에서는 `/`부터의 모든 `AGENTS.md`와 하위 폴더 것을 크기 제한 없이 지시로 넣음 | [§4](#agents-md) |
| token-weather | 그대로 사용 | 외부 접근 없음. `$.session.usage()`만 읽음. 다른 mod의 band를 가릴 수 있음 | [§1](#token-weather) |
| replay-theater | 그대로 사용 | 외부 접근 없음. 수정 내용을 메모리에만 보관. 다른 mod의 band를 가릴 수 있음 | [§1](#replay-theater) |
| blast-radius | 사용, 한계 숙지 | 미리보기가 피해를 작게 보여줄 수 있음. 우회 명령 다수 | [§2](#blast-radius-claude-codemodsblast-radiushooksblast-radiusmjs) |

### 외부 개발자 제공

| Mod | 판정 | 핵심 |
| --- | --- | --- |
| turn-timeline | 그대로 사용 | 외부 접근 없음. 기록·표시만 함 |
| context-bar | 그대로 사용 | 외부 접근 없음. 숨김 여부 하나만 저장 |
| token-ledger | 그대로 사용 | 외부 접근 없음. 금액이 정가 추정치일 수 있음 |
| change-ledger | **팀 카탈로그에 패치 적용** | 원본은 턴마다 git을 timeout 없이 실행. 큰 저장소에서 느려질 수 있음 |
| mission-control | **팀 카탈로그에 패치 적용** | 원본은 파일을 수정한 턴마다 Haiku를 호출해 코드 조각을 보냄. Linux에서는 쓸모없음 |
| Loop Breaker | 사용 | EDA 로그에서는 거의 동작하지 않음(잘못 막지는 않음) |
| Assertion Guardian | 사용, RTL엔 효과 제한 | SV/Verilog testbench 미인식. `/data/...` 경로에서 판별이 꺼짐 |

### 공통 주의: band(프롬프트 위 영역)를 독차지하는 mod

band는 여러 mod가 순서대로 그리는 공간입니다. 다른 mod의 내용을 지키려면 `next(e)`로 나머지를 그리게 한 뒤 덧붙여야 합니다.
context-bar, mission-control은 그렇게 하지만 **token-weather(`token-weather.mjs:54`)와 replay-theater(`replay-theater.mjs:225, 228`)는 자기 내용만 반환**합니다.
로드 순서에 따라 context-bar의 막대나, 터미널이 좁을 때 band로 뜨는 blast-radius의 확인 버튼이 가려질 수 있습니다.
blast-radius는 버튼이 안 보이면 10분 뒤 거부하므로 위험하진 않지만, 함께 쓸 때는 **터미널을 144컬럼 이상**으로 쓰세요.

### 공통 주의: 차단형 mod는 오류가 나면 통과시킵니다

mod의 hook이 예외를 던지거나 시간 제한(10초)을 넘기면 Claude Code는 그 hook을 건너뜁니다.
이걸 막으려면 hook을 등록할 때 `.catch`로 "거부"를 돌려주는 대비책을 붙여야 하는데,
**blast-radius, Loop Breaker, Assertion Guardian 셋 다 이 대비책이 없습니다.**
따라서 셋 다 "최선을 다하는 안전망"이고, 반드시 막아야 하는 명령은 permission deny 규칙으로 막아야 합니다.

---

## 1. 가시성 mods

### mission-control (`mods/mission-control/hooks/register.tsx`)

| 항목 | 결과 |
| --- | --- |
| 네트워크 | 없음 |
| 프로세스 실행 | 1곳(326행): 맵 렌더링용 `sh -c` + headless Chrome. 스크립트는 고정 문자열이고 변수는 모두 인용된 인자로 전달되어 주입 위험 없음. 단 **macOS Chrome 경로가 있을 때만 실행**(14, 306행) → Linux에서는 실행되지 않음 |
| 파일 읽기 | Claude가 툴로 접근하는 코드 파일을 읽어 import 관계만 추출(102행). 툴 실행 **전에** 읽으므로 거부된 호출의 파일도 읽지만, 외부로 나가지 않음 |
| 파일 쓰기 | `$TMPDIR/mission-control/`에 SVG·PNG·Chrome 프로필(macOS에서만). 정리하지 않음 |
| 환경변수 | `TMPDIR`만 |
| 모델 호출 | **있음.** 아래 참고 |
| 툴 차단·재작성 | 없음. 기록만 하고 그대로 통과 |

**Haiku 호출 상세**

- 메인 턴이 끝날 때 수정된 파일이 있으면 백그라운드로 `explain()`을 호출합니다(134행).
- 보내는 내용(113, 261행): 파일 경로 끝 두 단계, Edit의 수정 전후 각 300자, Write의 앞 400자. 파일당 최대 1,500자, **파일 개수 제한 없음**.
- 용도: 파일 맵(Code 뷰) 카드 아래 "무엇이 바뀌었는지" 한 줄 요약(map.ts 130, 227~228행). **Who 뷰와 상단 band는 이 요약을 쓰지 않습니다.**
- 끄는 설정이 없고 실패해도 표시가 없습니다(`.catch(() => {})`).

**Linux 영향:** Code 뷰는 Linux에서 그려지지 않으므로 이 호출은 **쓸모없이 비용만 들고 수정 코드 조각을 모델로 보냅니다.**

**권장 패치:** 134행 삭제. → 팀 카탈로그에 적용됨, 패치 후 `claude plugin validate` 결과에서 `$.model.complete` 호출이 사라진 것을 확인했습니다.

```diff
-      void explain($, t, new Map(changes)).catch(() => {})
```

### change-ledger (`change-ledger/hooks/register.tsx`)

| 항목 | 결과 |
| --- | --- |
| 네트워크 | 없음 (ahead/behind는 로컬 추적 ref만 읽음) |
| 프로세스 실행 | `git status --porcelain=v1 --branch`(93행), `git diff --numstat HEAD`(96행). 인자 배열로 실행(셸 없음), Claude가 준 문자열이 들어가지 않음. **timeout 없음** |
| 파일 읽기 | Write 직전 대상 파일을 통째로 읽어 지워질 줄 수 계산(158행) |
| 파일 쓰기 | 없음 (메모리 상태만) |
| 환경변수·모델 | 없음 |
| 툴 차단·재작성 | 없음 |

**문제점:** 메인 턴이 끝날 때마다 git 두 개를 실행하고 끝날 때까지 기다립니다(168~169행).
생성 파일이 많거나 `.gitignore`가 덜 된 RTL 저장소에서는 `git status`의 untracked 스캔과 `git diff`가 오래 걸려 **턴 종료가 늦어질 수 있습니다.**
`/changes` 명령 쪽은 git 실패를 처리하지 않아 오류가 날 수 있습니다(145행).

**권장 패치:** `git status`에 `-uno`, 두 명령에 `--no-optional-locks` 추가, 또는 패널이 열려 있을 때만 git 실행.
→ 팀 카탈로그에는 `-uno`, `--no-optional-locks`, 5초 timeout, `/changes` 오류 처리를 적용했습니다([patches/change-ledger.patch](./patches/change-ledger.patch)).

### turn-timeline (`turn-timeline/hooks/register.tsx`)

네트워크, 프로세스 실행, 파일 읽기·쓰기, 환경변수, 모델 호출이 **모두 없습니다.**
`tool.call`(196~206행)과 `turn.step`(208~220행)은 시간만 재고 결과를 그대로 돌려줍니다.
툴 호출의 명령·파일 경로 앞부분을 메모리에만 보관하며(최대 400단계), 외부로 보내거나 저장하지 않습니다.
**패치 불필요.**

### context-bar (`mods/context-bar/hooks/register.tsx`, 182줄)

| 항목 | 결과 |
| --- | --- |
| 네트워크·프로세스·파일·환경변수·모델 | **모두 없음** |
| 데이터 출처 | `$.session.usage({ breakdown: 'summary' })`(105행). `/context`와 같은 분류를 엔진이 로컬로 추정 |
| 저장 | `$.store`에 숨김 여부(`isHidden`) 하나만(48행) |
| 툴 차단·재작성 | 없음. 프롬프트 위 영역에 막대를 덧그릴 뿐, 다른 mod가 그린 내용은 유지(54행) |
| 갱신 시점 | 세션 시작, 메인 턴 종료(서브에이전트 턴 제외, 36행), `/compact` 후 |

모든 오류를 `.catch(() => {})`로 삼켜서 실패해도 세션에 영향을 주지 않습니다. Linux 의존성 없음. **패치 불필요.**

### token-ledger (`plugins/token-ledger/hooks/register.tsx`, `format.ts`)

| 항목 | 결과 |
| --- | --- |
| 네트워크·프로세스·파일·환경변수·모델 | **모두 없음** (`format.ts`에도 없음) |
| 데이터 출처 | `$.session.usage().cost.usd`의 턴 사이 변화량(40~46, 105행), 턴 종료 이벤트의 토큰 수 |
| 저장 | `$.store`에 세션 ID와 턴별 행(모델명, 토큰 수, 캐시, 소요 시간, 금액). 최대 200행. **프롬프트나 코드 내용은 저장하지 않음** |
| 툴 차단·재작성 | 없음. `/clear`, `/resume`은 관찰만 하고 그대로 통과(191~201행) |
| 화면 | `$.ui.status`로 프롬프트 아래 한 줄(51행), `/ledger` 패널 |

**주의:** 금액은 Claude Code가 계산한 값을 그대로 씁니다. 이 값은 **`modelPricing` 설정이 없으면 정가 기준 추정치**라 실제 엔터프라이즈 청구액과 다를 수 있습니다.
서브에이전트 비용이 메인 턴 행에 섞일 수 있습니다. **패치 불필요.**

### token-weather

Anthropic DevRel 예제 (`claude-code/mods/token-weather/hooks/token-weather.mjs`, 122줄).

| 항목 | 결과 |
| --- | --- |
| 네트워크·프로세스·파일·환경변수·모델·저장 | **모두 없음** |
| 데이터 출처 | `$.session.usage()`의 `context`(60행). 분류(breakdown)를 요청하지 않아 토큰 계산 요청도 없음 |
| 상태 | 최근 12턴 사용량을 모듈 변수에만 보관 |
| 툴 차단·재작성 | 없음 |
| 갱신 | 세션 시작, 메인 턴 종료(서브에이전트 턴 제외) |

**주의:** band를 그릴 때 `next(e)`를 부르지 않고 자기 한 줄만 반환합니다(54행). 다른 mod의 band가 가려질 수 있습니다.
사용률은 **전체 창 크기** 기준이라 compaction 지점 기준인 Claude Code 알림보다 낮게 나옵니다(README: band 81% / 알림 90%). **패치 불필요.**

### replay-theater

Anthropic DevRel 예제 (`claude-code/mods/replay-theater/hooks/replay-theater.mjs`, 249줄).

| 항목 | 결과 |
| --- | --- |
| 네트워크·프로세스·환경변수·모델 | **없음** |
| 파일 읽기 | Write 직전 대상 파일을 통째로 읽어 diff 계산(94~95행) |
| 파일 쓰기·저장 | 없음. 이번 턴의 수정 내용(diff)을 모듈 변수에만 보관 |
| 툴 차단·재작성 | 없음. 기록 후 그대로 통과(196~204행) |
| 대상 툴 | Edit, Write, MultiEdit |

**주의**

- band에서 `next(e)`를 부르지 않습니다(225, 228행). 다른 mod의 band가 가려질 수 있습니다.
- 툴 실행 **전에** 기록하므로, 거부되거나 실패한 수정도 재생 목록에 나옵니다.
- diff 계산은 양쪽 400줄 이하일 때만 LCS로 하고, 넘으면 전부 삭제/추가로 표시합니다(36~38행). 큰 파일에서도 느려지지 않습니다.

**패치 불필요.**

---

## 2. 안전장치 mods

### blast-radius (`claude-code/mods/blast-radius/hooks/blast-radius.mjs`)

| 항목 | 결과 |
| --- | --- |
| 네트워크 | 없음. force push 미리보기는 로컬 원격 추적 ref만 읽음 |
| 프로세스 실행 | `bash -c` 고정 스크립트 + 인자 전달(278, 337행), `git` 여러 명령, `find`, `du`, 대기용 `sleep 0.25`. 모두 timeout 있음(5~20초) |
| 주입 위험 | **없음.** 경로 확장은 `compgen -G`로만 하고 모든 사용처가 인용됨(318~330행). `$(touch …)`, 백틱, `${x:=$(…)}` 패턴으로 직접 시험해도 미리보기 중 실행되지 않음 |
| 파일 쓰기·환경변수·모델 | 없음 |
| 대상 | Bash 툴만(27행) |

**문제점**

- **미리보기가 피해를 작게 보여줄 수 있습니다.** 경로를 문자 그대로 계산하므로 `rm -rf "$BUILD_DIR"/`는 "지울 것 없음"으로 표시됩니다(341행). 변수가 비어 있으면 실제로는 `/`가 지워집니다.
- **잡지 못하는 명령:** `timeout 5 rm -rf`, `bash -c "…"`, `sh -c`, `eval`, `xargs rm`, `find -delete`, 플래그 없는 `rm *`, `git branch -D`, `git push --delete`, `git stash drop`.
- **실패 시 통과:** 대부분은 `try` 안에서 실패하면 거부하지만, 명령 분류(28행)와 대기열 루프(35~40행)는 `try` 밖이라 여기서 오류가 나면 통과됩니다.
- 마이그레이션 명령은 Proceed 전에 미리보기용으로 프로젝트 코드(`manage.py` 등)를 실행합니다(443~446행). RTL 팀과는 관련이 적습니다.

**권장 패치:** `$`나 백틱이 든 경로는 "측정 불가"로 표시, 분류기에 `timeout`·`xargs`·`bash -c`·`find -delete` 추가, `.catch` 거부 대비책 추가.

> **이름 충돌 주의:** hamzafer 저장소에도 `blast-radius`가 있는데, Anthropic 것의 사본이 아니라 **다른 작성자의 다른 프로그램**(v0.2.2)이고 점검하지 않았습니다. Anthropic 저장소의 것을 쓰세요.

### Loop Breaker (`hooks/register.tsx`)

| 항목 | 결과 |
| --- | --- |
| 네트워크·프로세스·환경변수·모델 | 없음 |
| 파일 읽기 | 왕복 수정 판정 시 대상 파일만(226행) |
| 저장 | 세션 상태에 해시와 개수만. 파일 내용 저장 안 함 |
| 차단 | 같은 호출이 같은 에러로 2번 실패하면 3번째 거부(290~302행), 3번 왕복한 수정의 4번째 거부(304~321행) |

**EDA 환경에서의 한계:** 에러 비교 전 지우는 값(72~85행)은 시각, 소요 시간(ms/s/min), `/tmp` 경로, PID, 16진 주소입니다.
**랜덤 시드, 시뮬레이션 시각(ns/ps), job·run ID, `/scratch` 같은 경로는 그대로** 비교하므로, 이런 값이 매번 바뀌는 EDA 로그에서는 "다른 에러"로 보고 **아예 동작하지 않습니다.** 잘못 막는 일은 없습니다.
`/var/folders` 정규식(77행)은 macOS 전용이라 Linux에서는 의미 없습니다.

**권장 패치:** `normalizeError`에 시드·시뮬레이션 시각·job ID·팀 작업 경로 정규화 추가.

### Assertion Guardian (`hooks/register.tsx`, `hooks/analyze.ts`)

| 항목 | 결과 |
| --- | --- |
| 네트워크·환경변수·모델 | 없음 |
| 프로세스 실행 | 테스트 파일이 디스크에 없을 때만 `git rev-parse`, `git show HEAD:<경로>`(32, 41행). 인자 배열, timeout 5초 |
| 파일 쓰기 | 없음 |
| 차단 | 테스트 파일을 약하게 만드는 Edit/Write/NotebookEdit, 테스트 파일을 건드리는 Bash 거부 |
| 실패 시 | 이전 버전을 못 읽으면 거부(안전), 그 외 예외·시간 초과는 통과 |

**RTL 팀이 알아야 할 점 (analyze.ts 8~19행 확인)**

- 테스트 파일 이름 패턴은 JS/TS, Python, Go, Ruby, Java, Swift, Kotlin, C#, Scala, PHP 등만 있습니다.
- `tests/` 폴더 안이어도 확장자가 `CODE_FILE` 목록에 있어야 하는데, **`.sv`, `.v`, `.c`가 없습니다.** `tb_*.sv`도 인식하지 않습니다.
- **cocotb `test_*.py`는 보호됩니다.** 다만 `@cocotb.test(skip=True)`, `expect_fail=True`는 못 잡고, 테스트 개수는 `def test_*`만 셉니다.
- **Linux 서버 주의:** 폴더 규칙을 절대경로 전체에 적용합니다(17~18행). 저장소가 **`/data/...` 아래**에 있으면 `data/`가 "테스트 아님" 폴더로 걸려 폴더 기준 판별이 통째로 꺼집니다. 반대로 상위 경로에 `test/`가 있으면 모든 코드 파일이 테스트로 취급됩니다.

**권장 패치:** `CODE_FILE`에 `sv|svh|v|vhd|c|cc|cpp|h` 추가, `tb_*.sv`·`*_tb.sv` 패턴 추가, cocotb skip/expect_fail 패턴 추가, 폴더 규칙을 저장소 루트 기준 상대경로로 적용, `.catch` 거부 대비책 추가.

---

## 3. 폐쇄망 설치 관련

| 마켓플레이스 | 플러그인 출처 | 폐쇄망에서 `marketplace add ./` |
| --- | --- | --- |
| **tdjung/mods (팀 카탈로그 `team-mods`)** | 로컬 경로(`./plugins/...`) | **가능** (설치 테스트 완료) |
| hamzafer/claude-code-mods | 로컬 경로(`./mods/...`) | 가능 |
| arasovic/claude-code-mods | 로컬 경로(`./change-ledger` 등) | 가능 |
| anthropics/claude-code-playground | 로컬 경로(`./blast-radius`) | 가능 (`claude-code/mods` 폴더에서) |
| Arunjay4213/claude-mods | 로컬 경로(`./plugins/...`) | 가능 |
| **ccdwyer/claude-mods** | **GitHub** (`"source": "github"`) | **불가.** `--plugin-dir` 또는 `CLAUDE_CODE_PLUGIN_DIRS`로 폴더를 직접 로드 |

- ccdwyer 저장소의 `install.sh`는 GitHub에 접속하므로 폐쇄망에서 쓰지 마세요.
- hamzafer와 arasovic는 마켓플레이스 이름이 둘 다 `claude-code-mods`라 동시에 등록할 수 없습니다. 한쪽은 폴더로 직접 로드하세요.
- `plugin.json`의 GitHub URL(작성자, 홈페이지)은 메타데이터일 뿐 mod 코드가 접속하지 않습니다.

## 4. 빌트인 mods

설치하지 않아도 Claude Code 실행 파일 안에 들어 있는 mod입니다. 소스는 [anthropics/claude-code `mods/`](https://github.com/anthropics/claude-code/tree/main/mods)(커밋 `3deb873`)에 공개돼 있습니다.
**마켓플레이스에 없어서 `/plugin`에서 검색해도 나오지 않습니다.** 팀 환경은 Claude Code 2.1.295입니다.

| Mod | 로드 조건 | 팀 환경 확인 결과 | 점검 범위 |
| --- | --- | --- | --- |
| `diff` | 항상 | `/diff` 동작 확인 | 코드 전체 |
| `agents-md` | 항상 | 미확인 (`AGENTS.md` 지원) | 코드 전체 |
| `telemetry` | Claude Code 분석이 켜진 환경 | **켜지는 조건에 해당** (아래 참고) | 전송 조건·대상·재시도 코드 |
| `sec-default` | 관리 설정이 있는 머신, 또는 Team·Enterprise 조직 계정 | 관리 설정 없음. 계정 인식 여부는 화면으로 확인 불가 | README |

### diff

`/diff` 패널. 경로는 `mods/diff/hooks/` 기준입니다.

| 항목 | 결과 |
| --- | --- |
| 네트워크·모델 | 없음 |
| 프로세스 실행 | `git`만, 인자 배열(셸 없음), **timeout 5초**(`limits/timing/git-timeout-ms.ts`), 모든 읽기에 `--no-optional-locks` |
| 인자로 들어가는 값 | git 출력에서 나온 경로(`--literal-pathspecs`), 검증된 SHA, `origin/HEAD`의 브랜치 이름. **사용자나 Claude가 입력한 문자열은 들어가지 않음** |
| 파일 | 읽기는 `.git/HEAD`와 파일 시각 정보만. 쓰기 없음. `$.store`에 패널 열림 여부와 비교 기준만 저장 |
| 환경변수 | `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING`만 |
| 툴 차단·재작성 | 없음. 패널의 "ask" 버튼을 누를 때만 해당 파일 diff를 다음 프롬프트에 붙임(400줄, 32,000자 제한) |
| telemetry | 열린 방식, 화면 폭 구간 같은 **고정값만**. 경로나 diff 내용은 없음 |

**큰 RTL 저장소에서의 비용**

- **패널이 열려 있으면** Edit·Write·Bash 호출마다(150ms 묶음) 저장소 전체 `git diff HEAD --shortstat`, `--numstat`을 실행합니다. 백그라운드라 턴을 늦추지는 않습니다.
- **패널이 닫혀 있으면** git을 실행하지 않습니다. 단 Claude의 첫 수정 때 저장소 확인을 한 번 하고, 터미널이 넓으면 패널을 자동으로 열 수 있습니다.
- 처음 열 때 `git status --untracked-files=all`로 추적 안 되는 파일을 모두 훑습니다. **빌드 산출물이 `.gitignore`에 없으면 느립니다.**
- 변경 파일 500개를 넘으면 합계만, 파일당 diff는 400줄·1MB까지만 처리합니다.
- partial clone(지연 다운로드) 저장소는 폐쇄망에서 git이 객체를 받으려다 5초 timeout으로 실패할 수 있습니다.

**판정:** 그대로 사용. Bash를 많이 쓰는 세션에서는 패널을 닫아 두세요.

### agents-md

`AGENTS.md`를 프로젝트 지시 파일로 읽는 mod. 경로는 `mods/agents-md/hooks/` 기준입니다.

| 항목 | 결과 |
| --- | --- |
| 네트워크·프로세스·모델 | 없음 |
| 파일 읽기 | `AGENTS.md`, `.claude/AGENTS.md`와 그 안의 `@` import |
| 환경변수 | `CLAUDE_CODE_SIMPLE`, `CLAUDE_CODE_DISABLE_ATTACHMENTS`, `HOME` |
| telemetry | 파일 수, 길이 같은 **숫자만** |

**읽는 범위 (보안상 중요)**

- 기본 모드(`claude-md-or-agents-md`)는 `/`부터 작업 폴더까지 `CLAUDE.md`가 하나도 없을 때, **같은 범위(`/`부터 작업 폴더까지)의 모든 `AGENTS.md`**를 지시로 넣습니다(`register.ts:113~130`). `/AGENTS.md`, `/workspace/AGENTS.md`, `$HOME/AGENTS.md`도 대상입니다.
- 같은 조건(`CLAUDE.md` 없음)에서, 작업 중 Claude가 하위 폴더의 파일을 Read하면 그 폴더까지의 `AGENTS.md`를 툴 결과에 붙입니다(`register.ts:203~213`).
- 즉 **기본 모드에서는 `CLAUDE.md`가 있는 프로젝트라면 아무것도 넣지 않습니다.** `claude-md-and-agents-md` 모드로 바꾸면 `CLAUDE.md`가 있어도 위 동작이 모두 일어납니다.
- **크기 제한이 없고 내용을 그대로** 지시로 넣습니다.

**위험:** `CLAUDE.md`가 없는 외부 저장소를 받아 그 안에서 Claude Code를 실행하면, 그 저장소나 벤더 IP·SDK 폴더 안의 `AGENTS.md`가 **신뢰된 프로젝트 지시**가 됩니다. 프롬프트 주입 경로입니다.
팀 저장소에 `CLAUDE.md`를 두면 기본 모드에서는 이 경로가 막힙니다.

**권장 설정:** 팀이 `AGENTS.md`를 쓰지 않는다면 끄세요. 사용자 설정(`~/.claude/settings.json`)에 넣거나 `/config`의 "Project instructions"에서 바꿀 수 있습니다.
**프로젝트의 `.claude/settings.json`에서는 읽지 않습니다.**

```json
{
  "pluginConfigs": {
    "agents-md@builtin": { "options": { "instructionFiles": "claude-md" } }
  }
}
```

특정 경로만 빼려면 `claudeMdExcludes`도 적용됩니다. mod 자체를 끄려면 `/plugin`의 내장(built-in) 목록에서 끕니다.
`disableAllHooks`나 `--bare`로는 꺼지지 않습니다(agents-md README). 위 설정 키는 코드와 README 기준이며 실제 적용은 `/config`에서 확인하세요.

### telemetry

| 항목 | 결과 |
| --- | --- |
| 전송 대상 | `https://api.anthropic.com/api/event_logging/v2/batch` 하나 (`entries/ingest-url.ts:9`) |
| 보내는 내용 | 이벤트 이름과 속성. 값은 숫자, 불리언, 정해진 선택값만이고 **프롬프트·코드 같은 자유 텍스트는 없음**. 세션 ID, 모델, OS·셸 등 실행 환경 정보, 저장소 원격 주소의 해시(`rh`, `metadata-of.ts:14`)가 함께 들어감 |
| 실패 시 | 한 번 재시도 후 버림. 인증 정보가 없어도 버림 (`telemetry-of.ts:17`) |
| 다른 mod의 사용 | 설치한 mod는 호출이 거부됨. 내장 플러그인만 사용 |

**전송하지 않는 조건** (`is-analytics-off.ts`). 하나라도 해당하면 아무것도 보내지 않습니다.

- `DISABLE_TELEMETRY` 또는 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`가 설정됨, `DO_NOT_TRACK`이 켜짐
- `CLAUDE_CODE_USE_BEDROCK`, `_VERTEX`, `_FOUNDRY` 등 서드파티 제공자 사용
- `CLAUDE_CODE_USE_GATEWAY`가 켜짐, 또는 관리 설정이 게이트웨이 로그인을 강제
- `CLAUDE_CODE_CUSTOM_OAUTH_URL`이 설정됨

**팀 환경:** 컨테이너에 있는 관련 환경변수는 `ANTHROPIC_BASE_URL`과 `CLAUDE_CODE_SSE_PORT` 두 개뿐입니다.
`ANTHROPIC_BASE_URL`은 위 조건에 없으므로 **telemetry는 분석이 켜진 것으로 판단합니다.**
폐쇄망이라 전송은 실패하고 한 번 재시도 후 버려지므로 실제로 나가는 데이터는 없습니다. 다만 외부 통신이 열리는 순간 전송이 시작될 수 있으니
**이미지에 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1`을 넣는 것을 권장합니다.** ([USAGE.md §1-3](./USAGE.md#1-3-외부-전송-끄기-권장))

> 위는 telemetry mod 코드 기준입니다. Claude Code 본체가 `ANTHROPIC_BASE_URL`만으로 자체 분석을 끄는지는 소스가 공개되지 않아 확인하지 못했습니다.
> 이 경우에도 위 환경변수를 넣으면 꺼집니다.

### sec-default

설치한 mod가 조직의 hook, 관리 설정, 권한 규칙(deny 등)을 건드리지 못하게 막는 보호 장치입니다. 자체 정책은 없습니다.

- 관리 설정이나 `managed-mcp.json`이 있는 머신, 또는 Team·Enterprise 조직 계정일 때 가장 바깥에 로드됩니다.
- 팀 컨테이너에는 관리 설정이 없으므로, 게이트웨이를 거친 로그인이 Enterprise 조직으로 인식될 때만 로드됩니다. **이는 화면으로 확인할 수 없습니다.**
- 따라서 **"설치한 mod가 내 deny 규칙을 우회할 수 있다"고 가정**하고 mod를 검토해야 합니다.
- 확실히 적용하려면 이미지에 `/etc/claude-code/managed-settings.json`을 넣으면 됩니다. 그러면 계정과 상관없이 항상 로드됩니다.

### 참고: 공식 플러그인 code-modernization

[anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official)에서 mod를 포함한 유일한 플러그인입니다. 레거시 코드 현대화용이라 추천 목록에는 넣지 않았고, 외부 통신 여부만 확인했습니다.

- mod 코드(`hooks/register.ts` 등)에는 네트워크·모델·프로세스 호출이 없습니다. 단 프롬프트마다 상태 한 줄(400자 이내)을, 레거시 파일 Read 때 분석 정보를 컨텍스트에 붙입니다.
- settings hook이 `scripts/telemetry.sh`를 실행합니다. 스크립트 자체는 네트워크 호출 없이 **숫자만 담은 한 줄**을 출력하고, Claude Code의 telemetry 경로로 넘깁니다. 기본으로 켜져 있고, **플러그인을 쓰지 않아도 설치만 하면** 버전당 한 번 `health` 카운트를 냅니다.
- `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1`(권장 설정)이나 `CODE_MODERNIZATION_TELEMETRY=0`이면 꺼집니다.

## 5. 점검 방법

- 각 저장소를 `--depth 1`로 받아 점검용 폴더에 격리하고, mod 코드를 실행하지 않았습니다.
- `$.http`, `$.process.run`, `$.fs`, `$.env`, `$.model`, `tool.check`, dynamic `import()`, `eval` 사용처를 모두 찾아 읽었습니다.
- blast-radius의 경로 확장 주입 여부는 같은 `compgen -G` 패턴을 별도 bash에서 시험했습니다(mod 코드는 실행하지 않음).
- 새 버전을 반입할 때는 [USAGE.md §5](./USAGE.md#5-보안-체크리스트)대로 `claude plugin validate`를 실행하고 위 표와 비교하세요.
