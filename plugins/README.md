# 팀 카탈로그(team-mods) 구성

이 폴더의 mod는 원작자 저장소에서 **점검한 커밋 그대로** 복사한 것입니다. 두 개만 팀에서 패치했습니다.
점검 내용은 [SECURITY.md](../SECURITY.md), 설치와 사용법은 [USAGE.md](../USAGE.md)를 보세요.

## 출처와 변경 내역

| Mod | 원본 저장소 | 원본 경로 | 커밋 | 라이선스 | 팀 변경 |
| --- | --- | --- | --- | --- | --- |
| blast-radius | [anthropics/claude-code-playground](https://github.com/anthropics/claude-code-playground) | `claude-code/mods/blast-radius` | `569c528` | Apache-2.0 | 없음 |
| token-weather | [anthropics/claude-code-playground](https://github.com/anthropics/claude-code-playground) | `claude-code/mods/token-weather` | `569c528` | Apache-2.0 | 없음 |
| replay-theater | [anthropics/claude-code-playground](https://github.com/anthropics/claude-code-playground) | `claude-code/mods/replay-theater` | `569c528` | Apache-2.0 | 없음 |
| context-bar | [hamzafer/claude-code-mods](https://github.com/hamzafer/claude-code-mods) | `mods/context-bar` | `11abba8` | MIT | 없음 |
| mission-control | [hamzafer/claude-code-mods](https://github.com/hamzafer/claude-code-mods) | `mods/mission-control` | `11abba8` | MIT | **패치** ([patches/mission-control.patch](../patches/mission-control.patch)), 버전 `0.1.2-team.1` |
| change-ledger | [arasovic/claude-code-mods](https://github.com/arasovic/claude-code-mods) | `change-ledger` | `9f1bef6` | MIT | **패치** ([patches/change-ledger.patch](../patches/change-ledger.patch)), 버전 `0.1.5-team.1` |
| turn-timeline | [arasovic/claude-code-mods](https://github.com/arasovic/claude-code-mods) | `turn-timeline` | `9f1bef6` | MIT | 없음 |
| loop-breaker | [ccdwyer/loop-breaker](https://github.com/ccdwyer/loop-breaker) | 저장소 루트 | `1eeaaba` | MIT | 없음 |
| assertion-guardian | [ccdwyer/assertion-guardian](https://github.com/ccdwyer/assertion-guardian) | 저장소 루트 | `ae1de89` | MIT | 없음 |

각 폴더의 `LICENSE`는 원본 저장소의 라이선스 파일입니다. 원본 폴더에 없을 때는 저장소 루트의 것을 복사했습니다.
`.git` 폴더만 빼고 나머지 파일은 원본과 같습니다.

## 패치 내용

**mission-control** (`hooks/register.tsx`)
- 파일을 수정한 턴마다 수정 코드 조각을 Haiku로 보내던 호출 한 줄을 지웠습니다.
- 이 요약은 macOS 전용 Code 뷰에만 쓰여서 Linux에서는 비용만 들었습니다. Who 뷰와 상단 표시는 그대로 동작합니다.

**change-ledger** (`hooks/register.tsx`)
- `git status`, `git diff`에 5초 timeout과 `--no-optional-locks`를 넣었습니다.
- `git status`에 `-uno`를 넣어 추적하지 않는 파일을 훑지 않습니다. 셸 명령으로 새로 만든 파일은 Working tree에 나오지 않습니다(Claude가 Write로 만든 파일은 Edits에 나옴).
- `/changes`에서 git이 실패해도 명령이 깨지지 않게 했습니다.

## 원본을 새 버전으로 올릴 때

1. 원본 저장소에서 새 커밋을 받아 [SECURITY.md](../SECURITY.md)와 같은 방식으로 다시 점검합니다.
2. 해당 폴더를 새 버전으로 교체하고, 패치 대상이면 `patches/`의 파일을 다시 적용합니다.

   ```bash
   patch -d plugins/mission-control -p1 < patches/mission-control.patch
   ```

3. `claude plugin validate plugins/<mod>`가 통과하는지 확인하고, 위 표의 커밋을 갱신합니다.
