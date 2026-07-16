# My Claude Code OS

Claude Code 위에 쌓아 올린 **작은 자동 개발 운영체제(OS)**다.
사람이 "무엇을" 원하는지 말하면, 여러 전문 **서브에이전트**가 릴레이로 **분석 → 개발 → 검증 → 문서화**를 수행하고, 그 흐름을 **스킬**이 지휘한다.

> 이 문서는 처음 보는 사람이 이 OS의 구조와 사용법을 한 번에 이해하도록 쓴 개요다.

---

## 1. 큰 그림 — 세 가지 부품

이 OS는 세 종류의 부품으로 이루어진다.

| 부품 | 무엇인가 | 비유 |
|------|----------|------|
| **서브에이전트 (agent)** | 한 가지 일만 잘하는 전문가. 각자 고유한 권한(도구)과 지침을 가짐 | 팀의 개별 구성원 |
| **스킬 (skill)** | 에이전트들을 **어떤 순서로 부를지** 지휘하는 워크플로우 | 팀을 이끄는 매니저(오케스트레이터) |
| **지침 (guideline)** | 테스트·구현·리뷰·문서·회고의 **판단 기준**. `.claude/guidelines/`에 있고, 필요한 에이전트·스킬에 **참조로 주입**됨 | 팀이 공유하는 사내 규정집 |

핵심 아이디어: **스킬은 "흐름"을, 에이전트는 "실행"을, 지침은 "기준"을 담는다.** 셋을 분리했기 때문에 하나의 에이전트를 여러 스킬이 재활용하고, 하나의 지침을 여러 에이전트가 공유한다.

> 💡 **왜 지침이 별도 부품인가?** "좋은 테스트란 무엇인가" 같은 기준을 에이전트마다 적어두면 시간이 지나며 서로 어긋난다(드리프트). 기준을 파일 하나로 떼어내 **단일 출처(SSOT)**로 두고 각 소비처는 포인터만 갖는다 → 자세한 구조는 3장.

---

## 2. 디렉터리 구조

```
my-claude-code-os/
├─ CLAUDE.md                     # 프로젝트 전역 지침(메인 세션에 항상 로드됨)
├─ OS.md                         # ← 이 문서
├─ README.md
├─ context-system.html           # 산출물: 컨텍스트 체계 1페이지 도식
├─ scripts/
│  ├─ skill_stats.py             # 산출물: 스킬 사용 로그 통계 유틸
│  ├─ injection_check.py         # 산출물: 지침 주입(배선) 검사 유틸
│  └─ daily_review.py            # 산출물: 일일 회고 집계 유틸
├─ tests/
│  ├─ test_skill_stats.py        # 산출물: unittest (24개)
│  ├─ test_injection_check.py    # 산출물: unittest (25개)
│  └─ test_daily_review.py       # 산출물: unittest (24개)
└─ .claude/
   ├─ settings.json              # 훅 등록 등 프로젝트 설정
   ├─ settings.local.json        # 로컬 전용 설정(권한 등)
   ├─ guidelines/                # ── 판단 기준의 단일 출처(SSOT) ── (→ 3장)
   │  ├─ testing.md              #   주입: 테스트 작성 기준
   │  ├─ coding-style.md         #   주입: 구현 코드 기준
   │  ├─ review-criteria.md      #   주입: 리뷰 판정 기준(🔴🟡🟢)
   │  ├─ doc-style.md            #   주입: 문서 작성 기준
   │  ├─ retro-guideline.md      #   주입: 회고 작성 기준
   │  └─ subagent-specialization.md  # 주입 안 됨 — 설계 청사진(읽는 문서)
   ├─ agents/                    # ── 서브에이전트 정의 (each *.md) ──
   │  ├─ code-analyzer.md        #   ① 코드베이스 파악 (읽기 전용)
   │  ├─ test-writer.md          #   ②a 요구 분석 + red 테스트 작성 (쓰기)
   │  ├─ impl-writer.md          #   ②b red 테스트를 green으로 구현 (쓰기)
   │  ├─ review-correctness.md   #   ③ 리뷰 렌즈: 정확성·안정성 (읽기 전용, 병렬)
   │  ├─ review-tests.md         #   ③ 리뷰 렌즈: 테스트 (읽기 전용, 병렬)
   │  ├─ code-reviewer.md        #   ③ 단일 리뷰·판정 (quick-review 전용, 읽기 전용)
   │  ├─ doc-writer.md           #   ④ 문서화 (문서만 쓰기)
   │  ├─ retro-writer.md         #   회고 작성 (저널만 쓰기)
   │  └─ ralph-worker.md         #   랄프 루프 1회차 실행 (독립 컨텍스트)
   ├─ skills/                    # ── 스킬 정의 (each <이름>/SKILL.md) ──
   │  ├─ feature-dev/            #   기능 개발 풀 파이프라인
   │  ├─ quick-review/           #   가벼운 코드 리뷰
   │  ├─ git-commit/             #   커밋·푸시
   │  ├─ skill-stat/             #   스킬 사용 통계(bash)
   │  ├─ daily-review/           #   일일 개발 회고
   │  └─ ralph-loop/             #   목표까지 반복하는 랄프 루프
   ├─ hooks/
   │  └─ skill-counter.sh        # 스킬 실행 때마다 로그를 남기는 훅
   ├─ journal/                   # 회고 저널 (YYYY-MM-DD.md) — retro-writer가 씀
   └─ logs/
      └─ skill-usage.log         # "시각<TAB>스킬이름" 한 줄씩 누적
```

