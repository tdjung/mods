# 추천 Mods 상세 사용법 (Linux)

[README](./README.md)에서 소개한 추천 mod의 설치·사용·문제 해결 방법입니다. 코드 점검 결과와 권장 패치는 [SECURITY.md](./SECURITY.md)를 보세요.
팀 환경인 **Linux + 터미널(CLI)** 기준으로 작성했습니다.

> 기준일: 2026-10-12. mod API는 아직 릴리스마다 바뀔 수 있으니, 내용이 다르면 각 mod 저장소의 README를 우선하세요.

## 목차

1. [사전 확인](#1-사전-확인)
2. [설치: 팀 카탈로그(team-mods)](#2-설치-팀-카탈로그team-mods)
3. [설치: 폐쇄망](#3-설치-폐쇄망)
4. [Mod별 사용법](#4-mod별-사용법)
5. [보안 체크리스트](#5-보안-체크리스트)
6. [끄기, 제거, 문제 해결](#6-끄기-제거-문제-해결)
7. [참고: 원작자 카탈로그에서 직접 설치](#7-참고-원작자-카탈로그에서-직접-설치)

---

## 1. 사전 확인

### 1-1. Claude Code 버전

```bash
claude --version   # 2.1.287 이상이어야 함
```

폐쇄망은 업데이트가 수동이므로 먼저 버전부터 확인하세요.

### 1-2. 실행 환경

- **터미널에서 `claude`로 실행**해야 패널이 그려집니다. VS Code 확장의 채팅 패널이나 `claude -p`에서는 mod가 동작은 하지만 화면에 아무것도 그려지지 않습니다.
- **터미널 폭 144컬럼 이상**을 권장합니다. 사이드 패널(pane)은 이 폭 이상일 때 대화 옆에 열립니다. 좁으면 프롬프트 위 band로 표시되거나 열리지 않고 대기합니다.

  ```bash
  tput cols   # 현재 터미널 폭 확인
  ```

  tmux를 쓴다면 Claude Code가 있는 pane을 충분히 넓게 잡으세요.
- blast-radius는 `bash`, `git`, `find`, `du`가 PATH에 있어야 합니다. 일반 Linux에는 모두 기본으로 있습니다.

### 1-3. 외부 전송 끄기 (권장)

팀 컨테이너는 `ANTHROPIC_BASE_URL`로 사내 게이트웨이에 연결합니다. 이 방식은 빌트인 `telemetry` mod가 "분석 꺼짐"으로 판단하는 조건에 들어가지 않아서,
분석 이벤트를 `api.anthropic.com`으로 보내려다 폐쇄망이라 실패합니다. 동작에는 문제가 없지만 확실히 끄려면 **Docker 이미지 환경변수**에 아래를 넣으세요.

```dockerfile
ENV CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1
```

telemetry와 함께 Claude Code의 다른 비필수 외부 통신도 꺼집니다. 게이트웨이로 가는 모델 요청은 필수 통신이라 영향이 없고, 꺼지는 것들은 폐쇄망에서 어차피 실패하는 통신입니다.
자세한 근거는 [SECURITY.md §4](./SECURITY.md#4-빌트인-mods)를 보세요.

---

## 2. 설치: 팀 카탈로그(team-mods)

추천 mod 9개는 이 저장소의 `plugins/` 폴더에 **점검한 버전 그대로** 들어 있고, mission-control과 change-ledger는 **패치가 이미 적용돼** 있습니다.
저장소를 받아 카탈로그로 한 번 등록하면, 원작자 저장소를 따로 찾을 필요 없이 `install`만 하면 됩니다. (token-ledger는 라이선스 파일이 없어 넣지 않았습니다. 2-6 참고)

출처, 커밋, 패치 내용은 [plugins/README.md](./plugins/README.md)에 있습니다.

### 2-1. 저장소 받기

```bash
git clone https://github.com/tdjung/mods.git ~/team-mods   # 폐쇄망은 사내 Git 미러 주소로 (3장)
```

> **이 폴더를 지우거나 옮기지 마세요.** 폴더로 등록한 카탈로그는 복사본을 만들지 않고 **이 폴더를 그대로 읽어서** 실행합니다.

### 2-2. 카탈로그 등록 (한 번만)

```bash
claude plugin marketplace add ~/team-mods
claude plugin marketplace list    # team-mods 가 보이면 성공
```

### 2-3. 원하는 mod 설치

```bash
# 처음 시작 추천
claude plugin install context-bar@team-mods
claude plugin install blast-radius@team-mods

# 필요하면 추가
claude plugin install mission-control@team-mods      # 패치 적용본
claude plugin install change-ledger@team-mods        # 패치 적용본
claude plugin install turn-timeline@team-mods
claude plugin install loop-breaker@team-mods
claude plugin install assertion-guardian@team-mods
claude plugin install token-weather@team-mods        # 다른 mod 표시를 가릴 수 있음
claude plugin install replay-theater@team-mods       # 다른 mod 표시를 가릴 수 있음
```

설치 후 열려 있는 Claude Code에서 `/reload-plugins`를 실행하면 재시작 없이 반영됩니다. 세션 안의 `/plugin` 화면으로 설치했다면 화면을 닫을 때 자동으로 반영됩니다.

> token-weather와 replay-theater는 프롬프트 위 영역에서 다른 mod의 표시를 가릴 수 있습니다. 함께 쓰면 터미널을 144컬럼 이상으로 쓰세요. ([SECURITY.md](./SECURITY.md#공통-주의-band프롬프트-위-영역를-독차지하는-mod))

### 2-4. 설치 확인

```bash
claude plugin list
```

mod마다 `Version`, `Read from`(읽는 폴더), `Status`가 나옵니다. 패치본은 버전이 `-team.1`로 끝납니다(예: `mission-control` `0.1.2-team.1`).

### 2-5. 업데이트

카탈로그가 저장소 폴더를 그대로 읽기 때문에, 저장소를 최신으로 받기만 하면 됩니다.

```bash
cd ~/team-mods && git pull
```

그다음 열려 있는 세션에서 `/reload-plugins`를 실행하세요. 새 버전은 저장소 관리자가 점검과 패치를 마친 뒤에만 올라옵니다. ([plugins/README.md](./plugins/README.md#원본을-새-버전으로-올릴-때))

### 2-6. token-ledger (카탈로그에 없음)

README에는 MIT라고 적혀 있지만 라이선스 파일이 없어 팀 저장소에 복사하지 않았습니다. 쓰려면 원작자([Arunjay4213/claude-mods](https://github.com/Arunjay4213/claude-mods)) 카탈로그에서 직접 설치합니다(인터넷 필요).

```bash
claude plugin marketplace add Arunjay4213/claude-mods
claude plugin install token-ledger@claude-mods
```

### 2-7. 원작자 카탈로그에서 직접 설치하려면

팀 카탈로그의 mod는 모두 원작자의 작업을 가져온 것입니다. 원작자 카탈로그에서 최신 버전을 직접 설치하는 방법은 [7장](#7-참고-원작자-카탈로그에서-직접-설치)에 그대로 남겨 두었습니다.
다만 버전이 고정되지 않아 점검하지 않은 코드가 설치될 수 있고, 팀 패치도 적용되지 않습니다.

---

## 3. 설치: 폐쇄망

팀 카탈로그는 mod 코드가 모두 저장소 안에 있어서, **저장소만 반입하면 설치 중에 외부에 접속하지 않습니다.**
추천 mod 모두 네트워크 호출이 없어 반입 후 정상 동작합니다.

### 3-1. 사내 Git에 미러링 (권장)

인터넷 PC에서 한 번 미러를 만들고 사내 Git으로 올립니다.

```bash
git clone --mirror https://github.com/tdjung/mods.git
cd mods.git && git push --mirror <사내 Git 주소>/mods.git
```

이후 팀원은 2장과 같은 방법을 쓰되, 2-1의 주소만 사내 Git으로 바꿉니다.

사내 Git이 없다면 `git bundle create team-mods.bundle --all`로 파일 하나로 만들어 반입한 뒤 `git clone team-mods.bundle ~/team-mods`로 풀어도 됩니다.

### 3-2. Docker 이미지에 넣기 (선택)

팀원 모두 Docker를 쓰므로, 이미지에 저장소를 넣어 두면 받는 단계가 사라집니다.

```dockerfile
RUN git clone <사내 Git 주소>/mods.git /opt/team-mods
ENV CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1
```

팀원은 컨테이너 안에서 `claude plugin marketplace add /opt/team-mods`와 `install`만 하면 됩니다.
카탈로그 등록 정보는 `~/.claude` 아래에 저장되므로, 컨테이너를 새로 만들 때마다 지워진다면 `~/.claude`를 볼륨으로 유지하세요.

### 3-3. 카탈로그 없이 폴더로 바로 로드

설치하지 않고 이번 실행에만 쓰려면 `--plugin-dir`를 씁니다. **여러 번 붙일 수 있습니다.**

```bash
claude --plugin-dir ~/team-mods/plugins/context-bar --plugin-dir ~/team-mods/plugins/blast-radius
```

- 이 방식은 파일을 저장할 때마다 mod를 다시 로드하므로, mod를 수정·개발할 때도 씁니다.
- 같은 이름의 mod가 설치돼 있으면 `--plugin-dir` 쪽이 우선합니다.
- 항상 로드하려면 `CLAUDE_CODE_PLUGIN_DIRS` 환경변수를 쓸 수 있습니다. 여러 폴더를 지정하는 구분자는 설치된 버전의 [공식 문서](https://code.claude.com/docs/en/plugins/mods/overview)에서 확인하세요.

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
- **팀 카탈로그 버전은 패치돼 있습니다**(`0.1.2-team.1`). 원본은 파일을 수정한 턴이 끝날 때마다 수정 코드 조각을 Haiku로 보내는데, 이 요약은 macOS 전용 Code 뷰에만 쓰여서 Linux에서는 비용만 들었습니다. 팀 버전에서는 이 호출을 지웠습니다. ([patches/mission-control.patch](./patches/mission-control.patch), [SECURITY.md](./SECURITY.md#mission-control-modsmission-controlhooksregistertsx))
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

- **팀 카탈로그 버전은 패치돼 있습니다**(`0.1.5-team.1`). git 명령이 최대 5초만 기다리고, `git status`가 추적하지 않는 파일을 훑지 않습니다(`-uno`). 그래서 셸 명령으로 **새로 만든** 파일은 Working tree에 나오지 않습니다. ([patches/change-ledger.patch](./patches/change-ledger.patch))
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

- 금액은 Claude Code가 계산한 세션 비용의 변화량입니다. **`modelPricing` 설정이 없으면 정가 기준 추정치**라 실제 엔터프라이즈 청구액과 다를 수 있습니다.
- `modelPricing`은 **managed settings에서만 읽힙니다.** 각자의 `~/.claude/settings.json`에 넣으면 무시되므로, Docker 이미지의 `/etc/claude-code/managed-settings.json`에 넣어야 합니다. 게이트웨이 뒤의 모델 ID(open weight 포함)를 키로 써서 사내 단가를 지정할 수 있습니다. ([설정 문서](https://code.claude.com/docs/en/settings-reference))
- 게이트웨이가 open weight 모델에 Claude 모델 이름을 붙여 쓰면 Claude 정가로 계산돼 금액이 부풀려 보입니다. 금액보다 **토큰 수와 캐시 적중률**을 기준으로 보는 걸 권장합니다.
- 서브에이전트 비용이 메인 턴 행에 섞일 수 있습니다.
- 첫 턴은 기준값이 잡힌 뒤부터 금액이 계산됩니다.

**이럴 때 유용:** 캐시 적중률이 갑자기 떨어지는 턴(모델 변경, 오래 쉰 뒤 등)을 찾아 비용이 튀는 원인을 확인할 때.

### 4-9. token-weather: 컨텍스트 사용률 한 줄 요약

**명령:** 없음 (자동 표시)

프롬프트 위 한 줄에 날씨 아이콘, 사용률, 사용 토큰 / 전체 창, 최근 12턴 추이 그래프, 직전 턴 증가량을 보여줍니다.

| 사용률 | 표시 |
| --- | --- |
| 25% 미만 | ☀ Clear |
| 25~49% | ☁ Cloudy |
| 50~74% | ☂ Showers |
| 75~89% | ☇ Storm |
| 90% 이상 | ↯ Compact soon |

- 사용률은 **전체 창 크기** 기준입니다. compaction 지점 기준인 Claude Code 알림이나 context-bar보다 낮게 나옵니다.
- 턴이 끝날 때만 갱신됩니다.
- **context-bar와의 차이:** token-weather는 "얼마나, 얼마나 빨리" 차는지, context-bar는 "무엇이" 차지하는지 보여줍니다. 비용 원인을 찾으려면 context-bar가 낫습니다.

### 4-10. replay-theater: 직전 턴 수정 다시 보기

**명령:** `/replay` (또는 프롬프트 위 `Replay` 버튼, 단축키 `r`)

- 직전 턴에서 Claude가 Edit·Write·MultiEdit로 바꾼 내용을 한 단계에 diff 하나씩 보여줍니다.
- 패널에서 `p` 이전, `n` 다음, `c` 또는 Esc 닫기.
- 툴 실행 전에 기록하므로, **거부되거나 실패한 수정도 목록에 나옵니다.**
- change-ledger가 "세션 전체에서 어떤 파일이 얼마나" 바뀌었는지라면, replay-theater는 "직전 턴에서 정확히 무엇을" 바꿨는지 봅니다.

### 4-11. 내장 mod 설정: agents-md 끄기 (권장)

`agents-md`는 설치하지 않아도 켜져 있습니다. `CLAUDE.md`가 없는 프로젝트에서는 `/`부터 작업 폴더까지의 모든 `AGENTS.md`를 지시로 넣습니다.
받아온 외부 저장소나 벤더 IP·SDK 폴더 안의 `AGENTS.md`가 지시가 되는 걸 막으려면 `~/.claude/settings.json`에 아래를 넣거나 `/config`의 "Project instructions"를 `claude-md`로 바꾸세요.

```json
{
  "pluginConfigs": {
    "agents-md@builtin": { "options": { "instructionFiles": "claude-md" } }
  }
}
```

프로젝트의 `.claude/settings.json`에서는 이 설정을 읽지 않습니다. 근거는 [SECURITY.md](./SECURITY.md#agents-md)를 보세요.

### 4-12. mod 없이 status line에 context 표시

context 사용률을 숫자로만 보고 싶다면 mod가 필요 없습니다. 기본 status line 스크립트가 받는 JSON에
`context_window.used_percentage`와 `cost.total_cost_usd`가 들어 있습니다. mod가 아니라서 보안 검토 부담도 없습니다.

> **팀 컨테이너에서는 `statusLine`을 덮어쓰지 마세요.** 이미 게이트웨이 스크립트(`~/.local/share/gate.../statusline`)가
> 설정돼 있습니다. 새 명령으로 바꾸면 그 스크립트가 보여주던 정보가 사라집니다. 아래처럼 **감싸서 덧붙이세요.**

**1. 기존 스크립트를 감싸는 래퍼 만들기**

```bash
cat > ~/.claude/statusline-wrap.sh <<'EOF'
#!/usr/bin/env bash
input=$(cat)
orig=$(printf '%s' "$input" | ~/.local/share/gate.../statusline)   # settings.json에 있던 실제 경로로 바꾸기
ctx=$(printf '%s' "$input" | jq -r '.context_window.used_percentage // empty')
printf '%s%s\n' "$orig" "${ctx:+ · ctx ${ctx}%}"
EOF
chmod +x ~/.claude/statusline-wrap.sh
```

**2. `~/.claude/settings.json`의 `statusLine.command`만 래퍼로 교체**

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline-wrap.sh"
  }
}
```

기존 표시 뒤에 `· ctx 23%`가 붙습니다. 되돌리려면 `command`를 원래 경로로 바꾸면 됩니다.

- 컨테이너에 `jq`가 설치돼 있어야 합니다.
- 비용도 붙이려면 `.cost.total_cost_usd`를 같은 방식으로 읽으면 됩니다. 단 팀 환경에는 `modelPricing`이 없어서 **정가 기준 추정치**입니다([4-8](#4-8-token-ledger-턴별-비용) 참고).
- 필드 목록은 [status line 문서](https://code.claude.com/docs/en/statusline)를 보세요. `rate_limits.spend_limit`은 Claude apps gateway 전용이라, `ANTHROPIC_BASE_URL`로 연결하는 팀 환경에서는 값이 없을 가능성이 큽니다.

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
| `$.http.fetch` | 네트워크 접속 | 추천 mod에는 없어야 정상 |
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

## 6. 끄기, 제거, 문제 해결

마음에 들지 않으면 **먼저 끄고(disable)**, 다시 쓸 일이 없으면 **제거(uninstall)** 하세요. 끄기는 설치를 그대로 두므로 `enable`로 바로 되돌릴 수 있습니다.
명령은 셸에서 실행하고, 끝나면 Claude Code를 다시 시작하거나 열린 세션에서 `/reload-plugins`를 실행하세요.

### 6-1. 잠깐 끄기 / 다시 켜기

```bash
claude plugin disable token-weather@team-mods
claude plugin enable  token-weather@team-mods
```

- Claude Code 안에서는 `/plugin` → Installed 탭에서 끄고 켤 수 있습니다.
- 이름만(`token-weather`) 써도 되지만, 같은 이름이 여러 카탈로그에 있으면 `이름@카탈로그`로 써야 합니다.
- 설치된 목록과 정확한 이름은 `claude plugin list`로 확인합니다.

### 6-2. 완전히 제거하기

```bash
claude plugin uninstall token-weather@team-mods
```

- 기본으로 `user` 범위에서 지웁니다. 다른 범위에 설치했다면 `--scope project` 처럼 지정하세요.
- mod가 저장한 데이터(`~/.claude/plugins/data/<id>/`)도 **함께 삭제됩니다.** 설정이나 기록을 남기려면 `--keep-data`를 붙이세요.

추천 mod별 제거 명령:

| Mod | 명령 |
| --- | --- |
| blast-radius | `claude plugin uninstall blast-radius@team-mods` |
| token-weather | `claude plugin uninstall token-weather@team-mods` |
| replay-theater | `claude plugin uninstall replay-theater@team-mods` |
| mission-control | `claude plugin uninstall mission-control@team-mods` |
| context-bar | `claude plugin uninstall context-bar@team-mods` |
| change-ledger | `claude plugin uninstall change-ledger@team-mods` |
| turn-timeline | `claude plugin uninstall turn-timeline@team-mods` |
| Loop Breaker | `claude plugin uninstall loop-breaker@team-mods` |
| Assertion Guardian | `claude plugin uninstall assertion-guardian@team-mods` |
| token-ledger (카탈로그 밖) | `claude plugin uninstall token-ledger@claude-mods` |

원작자 카탈로그로 설치했다면 `@` 뒤의 이름이 다릅니다. → [7-6](#7-6-원작자-카탈로그로-설치한-mod-제거)

### 6-3. 카탈로그(마켓플레이스)까지 지우기

```bash
claude plugin marketplace list                        # 등록된 카탈로그 이름 확인
claude plugin marketplace remove team-mods
```

> **주의:** 카탈로그를 지우면 **그 카탈로그에서 설치한 mod가 모두 함께 제거됩니다.** `team-mods`를 지우면 팀 카탈로그로 설치한 mod 9개가 전부 사라집니다.
> mod 하나만 빼고 싶을 때는 6-2의 `uninstall`을 쓰세요. 이름은 GitHub 저장소 이름이 아니라 `marketplace list`에 나오는 이름입니다.

### 6-4. 폴더로 로드한 mod (폐쇄망 방식)

`--plugin-dir`나 `CLAUDE_CODE_PLUGIN_DIRS`로 로드한 mod는 설치된 게 아니라서 `uninstall` 대상이 아닙니다.

- `--plugin-dir`: 다음 실행부터 그 옵션을 빼면 됩니다.
- `CLAUDE_CODE_PLUGIN_DIRS`: `~/.claude/settings.json`의 `env`나 Docker 이미지 환경변수에서 해당 경로를 지우세요. 폴더 자체는 남아 있어도 로드되지 않습니다.

### 6-5. 내장 mod와 status line 되돌리기

- **내장 mod**(`diff`, `agents-md`)는 지울 수 없고 끄기만 됩니다. `/plugin`의 내장(built-in) 목록에서 끕니다. agents-md는 끄는 대신 [4-11](#4-11-내장-mod-설정-agents-md-끄기-권장) 설정을 권장합니다.
- **status line 래퍼**([4-12](#4-12-mod-없이-status-line에-context-표시))를 되돌리려면 `statusLine.command`를 원래 게이트웨이 스크립트 경로로 바꾸고 `~/.claude/statusline-wrap.sh`를 지우면 됩니다.

### 6-6. 한꺼번에 끄기 (문제가 생겼을 때)

| 범위 | 방법 |
| --- | --- |
| 설치한 플러그인 전부 | `claude plugin disable --all` |
| 이번 세션만 mod 없이 | `claude --safe-mode` |
| 항상 mod와 hook 전부 | `~/.claude/settings.json`에 `"disableAllHooks": true` (기존 settings hook도 같이 꺼짐, 내장 mod는 해당 없음) |

### 문제 해결

### 문제 해결

| 증상 | 확인할 것 |
| --- | --- |
| `/plugin`에 mod가 안 보임 | `claude --version` ≥ 2.1.287, 설치 후 재시작 또는 `/reload-plugins` |
| 패널이 안 열림 / "waiting" 메시지 | 터미널 폭(`tput cols`) 144 이상인지, VS Code 채팅 패널이 아닌 터미널인지 |
| `/mission code`에 지도가 안 나옴 | Linux에서는 정상. Who 뷰(`w`)를 사용 |
| change-ledger Edits에 파일이 빠짐 | 셸 명령으로 바뀐 파일은 Working tree에만 표시됨 |
| blast-radius 버튼이 안 보이고 10분 뒤 거부 | 다른 mod가 band를 쓰는 중. 터미널을 넓혀 사이드 패널로 열리게 하기 |
| `marketplace add`가 충돌 | 같은 이름(`claude-code-mods`) 마켓플레이스가 이미 있음(2-2 참고) |
| Claude 동작이 이상해짐 | `claude --safe-mode`로 mod 없이 재현되는지 확인 후 mod를 하나씩 켜서 원인 찾기 |

## 7. 참고: 원작자 카탈로그에서 직접 설치

팀 카탈로그(`team-mods`)를 만들기 전에 정리했던 원래 설치 방법입니다. 각 mod를 만든 원작자의 카탈로그에서 바로 설치합니다.
**팀에서는 2장의 팀 카탈로그를 권장합니다.** 이 방법은 원작자의 최신 버전을 써 보거나, 팀 카탈로그가 어디서 왔는지 확인할 때 참고하세요.

| Mod | 원작자 카탈로그 등록 | 카탈로그 이름 | 팀 카탈로그와 다른 점 |
| --- | --- | --- | --- |
| mission-control, context-bar | `claude plugin marketplace add hamzafer/claude-code-mods` | `claude-code-mods` | mission-control은 **Haiku 호출 패치가 적용되지 않음** |
| change-ledger, turn-timeline | `claude plugin marketplace add arasovic/claude-code-mods` | `claude-code-mods` | change-ledger는 **git timeout 패치가 적용되지 않음** |
| loop-breaker, assertion-guardian | `claude plugin marketplace add ccdwyer/claude-mods` | `ccdwyer-mods` | 폐쇄망에서 설치 안 됨(아래 참고) |
| blast-radius, token-weather, replay-theater | clone 후 `claude-code/mods` 폴더에서 `claude plugin marketplace add ./` | `claude-code-playground-mods` | 없음 |
| token-ledger | `claude plugin marketplace add Arunjay4213/claude-mods` | `claude-mods` | 팀 카탈로그에 없음 |

공통으로, 원작자 카탈로그는 **설치하는 시점의 최신 코드**를 받습니다. [SECURITY.md](./SECURITY.md)에서 점검한 커밋과 다를 수 있으니 설치 후 `claude plugin list`로 버전을 확인하세요.

### 7-1. mission-control, context-bar (hamzafer)

```bash
claude plugin marketplace add hamzafer/claude-code-mods
claude plugin install mission-control@claude-code-mods
claude plugin install context-bar@claude-code-mods
```

hamzafer 저장소에도 `blast-radius`가 있지만 Anthropic 것과 **다른 작성자의 다른 프로그램**이고 점검하지 않았습니다.

### 7-2. change-ledger, turn-timeline (arasovic)

```bash
claude plugin marketplace add arasovic/claude-code-mods
claude plugin install change-ledger@claude-code-mods
claude plugin install turn-timeline@claude-code-mods
```

> **카탈로그 이름 충돌:** hamzafer와 arasovic 카탈로그는 이름이 둘 다 `claude-code-mods`라 동시에 등록할 수 없습니다.
> `claude plugin marketplace list`로 확인하고, 둘 다 쓰려면 한쪽은 [3-3](#3-3-카탈로그-없이-폴더로-바로-로드)처럼 폴더로 로드하세요.
> 팀 카탈로그는 이 둘을 `team-mods` 하나로 합쳐서 이 문제가 없습니다.

### 7-3. Loop Breaker, Assertion Guardian (ccdwyer)

```bash
claude plugin marketplace add ccdwyer/claude-mods
claude plugin install loop-breaker@ccdwyer-mods
claude plugin install assertion-guardian@ccdwyer-mods
```

> ccdwyer 카탈로그는 각 mod의 출처가 GitHub(`"source": "github"`)로 적혀 있어, 받아온 폴더를 `marketplace add ./`로 등록해도 **폐쇄망에서는 설치가 실패합니다.**
> ccdwyer의 `install.sh`도 GitHub에 접속합니다. 폐쇄망에서는 팀 카탈로그를 쓰세요.

### 7-4. blast-radius, token-weather, replay-theater (Anthropic DevRel 예제)

카탈로그 파일이 저장소 루트가 아닌 하위 폴더에 있어서, clone한 뒤 경로로 등록합니다.

```bash
git clone https://github.com/anthropics/claude-code-playground.git
cd claude-code-playground/claude-code/mods
claude plugin marketplace add ./
claude plugin install blast-radius@claude-code-playground-mods --scope user
claude plugin install token-weather@claude-code-playground-mods --scope user
claude plugin install replay-theater@claude-code-playground-mods --scope user
```

### 7-5. token-ledger (Arunjay4213)

```bash
claude plugin marketplace add Arunjay4213/claude-mods
claude plugin install token-ledger@claude-mods
```

### 7-6. 원작자 카탈로그로 설치한 mod 제거

이름 뒤의 카탈로그만 다릅니다. 예: `claude plugin uninstall mission-control@claude-code-mods`, `claude plugin uninstall loop-breaker@ccdwyer-mods`.
카탈로그를 지우면(`claude plugin marketplace remove claude-code-mods`) 그 카탈로그로 설치한 mod가 모두 함께 제거됩니다.

## 참고

- [Claude Code mods 공식 문서](https://code.claude.com/docs/en/plugins/mods/overview)
- [mods API 레퍼런스](https://code.claude.com/docs/en/plugins/mods/reference)
- [관리자용 mod 설정](https://code.claude.com/docs/en/plugins/mods/admin)
- 각 mod 상세: [mission-control](https://claude-mods.com/mission-control/) · [change-ledger](https://claude-mods.com/change-ledger/) · [turn-timeline](https://claude-mods.com/turn-timeline/) · [blast-radius](https://claude-mods.com/blast-radius/) · [Loop Breaker](https://claude-mods.com/loop-breaker/) · [Assertion Guardian](https://claude-mods.com/assertion-guardian/) · [context-bar](https://claude-mods.com/context-bar/) · [token-ledger](https://claude-mods.com/token-ledger/)
