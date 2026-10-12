# 추천 Mods 상세 사용법 (Linux)

[README](./README.md)에서 소개한 8개 mod의 설치·사용·문제 해결 방법입니다. 코드 점검 결과와 권장 패치는 [SECURITY.md](./SECURITY.md)를 보세요.
팀 환경인 **Linux + 터미널(CLI)** 기준으로 작성했습니다.

> 기준일: 2026-10-12. mod API는 아직 릴리스마다 바뀔 수 있으니, 내용이 다르면 각 mod 저장소의 README를 우선하세요.

## 목차

1. [사전 확인](#1-사전-확인)
2. [설치: 인터넷 가능 환경](#2-설치-인터넷-가능-환경)
3. [설치: 폐쇄망](#3-설치-폐쇄망)
4. [Mod별 사용법](#4-mod별-사용법)
5. [보안 체크리스트](#5-보안-체크리스트)
6. [끄기와 문제 해결](#6-끄기와-문제-해결)

---

## 1. 사전 확인

### 1-1. Claude Code 버전

```bash
claude --version   # 2.1.287 이상이어야 함
```

폐쇄망은 업데이트가 수동이므로 먼저 버전부터 확인하세요.

### 1-2. 회사 관리 설정(managed settings)

IT에서 배포한 관리 설정에 아래 항목이 켜져 있으면 개인 mod가 로드되지 않습니다.

| 설정 | 효과 |
| --- | --- |
| `allowManagedModsOnly` | 조직이 등록한 mod만 허용, 개인 mod 차단 |
| `disableSideloadFlags` | `--plugin-dir` 사용 차단 |
| `disableAllHooks` | 모든 mod와 settings hook 차단 |

설치했는데 mod가 안 보이면 이것부터 의심하고 IT 담당자에게 문의하세요.
반대로 관리 설정이 있는 머신에서는 빌트인 가드 `sec-default`가 먼저 로드되어,
사용자 mod가 deny 규칙이나 관리형 hook을 우회하지 못하므로 더 안전합니다.

### 1-3. 실행 환경

- **터미널에서 `claude`로 실행**해야 패널이 그려집니다. VS Code 확장의 채팅 패널이나 `claude -p`에서는 mod가 동작은 하지만 화면에 아무것도 그려지지 않습니다.
- **터미널 폭 144컬럼 이상**을 권장합니다. 사이드 패널(pane)은 이 폭 이상일 때 대화 옆에 열립니다. 좁으면 프롬프트 위 band로 표시되거나 열리지 않고 대기합니다.

  ```bash
  tput cols   # 현재 터미널 폭 확인
  ```

  tmux를 쓴다면 Claude Code가 있는 pane을 충분히 넓게 잡으세요.
- blast-radius는 `bash`, `git`, `find`, `du`가 PATH에 있어야 합니다. 일반 Linux에는 모두 기본으로 있습니다.

---

## 2. 설치: 인터넷 가능 환경

셸에서 실행합니다. (Claude Code 안에서 `/plugin` 메뉴로 해도 같습니다.)
설치 후 Claude Code를 재시작하거나, 열린 세션에서 `/reload-plugins`를 실행합니다.

### 2-1. mission-control

```bash
claude plugin marketplace add hamzafer/claude-code-mods
claude plugin install mission-control@claude-code-mods
```

### 2-2. change-ledger, turn-timeline

```bash
claude plugin marketplace add arasovic/claude-code-mods
claude plugin install change-ledger@claude-code-mods
claude plugin install turn-timeline@claude-code-mods
```

> **마켓플레이스 이름 충돌 주의:** 2-1(hamzafer)과 2-2(arasovic)는 마켓플레이스 이름이 둘 다 `claude-code-mods`입니다.
> 이미 하나가 등록돼 있으면 다른 하나의 `add`가 충돌합니다.
>
> ```bash
> claude plugin marketplace list                      # 등록된 마켓플레이스 확인
> claude plugin marketplace remove claude-code-mods   # 필요 시 제거
> ```
>
> 둘 다 쓰려면 한쪽은 마켓플레이스로 설치하고, 다른 쪽은 [3장](#3-설치-폐쇄망)의 로컬 폴더 방식으로 로드하세요.

### 2-3. Loop Breaker, Assertion Guardian

```bash
claude plugin marketplace add ccdwyer/claude-mods
claude plugin install loop-breaker@ccdwyer-mods
claude plugin install assertion-guardian@ccdwyer-mods
```

### 2-4. blast-radius

마켓플레이스 파일이 저장소 루트가 아닌 하위 폴더에 있어서, clone한 뒤 경로로 추가합니다.

```bash
git clone https://github.com/anthropics/claude-code-playground.git
cd claude-code-playground/claude-code/mods
claude plugin marketplace add ./
claude plugin install blast-radius@claude-code-playground-mods --scope user
```

> hamzafer 저장소에도 같은 이름의 `blast-radius`가 있지만 **다른 작성자의 다른 프로그램**입니다. 위의 Anthropic 것을 쓰세요.

### 2-5. context-bar

mission-control과 같은 hamzafer 마켓플레이스입니다. 2-1을 했다면 `install`만 하면 됩니다.

```bash
claude plugin marketplace add hamzafer/claude-code-mods   # 2-1에서 했다면 생략
claude plugin install context-bar@claude-code-mods
```

### 2-6. token-ledger

```bash
claude plugin marketplace add Arunjay4213/claude-mods
claude plugin install token-ledger@claude-mods
```

### 2-7. 설치 확인

Claude Code에서 `/plugin`을 열면 탭 아래 흐린 줄에 활성화된 mod 이름이 표시됩니다.

---

## 3. 설치: 폐쇄망

폐쇄망에서는 GitHub 마켓플레이스에 접근할 수 없으므로, **인터넷 PC에서 소스를 받아 반입한 뒤 로컬 폴더로 로드**합니다.
8개 mod 모두 네트워크 호출이 없어 반입 후 정상 동작합니다.

### 3-1. 인터넷 PC에서 준비

```bash
mkdir -p ~/mods-src && cd ~/mods-src
git clone https://github.com/hamzafer/claude-code-mods.git hamzafer-mods
git clone https://github.com/arasovic/claude-code-mods.git arasovic-mods
git clone https://github.com/ccdwyer/loop-breaker.git
git clone https://github.com/ccdwyer/assertion-guardian.git
git clone https://github.com/anthropics/claude-code-playground.git
git clone https://github.com/Arunjay4213/claude-mods.git    arunjay-mods

# 반입할 버전(커밋)을 기록해둔다
for d in */; do echo "$d $(git -C "$d" rev-parse HEAD)"; done > VERSIONS.txt
```

각 mod 폴더는 `.claude-plugin/plugin.json`이 들어 있는 폴더입니다. 저장소마다 위치가 다르니 아래로 찾으세요.

```bash
find ~/mods-src -path '*/.claude-plugin/plugin.json' | sed 's#/.claude-plugin/plugin.json##'
```

반입 전에 각 mod 폴더에 대해 [5장 보안 체크리스트](#5-보안-체크리스트)의 `claude plugin validate`를 실행하고 결과를 함께 보관하세요.

### 3-2. 폐쇄망 PC에 배치

반입한 mod 폴더를 고정 경로에 둡니다. 예:

```
~/claude-mods/
├── mission-control/
├── change-ledger/
├── turn-timeline/
├── loop-breaker/
├── assertion-guardian/
├── blast-radius/
├── context-bar/
└── token-ledger/
```

### 3-3. 로드 방법

**방법 A. 한 세션만 써보기** (처음 테스트할 때 권장)

```bash
claude --plugin-dir ~/claude-mods/mission-control
```

이 방식은 파일을 저장할 때마다 mod를 다시 로드하므로, mod를 수정·개발할 때도 씁니다.

**방법 B. 항상 로드하기**

`CLAUDE_CODE_PLUGIN_DIRS` 환경변수에 mod 폴더 경로를 지정합니다.
`~/.claude/settings.json`의 `env`에 넣으면 매번 적용됩니다.

```json
{
  "env": {
    "CLAUDE_CODE_PLUGIN_DIRS": "/home/<사용자>/claude-mods/mission-control"
  }
}
```

> 여러 폴더를 한 번에 지정하는 구분자 문법은 설치된 버전의 공식 문서([mods overview](https://code.claude.com/docs/en/plugins/mods/overview))에서 확인하세요.

**방법 C. 로컬 마켓플레이스로 설치하기**

마켓플레이스 파일의 플러그인 출처가 **로컬 경로**인 저장소는 받아온 폴더를 그대로 마켓플레이스로 등록할 수 있습니다.
hamzafer, arasovic, Arunjay4213 저장소 루트와 Anthropic의 `claude-code/mods` 폴더가 여기에 해당합니다.

> **Loop Breaker, Assertion Guardian은 이 방법이 안 됩니다.** ccdwyer 마켓플레이스는 플러그인 출처가 GitHub로 적혀 있어
> 폐쇄망에서 설치에 실패합니다. 이 둘은 방법 A나 B로 폴더를 직접 로드하세요. ccdwyer의 `install.sh`도 GitHub에 접속하므로 쓰지 마세요.

```bash
cd ~/mods-src/claude-code-playground/claude-code/mods
claude plugin marketplace add ./
claude plugin install blast-radius@claude-code-playground-mods --scope user
```

### 3-4. 팀 배포 권장 방식

1. 한 사람이 검토(3-1, 5장)를 끝낸 버전을 **사내 Git 저장소**에 올립니다.
2. 팀원은 그 사본만 받아서 3-2, 3-3대로 로드합니다.
3. 업데이트할 때도 같은 검토를 다시 거칩니다.

---

## 4. Mod별 사용법

### 4-1. mission-control: 에이전트·툴 호출 실시간 트리

**명령:** `/mission` (열기) · `w` Who 뷰 · `q` 닫기

**Who 뷰에 표시되는 것**

- 메인 에이전트 1행(내 프롬프트 앞부분이 이름으로 붙음)
- 서브에이전트마다 1행: 실행 시간, 툴 호출 수
- 툴 호출마다 1행: Bash 명령 설명, 대상 파일, 검색 패턴, MCP 액션
- 실행 중인 호출은 **노란색**, 실패한 호출은 **빨간색**
- 같은 부모 아래 연속된 같은 종류의 호출은 한 줄로 접혀서 표시

**프롬프트 위 band:** Claude가 작업하는 동안과 끝난 뒤 30초간, 실행 중/전체 에이전트 수, 툴 호출 수, 실패 수, 변경 파일 수를 보여줍니다.

**Linux에서 주의할 점**

- **Code 뷰(`c`, `/mission code`)는 Linux에서 동작하지 않습니다.** 파일 맵을 그리려면 macOS, `/Applications`의 Google Chrome, 이미지 표시 터미널이 필요합니다. Linux에서는 Code 뷰를 열면 지도 대신 안내 메시지가 나옵니다. Who 뷰와 band는 정상 동작합니다.
- **파일을 수정한 턴이 끝날 때마다 수정 코드 조각을 Haiku로 보냅니다.** 이 요약은 Code 뷰 카드에만 쓰이는데 Linux에서는 Code 뷰가 그려지지 않으므로, 비용만 들고 쓸모가 없습니다. 끄는 설정이 없으니 반입 전에 `hooks/register.tsx`의 아래 한 줄을 지우세요. ([SECURITY.md](./SECURITY.md#mission-control-modsmission-controlhooksregistertsx) 참고)

  ```diff
  -      void explain($, t, new Map(changes)).catch(() => {})
  ```
- 트리는 최대 300행까지 유지하고, 넘으면 오래된 툴 호출부터 버립니다.
- 다음 턴이 시작될 때 아직 실행 중인 백그라운드 서브에이전트는 새 트리로 이어집니다.

**이럴 때 유용:** 서브에이전트를 여러 개 띄운 작업에서 "지금 누가 뭘 하고 있는지", "어디서 실패했는지"를 한눈에 볼 때.

### 4-2. change-ledger: 세션에서 바뀐 파일 목록

**명령:** `/changes`

**표시되는 것**

- **상단 줄:** 변경 파일 수, 총 추가/삭제 줄 수, 브랜치와 ahead/behind
- **Edits:** 최신순. 파일마다 diffstat 막대, 새로 만든 파일이면 `new` 태그, 폴더, 수정한 에이전트(`main` 또는 서브에이전트 종류), 수정 횟수, 마지막 수정 후 경과 시간
- **Working tree:** `git status` 결과. 세션이 수정한 파일은 `●`, 그 외(내가 직접 고친 것, 포매터, 셸 명령 등)는 `○`. 8개까지 표시하고 나머지는 개수로 접힘

**주의할 점**

- **Edit/Write 툴로 바뀐 파일만 Edits에 기록됩니다.** `sed`, 레지스터 맵 생성 스크립트, 코드 생성기 등 셸 명령으로 바뀐 파일은 Edits에 없고 Working tree에만 `○`로 나옵니다. RTL·펌웨어에서 생성 스크립트를 많이 쓴다면 Working tree 쪽을 같이 보세요.
- **git 저장소 안에서 Claude Code를 시작해야** Working tree가 보입니다.
- ahead/behind는 마지막 `git fetch` 기준입니다(원격에 접속하지 않음).

**이럴 때 유용:** 작업이 끝난 뒤 커밋 전에 "Claude가 정확히 뭘 바꿨는지", "내 변경과 Claude 변경이 섞이지 않았는지" 확인할 때.

### 4-3. turn-timeline: 턴의 시간이 어디로 갔는지

**명령:** `/timeline` (설치 후 **다음 턴부터** 기록)

**표시되는 것**

- **상단 줄:** 턴 경과 시간, 모델 요청·툴·대기 시간 비율
- **Timeline:** 메인 루프 1레인 + 서브에이전트마다 1레인. 초록 블록은 툴 호출, 빨강은 실패, 음영은 모델 요청, 점은 대기
- **Where time went:** 메인 루프 시간을 모델·툴별·대기로 나눈 초와 비율
- **Slowest:** 가장 오래 걸린 3단계와 대상(명령, 파일, 패턴)

**주의할 점**

- Where time went는 메인 루프만 집계합니다. 서브에이전트는 타임라인에만 나옵니다.
- 한 턴에 최근 400단계까지 유지합니다.

**이럴 때 유용:** 시뮬레이션, 합성, 빌드처럼 오래 걸리는 작업에서 "모델이 느린 건지, 툴(빌드)이 느린 건지" 병목을 확인할 때.

### 4-4. blast-radius: 위험 명령 실행 전 확인

**명령:** 없음 (자동 동작). 위험 명령을 감지하면 멈추고 대기합니다.

| 감지하는 명령 | 보여주는 내용 |
| --- | --- |
| `rm`에 `-r` 또는 `-f` | 지워질 파일 수, 총 크기, 처음 10개 파일 |
| `git reset --hard` | 변경 사항이 있는 파일, `git diff --shortstat` |
| `git checkout -- .`, `git restore .` | unstaged 변경이 있는 파일 |
| `git clean` | dry run으로 지워질 untracked 경로 |
| `git push --force`, `-f`, `--force-with-lease`, `+ref` | 원격 브랜치에서 사라질 커밋 |
| 마이그레이션(`manage.py migrate`, `alembic upgrade` 등) | 대기 중인 마이그레이션 |

**조작**

- `1` 또는 Proceed: 명령 전체를 그대로 실행
- `2` 또는 Cancel: 거부. Claude에게 이유와 영향 요약이 전달되고, 다시 시도하지 말라고 안내됨
- 창이 열리면 **Cancel에 포커스**가 있어서 Enter를 누르면 거부됩니다
- 10분간 응답이 없거나 Esc로 턴을 중단하면 거부됩니다
- 한 번에 하나만 대기합니다. 서브에이전트의 두 번째 위험 명령은 첫 번째가 처리된 뒤에 뜹니다

**주의할 점**

- **안전망이지 권한 시스템이 아닙니다.** `bash -c "..."`, `$(...)`, alias, `xargs rm`, `find -delete`, `rm`을 호출하는 스크립트, `timeout 5 rm` 같은 래퍼는 잡지 못합니다. 반드시 막아야 할 명령은 permission deny 규칙으로 막으세요.
- 한 줄에 여러 명령이 있으면 첫 번째 위험 부분만 측정하지만, Proceed는 줄 전체를 실행합니다.
- **band는 한 번에 한 mod만 쓸 수 있습니다.** 터미널이 144컬럼 미만이면 blast-radius가 band에 그려지는데, mission-control 등 다른 mod가 band를 쓰고 있으면 버튼이 안 보이고 10분 뒤 거부됩니다. **터미널을 넓게 쓰면** 사이드 패널로 열려서 이 문제가 없습니다.
- 처음엔 버려도 되는 디렉터리에서 `rm -rf build` 같은 명령을 Claude에게 시켜 동작을 확인해 보세요.

### 4-5. Loop Breaker: 같은 실패 반복 방지

**명령:** 자동 동작. 해제는 `/unstick` 또는 band의 `unstick` 버튼.

**동작**

- **반복 실패:** 같은 인자의 같은 호출이 **같은 에러로 두 번 연속** 실패하면 세 번째 시도를 거부합니다. 파일이 바뀌거나(Edit/Write 성공, 파일을 바꾼 셸 명령, 서브에이전트의 변경), 다른 에러가 나거나, 성공하면 기록이 초기화됩니다. 그래서 일반적인 "고치고 다시 실행" 루프는 막지 않습니다.
- **왕복 수정:** 파일을 직전 버전으로 되돌리는 수정이 3번 연속되면 4번째를 거부합니다.
- **stuck meter:** 두 번째 같은 실패부터 프롬프트 위에 `stuck: make sim failed 2× in a row` 같은 줄이 표시됩니다.
- 거부되면 Claude에게 "원인(코드, 설정, 누락된 설치)을 먼저 고치거나 사용자에게 물어보라"고 전달됩니다.

**초기화되는 경우:** `/unstick`, 내가 새 프롬프트를 보낼 때, `/clear`, 세션 재개. (백그라운드 알림이나 예약 프롬프트로는 초기화되지 않음)

**주의할 점**

- 에러 비교 시 타임스탬프, 소요 시간, `/tmp` 경로, PID 등은 제거하고 비교합니다. 시뮬레이터 로그에 매번 바뀌는 다른 값(시드, 사이클 수 등)이 섞이면 "다른 에러"로 보고 막지 않을 수 있습니다.
- 한 메시지에서 병렬로 보낸 호출들은 반복 여부를 잡지 못합니다.
- 파일 내용은 저장하지 않고 해시와 개수만 메모리에 둡니다.

**이럴 때 유용:** 합성·컴파일·테스트 에러를 Claude가 같은 방식으로 계속 재시도하며 시간과 사용량을 쓰는 상황.

### 4-6. Assertion Guardian: 테스트 약화 방지

**명령:** 자동 동작. 1회 허용은 `/allow-test-change [파일명]` 또는 band의 `allow once`.

**거부하는 변경**

- 테스트 케이스나 assertion 개수가 줄어듦, assertion을 주석 처리
- 기대값 변경(`assert f() == 1` → `== 2`), 더 느슨한 matcher로 교체
- skip/xfail/todo 추가, 다른 테스트를 빼버리는 `.only`
- 항상 참인 assertion(`assert True`), 실패를 삼키는 try/except
- 테이블 테스트의 데이터 행 삭제·변경
- 스냅샷 파일 수정, 전체 스냅샷 승인 명령
- 테스트 파일을 쓰거나 지우는 셸 명령(`sed -i`, `rm`, `>` 리다이렉트 등)

이름 변경, 포맷 변경, 순서 변경, 테스트 추가, 더 엄격한 matcher로 교체는 통과합니다.
거부되면 Claude에게 "테스트가 아니라 테스트 대상 코드를 고치고, 테스트가 틀렸다고 생각하면 사용자에게 물어보라"고 전달됩니다.

**1회 허용:** 스펙이 바뀌어 기대값을 정말 바꿔야 할 때 `allow once`를 누른 뒤 Claude가 **똑같은 수정**을 다시 시도하면 통과합니다. 다른 수정은 다시 검사합니다.

**RTL 팀에서 꼭 알아둘 점**

- 테스트 파일을 **파일 이름**으로 판별합니다. Python(`test_*.py`, `*_test.py`, `tests.py`), JS/TS, Go, Ruby, Java, Swift, Rust, PHP 패턴만 압니다.
- **SystemVerilog/Verilog testbench(`tb_*.sv` 등)와 C 유닛테스트는 인식하지 않습니다.** SV `assert property` 같은 구문도 검사 대상이 아닙니다.
- **cocotb 테스트(`test_*.py`)는 Python `assert`로 인식되어 보호됩니다.**
- SV testbench까지 보호하려면 이 mod를 참고해 팀용 버전을 만드는 것을 권장합니다.
- **저장소 경로 주의:** 폴더 규칙을 절대경로 전체에 적용하므로, 저장소가 `/data/...` 아래에 있으면 폴더 기준 판별이 꺼집니다. 가능하면 `data`, `support`, `helpers`, `test` 같은 이름이 상위 경로에 없는 곳에 저장소를 두세요.

### 4-7. context-bar: 컨텍스트를 카테고리별 막대로

**명령:** `/context-bar` (숨기기/보이기, 선택은 다음 세션에도 유지)

**표시되는 것**

- 헤더: 사용 중인 토큰 / 전체 창 크기, 자동 compaction 시작 지점, 사용률 배지
- 배지 색: compaction 지점 대비 **70% 이상 노랑, 90% 이상 빨강**
- 막대: `/context`와 같은 분류(시스템 프롬프트, 시스템 툴, MCP 툴, 메모리 파일, skill, 메시지)별 색. 계속 늘어나는 "messages"는 주황색
- 범례: 카테고리별 토큰 수와 비율

**갱신 시점:** 세션 시작, 내 턴이 끝날 때, `/compact` 후. 서브에이전트 턴은 반영하지 않습니다.

**이럴 때 유용:** 대화를 시작하기도 전에 컨텍스트가 크게 차 있다면 대부분 MCP 툴 정의나 메모리 파일입니다. 막대에서 "mcp tools"가 크면 프로젝트에서 안 쓰는 MCP 서버를 빼세요. 매 요청마다 함께 보내지는 고정 비용이 줄어듭니다.

### 4-8. token-ledger: 턴별 비용

**명령:** `/ledger` (패널 열기/닫기)

**표시되는 것**

- 프롬프트 아래 한 줄: 세션 비용, 직전 턴의 토큰과 캐시 비율
- `/ledger` 패널: 턴마다 모델, 새 입력 토큰, 출력 토큰, 캐시 읽기/쓰기, 적중률, 소요 시간, 비용. 서브에이전트 턴은 `*` 표시. 최근 40행 표시

**주의할 점**

- 금액은 Claude Code가 계산한 세션 비용의 변화량입니다. **`modelPricing` 설정이 없으면 정가 기준 추정치**라 실제 엔터프라이즈 청구액과 다를 수 있습니다. 회사 계약 단가가 있다면 `modelPricing`을 설정하세요.
- 서브에이전트 비용이 메인 턴 행에 섞일 수 있습니다.
- 첫 턴은 기준값이 잡힌 뒤부터 금액이 계산됩니다.

**이럴 때 유용:** 캐시 적중률이 갑자기 떨어지는 턴(모델 변경, 오래 쉰 뒤 등)을 찾아 비용이 튀는 원인을 확인할 때.

### 4-9. mod 없이 status line에 context 표시

context 사용률을 숫자로만 보고 싶다면 mod가 필요 없습니다. 기본 status line 스크립트가 받는 JSON에
`context_window.used_percentage`와 `cost.total_cost_usd`가 들어 있습니다. mod가 아니라서 보안 검토 부담도 없습니다.

```json
// ~/.claude/settings.json
{
  "statusLine": {
    "type": "command",
    "command": "jq -r '\"[\\(.model.display_name)] ctx \\(.context_window.used_percentage // 0)% · $\\(.cost.total_cost_usd // 0)\"'"
  }
}
```

- 서버에 `jq`가 설치돼 있어야 합니다.
- Claude Code 안에서 `/statusline`을 입력하고 원하는 표시를 설명해도 대신 설정해 줍니다.
- 사내 게이트웨이를 쓰면 `rate_limits.spend_limit.used_percentage`로 지출 한도 사용률도 표시할 수 있습니다. ([status line 문서](https://code.claude.com/docs/en/statusline))

---

## 5. 보안 체크리스트

mod는 **샌드박스 없이** 내 권한으로 Claude Code 안에서 실행됩니다.
파일·환경변수 읽기, 모든 프롬프트와 툴 호출 열람, 툴 호출 대신 승인, 모델 사용량 소비가 가능합니다.
관리 설정이 없는 PC에서는 설치한 mod가 내 deny 규칙을 우회할 수 있다고 가정하세요.
또한 deny 규칙은 Claude를 제한할 뿐, mod 자신의 파일 접근(`$.fs`)과 프로세스 실행(`$.process`)은 막지 않습니다.

### 설치 전 검사

```bash
claude plugin validate ./<mod 폴더>
```

코드를 실행하지 않고 읽기만 해서, `hooks:`(받는 이벤트)와 `calls:`(호출하는 API)를 출력합니다.

| `calls:`에서 볼 항목 | 의미 | 확인할 것 |
| --- | --- | --- |
| `$.http.fetch` | 네트워크 접속 | 8개 추천 mod에는 없어야 정상 |
| `$.process.run` | 프로그램 실행 | git, find, du 등 기능에 맞는 명령인지 |
| `$.env.get` | 환경변수 읽기 | 키·토큰을 읽을 이유가 있는지 |
| `tool.check` | 권한 판단 덮어쓰기 | 관찰용 mod라면 없어야 정상 |
| `$.model.complete`, `$.model.fork` | 모델 호출(사용량 소비) | mission-control의 요약 기능 외에는 없어야 정상 |

### 체크리스트

- [ ] `claude plugin validate` 결과를 읽고 기능과 맞지 않는 호출이 없는지 확인했다
- [ ] 검토한 커밋 해시를 기록하고 그 버전만 반입했다
- [ ] 런타임에 외부 코드를 내려받는 부분이 없는지 확인했다
- [ ] 팀 배포본은 사내 저장소에 올리고, 업데이트도 같은 검토를 거친다

---

## 6. 끄기와 문제 해결

### 끄기

| 범위 | 방법 |
| --- | --- |
| mod 하나 | `/plugin` → Installed 탭에서 비활성화 |
| 이번 세션만 전부 | `claude --safe-mode` |
| 항상 전부 | `~/.claude/settings.json`에 `"disableAllHooks": true` (기존 settings hook도 같이 꺼짐) |

### 문제 해결

| 증상 | 확인할 것 |
| --- | --- |
| `/plugin`에 mod가 안 보임 | `claude --version` ≥ 2.1.287, 관리 설정(1-2), 설치 후 재시작 또는 `/reload-plugins` |
| 패널이 안 열림 / "waiting" 메시지 | 터미널 폭(`tput cols`) 144 이상인지, VS Code 채팅 패널이 아닌 터미널인지 |
| `/mission code`에 지도가 안 나옴 | Linux에서는 정상. Who 뷰(`w`)를 사용 |
| change-ledger Edits에 파일이 빠짐 | 셸 명령으로 바뀐 파일은 Working tree에만 표시됨 |
| blast-radius 버튼이 안 보이고 10분 뒤 거부 | 다른 mod가 band를 쓰는 중. 터미널을 넓혀 사이드 패널로 열리게 하기 |
| `marketplace add`가 충돌 | 같은 이름(`claude-code-mods`) 마켓플레이스가 이미 있음(2-2 참고) |
| Claude 동작이 이상해짐 | `claude --safe-mode`로 mod 없이 재현되는지 확인 후 mod를 하나씩 켜서 원인 찾기 |

## 참고

- [Claude Code mods 공식 문서](https://code.claude.com/docs/en/plugins/mods/overview)
- [mods API 레퍼런스](https://code.claude.com/docs/en/plugins/mods/reference)
- [관리자용 mod 설정](https://code.claude.com/docs/en/plugins/mods/admin)
- 각 mod 상세: [mission-control](https://claude-mods.com/mission-control/) · [change-ledger](https://claude-mods.com/change-ledger/) · [turn-timeline](https://claude-mods.com/turn-timeline/) · [blast-radius](https://claude-mods.com/blast-radius/) · [Loop Breaker](https://claude-mods.com/loop-breaker/) · [Assertion Guardian](https://claude-mods.com/assertion-guardian/) · [context-bar](https://claude-mods.com/context-bar/) · [token-ledger](https://claude-mods.com/token-ledger/)