**규칙**: 에이전트는 `.claude/agents/<이름>.md`, 스킬은 `.claude/skills/<이름>/SKILL.md`. 파일을 두면 Claude Code가 자동으로 인식한다(재시작·빌드 불필요).

---

## 3. 지침 체계 — 판단 기준의 단일 출처(SSOT)

`.claude/guidelines/`에는 6개 문서가 있고, 그중 **5개가 실제로 주입되는 지침**이다. 나머지 1개는 읽는 문서다.

| 지침 | 무엇의 기준인가 | 주입 여부 |
|------|-----------------|:---------:|
| **testing.md** | 테스트 작성 | ✓ |
| **coding-style.md** | 구현 코드 | ✓ |
| **review-criteria.md** | 리뷰 판정(🔴 수정필요 / 🟡 / 🟢) | ✓ |
| **doc-style.md** | 문서 작성 | ✓ |
| **retro-guideline.md** | 회고 작성 | ✓ |
| **subagent-specialization.md** | 에이전트 세분화 **설계 청사진** | ✗ (규칙으로 주입되지 않는, 읽는 문서) |

### 3-1. 3층 주입 구조 — 누가 언제 읽나

같은 지침이라도 **읽는 주체와 로드 시점**이 다르다.

| 층 | 경로 | 누가 읽나 | 로드 시점 |
|:--:|------|-----------|-----------|
| **1층** | `CLAUDE.md` | 메인 세션만 | 항상 |
| **2층** | 에이전트 `.md` 본문 | 해당 서브에이전트 | 스폰될 때만 |
| **3층** | `SKILL.md` | 오케스트레이터 | 그 스킬 호출 시 |

> 💡 **왜 2층이 필요한가?** **서브에이전트는 CLAUDE.md를 상속받지 않는다.** 메인 세션이 아는 것을 격리된 서브에이전트는 모른다. 그래서 에이전트 `.md` 본문이 **격리된 서브에이전트에 지침을 넣는 유일한 경로**다. 여기 포인터를 안 적으면 그 에이전트는 규칙 없이 일한다.
>
> 💡 **왜 정의를 복사하지 않고 포인터만 두나?** 각 소비처에는 `"`.claude/guidelines/testing.md`를 따른다"` 한 줄만 둔다. 정의를 복사해두면 원본이 바뀔 때 사본들이 뒤처져 서로 어긋난다(드리프트). 단일 출처면 **한 곳만 고치면 전부 반영**된다. 덤으로 지침 본문은 필요할 때만 로드돼 메인 컨텍스트가 가볍다(이 최적화로 2,452자 → 1,293자, **−47%**).

### 3-2. 배선이 끊기면 잡아낸다

포인터 방식의 유일한 약점은 **참조가 조용히 끊길 수 있다**는 것이다(지침 파일 이름 변경·삭제, 소비처에서 참조 실수로 제거). 그래서 `scripts/injection_check.py`가 배선을 검사한다.

```bash
python3 scripts/injection_check.py
# ✅ 컨텍스트 주입 배선 정상 — 지침 5개가 모두 올바르게 연결됨.
```

- 의도된 배선 맵은 `injection_check.py`의 `REQUIRED_INJECTIONS`가 단일 출처다.
- 깨진 참조(dangling)·주입 누락·없는 파일을 잡고, `tests/test_injection_check.py`(25개)가 이를 테스트로 고정한다.

> 💡 **왜 배선까지 테스트하나?** 주입은 "조용히" 깨진다 — 참조 한 줄이 사라져도 에러가 나지 않고, 그 에이전트가 규칙 없이 일할 뿐이라 결과물이 미묘하게 나빠진다. 실제로 **일부러 배선을 끊어** 이 검사가 빨갛게 되는 것을 확인했다.

---

## 4. 서브에이전트 (9개)

각 에이전트는 `.md` 파일 하나이고, 상단 frontmatter에 `name`·`description`·`tools`(권한)를 둔다.

| 에이전트 | 단계 | 역할 | 권한(tools) | 코드 수정 | 주입 지침 |
|----------|:----:|------|-------------|:--------:|-----------|
| **code-analyzer** | ① | 관련 파일·관습·진입점을 지도로 정리 | Read, Grep, Glob, Bash | ✗ (읽기 전용) | — |
| **test-writer** | ②a | 요구 분석 → 수용 기준 → **실패하는(red) 테스트** 작성 | Read, Grep, Glob, **Write, Edit**, Bash | 테스트만 | testing · coding-style |
| **impl-writer** | ②b | red 테스트를 **green**으로 만드는 최소 구현 (테스트는 못 고침) | Read, Grep, Glob, **Write, Edit**, Bash | 구현만 | coding-style |
| **review-correctness** | ③ | 리뷰 렌즈 **정확성·안정성** → 판정 (feature-dev, 병렬) | Read, Grep, Glob, Bash | ✗ (지적만) | review-criteria |
| **review-tests** | ③ | 리뷰 렌즈 **테스트 커버리지·green** → 판정 (feature-dev, 병렬) | Read, Grep, Glob, Bash | ✗ (지적만) | review-criteria · testing |
| **code-reviewer** | ③ | 단일 리뷰 후 **통과/수정필요** 판정 (quick-review 전용) | Read, Grep, Glob, Bash | ✗ (지적만) | review-criteria |
| **doc-writer** | ④ | 변경 내용을 문서로 기록 | Read, Grep, Glob, **Write, Edit** | 문서만 | doc-style |
| **retro-writer** | — | 하루 회고 작성 전담. `daily-review`가 집계 데이터를 넘겨 호출 | Read, Grep, Glob, **Write, Edit**, Bash | ✗ (회고 저널만) | retro-guideline |
| **ralph-worker** | — | 랄프 루프의 **한 이터레이션**을 독립 컨텍스트에서 실행. 종료·지표 판정은 안 함 | Read, Grep, Glob, **Write, Edit**, Bash | 목표에 필요한 만큼 | coding-style · testing |

> 💡 **최소 권한 원칙**: 필요한 도구만 준다. 읽기 전용(analyzer·review-*·code-reviewer)은 Write/Edit이 없고, doc-writer는 코드 실행이 필요 없어 Bash도 없다.
>
> 💡 **②단계 TDD 분리**: `test-writer`가 "무엇이 맞는가"(red 테스트=명세)를, `impl-writer`가 "어떻게 충족하는가"(green 구현)를 소유한다. impl-writer는 **남이 쓴 테스트를 통과시켜야** 하므로 자기 테스트를 무력화할 수 없다. 단, 도구 권한으로 경로를 막지는 못하므로 스킬이 `git diff`로 테스트 미변경을 사후 검증한다.
>
> 💡 **③단계 리뷰의 두 갈래**: 무거운 `feature-dev`는 관점을 나눈 `review-correctness`·`review-tests`를 **병렬로** 돌려 깊게 검증하고, 가벼운 `quick-review`는 단일 `code-reviewer`로 빠르게 본다. 자세한 설계 근거는 [subagent-specialization.md](.claude/guidelines/subagent-specialization.md) 참고.
>
> 💡 **"만드는 자 ↔ 재는 자" 분리는 파이프라인 밖에서도 같다**: `retro-writer`가 하루를 회고하는 이유도(자기 하루를 자평하면 편향), `ralph-worker`가 종료 판정을 못 하는 이유도(일한 주체가 자기 성과를 재면 편향) 리뷰어와 작성자를 나눈 것과 같은 원칙이다.

---

## 5. 스킬 (6개)

스킬은 두 종류로 나뉜다 — 에이전트를 지휘하는 **오케스트레이터**와, 직접 일을 처리하는 **실행형**.

| 스킬 | 유형 | 트리거(예) | 사용하는 에이전트 |
|------|------|-----------|-------------------|
| **feature-dev** | 오케스트레이터 | "~기능 만들어줘/구현해줘" | analyzer → test-writer → impl-writer → [review-correctness ∥ review-tests] ⇄ (라우팅) → doc-writer |
| **quick-review** | 오케스트레이터 | "리뷰해줘/봐줘" | analyzer → code-reviewer |
| **daily-review** | 오케스트레이터 | "오늘 회고", "일일 회고해줘", "retro" | `daily_review.py` 집계 → retro-writer → `.claude/journal/YYYY-MM-DD.md` 저장 |
| **ralph-loop** | 오케스트레이터 | "랄프 루프 돌려줘", "○○ 될 때까지 반복해줘" | 지표 측정 → ralph-worker 스폰(회차마다 독립 컨텍스트) → 재측정 → 종료 판정 → 체크포인트 커밋 |
| **git-commit** | 실행형 | "커밋해줘/푸시해줘" | (없음 — 직접 git 수행) |
| **skill-stat** | 실행형 | "스킬 통계 보여줘" | (없음 — 로그 집계) |

---

## 6. 오케스트레이션 & 파이프라인

### 6-1. feature-dev — 기능 개발 풀 파이프라인 (4단계 + 검증 루프)

```
   사용자: "○○ 기능 만들어줘"
        │
        ▼                                                   ┌─ review-correctness ─┐ (정확성·안정성)
 ① analyzer ─▶ ②a test-writer ─▶ ②b impl-writer ─▶ ③ ─────┤  (병렬 리뷰)          ├─┐
   (어디를)      (red 테스트=스펙)   (green 구현)             └─ review-tests ───────┘ │ (테스트)
                     ▲                  ▲                                             │
   테스트 🔴 ────────┘   정확성 🔴 ──────┘         판정=수정필요(🔴 하나라도)            │
   (test-writer 보강      (impl-writer 수정)  ◀───────────────────────────────────────┘
    →impl이 green)                                (최대 3회 반복)
                                          │ 둘 다 판정=통과
                                          ▼
                                    ④ doc-writer ──▶ 최종 보고
                                      (문서화)
```

- **② 개발이 TDD 분리**다. `test-writer`가 red 테스트(=명세)를 쓰고, `impl-writer`가 그걸 green으로 만든다. impl-writer는 **테스트를 못 고친다**(남이 쓴 명세를 통과시켜야 함) — 스킬이 `git diff`로 사후 검증한다.
- **③ 검증 루프**가 심장이다. 두 리뷰어를 **병렬로** 돌리고, 지적을 **작성자별로 라우팅**한다 — 정확성 🔴 → impl-writer, 테스트 🔴 → test-writer(보강)→impl-writer(green). **둘 다 "통과"거나 3회 도달 시** 종료한다.
- 리뷰어는 **직접 고치지 않는다.** 수정은 항상 test-writer/impl-writer가 한다.

### 6-2. quick-review — 가벼운 리뷰 (2단계, 읽기 전용)

```
   사용자: "이 코드 리뷰해줘"
        │
        ▼
 ① code-analyzer ──▶ ② code-reviewer ──▶ 판정·지적 보고
   (맥락 파악)         (통과/수정필요)
```

- 새로 만들지 않고 **읽고 판정만** 한다. 코드를 수정하지 않아 빠르고 안전하다.
- 가벼운 흐름이라 **단일 `code-reviewer`**를 쓴다(feature-dev의 관점 병렬 팬아웃은 이 스킬엔 과하다).

### 6-3. 공유 에이전트 (재활용)

```
                    feature-dev  quick-review  daily-review  ralph-loop
 code-analyzer           ✓            ✓             ✗            ✗     ◀── 공유
 test-writer             ✓            ✗             ✗            ✗     ◀── TDD 분리(feature-dev 전용)
 impl-writer             ✓            ✗             ✗            ✗     ◀── TDD 분리(feature-dev 전용)
 review-correctness      ✓            ✗             ✗            ✗     ◀── 관점 병렬(feature-dev 전용)
 review-tests            ✓            ✗             ✗            ✗     ◀── 관점 병렬(feature-dev 전용)
 code-reviewer           ✗            ✓             ✗            ✗     ◀── 단일 리뷰(quick-review 전용)
 doc-writer              ✓            ✗             ✗            ✗
 retro-writer            ✗            ✗             ✓            ✗     ◀── 회고 전담(daily-review 전용)
 ralph-worker            ✗            ✗             ✗            ✓     ◀── 1회차 실행(ralph-loop 전용)
```

`code-analyzer`는 **두 스킬이 함께 재활용**한다("코드를 파악하는 전문가"를 한 번 정의해 여러 워크플로우에서 돌려 씀). 리뷰는 무게에 따라 갈라진다 — 무거운 개발은 관점을 나눈 `review-*` 병렬, 가벼운 리뷰는 단일 `code-reviewer`. 같은 "판정" 역할이라도 상황에 맞는 깊이를 고르는 것이 세분화의 이점이다. 한편 `ralph-loop`는 워커가 "무거운 기능 개발이 필요하다"고 보고하면 그 회차를 **`feature-dev`에 통째로 위임**한다 — 스킬이 스킬을 재활용하는 형태다.

### 6-4. daily-review — 일일 개발 회고 (3단계)

```
   사용자: "오늘 회고" / "retro"
        │
        ▼
 ① daily_review.py ──▶ ② retro-writer ──▶ ③ .claude/journal/YYYY-MM-DD.md
   (집계: 커밋·타입           (회고 작성)        (저장 + 사용자 보고)
    분포·이월 액션아이템)
```

- **결정적 데이터**(커밋 수·타입 분포·이월 액션아이템)는 테스트된 스크립트가 집계하고, **질적 판단**(좋았던/아쉬웠던/보완사항)은 서브에이전트가 쓴다.
- `retro-writer`는 `retro-guideline.md`의 다섯 섹션(좋았던/아쉬웠던/보완사항/커밋 요약/지표)을 채우고, 이전 회고의 미해결 `- [ ]`를 맨 위에 **이월**한다.
- 이 스킬은 **회고 저널만 쓴다** — 코드·테스트·문서를 건드리지 않는다.

> 💡 **왜 데이터와 회고를 나누나?** "오늘 커밋 몇 개"는 결정적이라 스크립트로 집계해 **테스트로 고정**할 수 있고, "무엇이 아쉬웠나"는 판단이라 에이전트가 써야 한다. 이 OS가 곳곳에서 쓰는 "결정적 vs 질적" 분리 패턴이다.

### 6-5. ralph-loop — 목표까지 반복하는 랄프 루프

```
   사용자: "○○ 될 때까지 반복해줘" (+ 목표 G, 지표 M)
        │
        ▼
   ┌─▶ ① M 측정(baseline) ─▶ ② ralph-worker 스폰 ─▶ ③ M 재측정 ─▶ ④ 종료 판정
   │      (오케스트레이터)      (독립 컨텍스트, 새 워커)   (오케스트레이터)      │
   │                              "다음 한 가지"만 실행                        │
   │                              PROGRESS.md에 기록                          │
   └──────────────── ⑤ 체크포인트 커밋 ◀── 계속 ───────────────────────────────┘
                                          │ 종료
                                          ▼
                     목표 달성 / 개선 없음(연속 2회) / 최대 회차 → 최종 보고
```

- **종료 조건이 반드시 있다**: 목표 달성, 개선 없음(dry) 연속 2회, 최대 회차(기본 8), 지표 측정 실패·위험 작업.
- 되돌릴 수 있는 브랜치에서 돌고, **회차마다 체크포인트 커밋**을 남긴다(잘못되면 그 회차만 롤백).
- 지표는 **프로그램으로 측정 가능**해야 한다. 못 재는 목표면 이 스킬을 쓰지 않는다.

> 💡 **왜 회차마다 새 워커인가?** 서브에이전트는 매 호출이 격리된 새 컨텍스트다. 그래서 ① 회차가 쌓여도 워커 토큰이 안 불고, ② 앞 회차의 혼동·환각이 누적되지 않으며, ③ 기억을 대화가 아니라 **파일(`PROGRESS.md`)**에 남기도록 강제된다. 남기지 않은 것은 사라지므로 워커는 기록할 수밖에 없다 — 이게 랄프의 본질이다.
>
> 💡 **왜 측정은 오케스트레이터가 하나?** 일한 주체가 자기 성과를 자평하면 편향된다. **워커는 실행, 오케스트레이터는 지표로 판정**. 같은 이유로 오케스트레이터는 워커가 지표를 속이려 테스트를 삭제·무력화하지 않았는지도 함께 본다.

---

## 7. 실행 방법 — 어떻게 명령하나

명령하는 방법은 3가지이고, 대부분은 **①번(자연어)**이면 충분하다.

| 방식 | 방법 | 예시 |
|------|------|------|
| ① 자연어(기본) | 하고 싶은 걸 그냥 말함 → 알맞은 스킬이 자동 선택 | `"로그 파싱 기능 만들어줘"` |
| ② 슬래시(명시) | `/스킬이름`으로 특정 스킬 콕 집기 | `/feature-dev`, `/quick-review` |
| ③ 에이전트 지정 | 특정 에이전트 하나만 직접 호출 | `"code-analyzer로 구조 파악해줘"` |

**대표 4가지만 기억하면 된다:**
- 만들 때 → **feature-dev** (`"~만들어줘"`)
- 볼 때 → **quick-review** (`"리뷰해줘"`)
- 올릴 때 → **git-commit** (`"커밋해줘"`)
- 하루를 닫을 때 → **daily-review** (`"오늘 회고"`)

### 산출물 유틸 직접 실행
```bash
python3 -m unittest discover tests      # 테스트 73개 실행
python3 scripts/injection_check.py      # 지침 주입 배선 검사
python3 scripts/skill_stats.py          # 스킬 사용 통계
python3 scripts/skill_stats.py --top 2  # 상위 2개만
python3 scripts/daily_review.py         # 오늘 회고용 집계 데이터
```

---

## 8. 훅 & 로깅 — OS가 자기 사용을 기록한다

```
 스킬 실행(Skill 툴 호출)
        │  settings.json 의 PreToolUse 훅(matcher: "Skill")
        ▼
 skill-counter.sh  ──기록──▶  .claude/logs/skill-usage.log   ("시각<TAB>스킬이름")
                                          │
                          ┌───────────────┴───────────────┐
                          ▼                                ▼
                  skill-stat (스킬)                 scripts/skill_stats.py (유틸)
                  로그를 bash로 집계                로그를 파이썬으로 집계 + --top N
```

- 어떤 스킬이든 실행되면 훅이 자동으로 로그 한 줄을 남긴다.
- 그 로그를 `skill-stat` 스킬(bash)이나 `skill_stats.py`(파이썬)로 통계 낼 수 있다 — **OS가 자신의 사용 패턴을 관찰하는 피드백 루프**다.

---

## 9. 산출물 (이 OS로 만든 결과물)

| 파일 | 무엇 | 만든 방법 |
|------|------|-----------|
| `scripts/skill_stats.py` | 스킬 사용 로그 통계 유틸(`--top N` CLI 옵션 + 순수 함수 `count_skills`·`count_by_weekday`·`top_skills`) | **feature-dev 파이프라인**으로 개발 |
| `tests/test_skill_stats.py` | 위 유틸의 unittest 24개 (전부 green) | 같은 파이프라인의 개발 단계(현재는 test-writer→impl-writer)가 작성 |
| `scripts/injection_check.py` + `tests/test_injection_check.py` | 지침 주입(배선)이 깨지면 빨갛게 되는 검증 안전망(순수 함수 3 + 통합 테스트 5 + CLI, 테스트 25개). `python3 scripts/injection_check.py`로 배선 상태 확인 | 직접 작성 (testing.md·coding-style.md 준수) |
| `scripts/daily_review.py` + `tests/test_daily_review.py` | 일일 회고 집계 유틸(테스트 24개). 순수 함수 3개 — `commits_for_date`(git 로그 줄 → 특정 날짜 커밋), `summarize_commit_types`(커밋 제목 → conventional 타입별 집계), `extract_open_action_items`(회고 마크다운 → 미완료 `- [ ]` 추출) — 를 I/O 껍데기(`collect`/`main`)와 분리 | **feature-dev 파이프라인**으로 개발 |
| [`context-system.html`](context-system.html) | 컨텍스트 체계(SSOT→3중 주입→파이프라인 소비)와 주입 A/B를 담은 1페이지 도식 | 직접 작성 (아티팩트) |

이 산출물 자체가 "OS 전체 사이클이 실제로 한 바퀴 돈다"는 증거다. feature-dev를 여러 번(초기 구현 → `--top N` → 요일별 집계 → 일일 회고) 구동했고, 매번 4단계 + 검증 루프를 완주했다. 전체 테스트는 **73개**(skill_stats 24 + injection_check 25 + daily_review 24) 전부 green이다.

> 💡 **검증 루프가 실제로 버그를 잡았다.** `daily_review.py` 개발 중 `review-correctness`가 🔴을 검출했다 — `extract_open_action_items`가 **내용 없는 빈 체크박스 `- [ ]`**(회고 템플릿의 플레이스홀더)까지 액션아이템으로 집계해 빈 문자열이 다운스트림으로 새어나가는 버그. 지적은 자동으로 라우팅돼 test-writer(red 테스트 보강) → impl-writer(가드 추가) → 재리뷰 통과로 흘렀다. **작성자와 리뷰어를 나눈 설계가 실제로 값을 했다**는 증거다.
>
> 💡 **컨텍스트 지침 체계**는 [3장](#3-지침-체계--판단-기준의-단일-출처ssot)에 정리돼 있고, [`context-system.html`](context-system.html)에 시각화돼 있다. 각 에이전트의 역할·권한 요약은 **4장 표**가, 지침 자체의 정의는 `.claude/guidelines/`의 각 파일이 단일 출처(SSOT)다.

### 9-1. `count_by_weekday` — 요일별 호출 집계 (순수 함수)

로그의 시각에서 **요일을 파생**해 요일별 호출 횟수를 세는 순수 함수다. 반환은 `{요일한글: 횟수}` dict이며, 키 순서는 등장/횟수 순이 아니라 **월→화→수→목→금→토→일 고정 순서**(등장한 요일만 포함)다. 무시 규칙은 `count_skills`와 같고(빈 줄·탭 없는 줄·스킬이름 빈 줄), 여기에 **날짜 파싱 불가 줄**도 추가로 건너뛴다.

```python
from skill_stats import count_by_weekday

lines = [
    "2026-06-25 20:34:11\tgit-commit",   # 목
    "2026-06-25 09:00:00\tfeature-dev",  # 목
    "2026-07-02 08:43:54\tquick-review", # 목
]
count_by_weekday(lines)   # {"목": 3}
```

> 💡 **왜 CLI에 노출하지 않았나?** 이번엔 순수 함수와 모듈 상수(`WEEKDAYS_KO`)만 추가하고 `--by-weekday` 같은 CLI 옵션은 넣지 않았다(YAGNI). 실제 요구가 생기기 전에 인터페이스부터 늘리면 유지할 표면적만 커진다. 지금은 다른 코드가 `import`해 쓰거나 테스트로 검증하는 형태로만 존재한다.
>
> 💡 **왜 키 순서를 고정하나?** 요약 리포트에서 요일 축은 항상 같은 순서로 읽혀야 사람이 비교하기 쉽다. 그래서 등장 순서(dict 삽입 순)에 맡기지 않고 `WEEKDAYS_KO` 기준으로 재조립한다.

---

## 10. 설계 원칙 (왜 이렇게 만들었나)

1. **역할 분리** — 만드는 사람(writer)과 검증하는 사람(reviewer)을 나눈다. 자기 코드를 자기가 리뷰하면 생기는 확증 편향을 막는다. 같은 원칙이 회고(retro-writer)와 랄프 루프(워커는 실행, 오케스트레이터는 판정)에도 적용된다.
2. **최소 권한** — 각 에이전트에 딱 필요한 도구만 준다. 읽기 전용은 읽기만, 문서 담당은 문서만.
3. **단일 출처(SSOT)** — 판단 기준은 `.claude/guidelines/`에 한 번만 정의하고, 소비처는 **포인터**만 갖는다. 정의를 복사하면 시간이 지나며 어긋난다(드리프트). 배선은 `injection_check.py`가 지킨다.
4. **종료 조건 있는 루프** — 검증 루프는 "판정=통과" 또는 "3회"에서, 랄프 루프는 "목표 달성/개선 없음/최대 회차"에서 반드시 멈춘다. 자동화가 무한 반복에 빠지지 않게.
5. **명시적 인계** — 서브에이전트끼리는 대화 맥락을 공유하지 않으므로, 스킬이 앞 단계 결과를 다음 단계 입력으로 직접 넘긴다.
6. **결정적 vs 질적 분리** — 셀 수 있는 것은 테스트된 스크립트가 집계하고, 판단이 필요한 것은 에이전트가 쓴다.
7. **자동 발견** — 파일을 규칙대로 두기만 하면 인식된다. 별도 등록/빌드가 없다.

---

## 11. 확장하는 법

- **새 에이전트 추가** → `.claude/agents/<이름>.md` 생성. frontmatter에 `name`·`description`·`tools`를 적고, 본문에 역할·절차·출력형식·원칙을 쓴다. 어떤 스킬이 부를지도 함께 정한다(에이전트는 스킬이 불러야 동작). **따라야 할 지침이 있으면 본문에 포인터 한 줄을 반드시 넣는다** — 서브에이전트는 CLAUDE.md를 상속받지 않는다(→ 3장).
- **새 스킬 추가** → `.claude/skills/<이름>/SKILL.md` 생성. `description`에 트리거 문구("~할 때 사용한다")를 자연어로 적을수록 자동 선택이 잘 된다. 훅 카운팅은 자동 포함된다.
- **새 지침 추가** →
  1. `.claude/guidelines/<이름>.md`에 기준을 **한 번만** 정의한다(이 파일이 SSOT).
  2. 그 기준을 지켜야 할 소비처(에이전트 `.md` / `SKILL.md`)에 **포인터 한 줄**(`` `.claude/guidelines/<이름>.md`를 따른다``)을 넣는다. 정의를 복사하지 않는다.
  3. `scripts/injection_check.py`의 `INJECTED_GUIDELINES`와 `REQUIRED_INJECTIONS`에 등록한다(= 의도된 배선 선언).
  4. 이후로는 **테스트가 배선을 지켜준다** — 참조가 끊기면 `tests/test_injection_check.py`가 빨갛게 된다.
- **팁** — `description`은 곧 "리모컨 버튼의 라벨"이다. 자동 선택이 잘 안 되면 트리거 문구를 보강하라.
