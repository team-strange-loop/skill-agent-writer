---
name: skill-agent-writer
description: This skill should be used when the user asks to "write a skill", "create a SKILL.md", "review a skill", "refactor a skill", "write an agent", "create an agent", "review an agent", "에이전트 작성", "에이전트 만들기", "에이전트 리뷰", "스킬 작성", "스킬 만들기", "SKILL.md 만들기", "스킬 리뷰", "스킬 리팩토링", "skill best practices", "agent best practices", "스킬 베스트 프랙티스", or wants to create, review, or improve SKILL.md or agent .md files following Anthropic best practices.
version: 2.0.0
user-invocable: true
---

# 스킬 & 에이전트 작성기

Claude Code용 고품질 SKILL.md와 에이전트 `.md` 파일을 작성, 리뷰, 리팩토링하는 스킬. 모든 가이드라인은 `references/` 디렉토리에 번들된 공식 Anthropic 스킬 가이드와 에이전트 설계 패턴에 기반한다.

## 모드

이 스킬은 다섯 가지 모드로 동작한다:

1. **스킬 작성** — 대화형 위저드로 요구사항을 수집한 후 완성된 SKILL.md + SKILL-ko.md 쌍을 생성한다.
2. **스킬 리뷰** — 기존 SKILL.md를 10점 품질 체크리스트에 따라 분석하고 점수 보고서를 생성한다.
3. **스킬 리팩토링** — 리뷰 결과를 반영하여 기존 SKILL.md를 원래 의도를 유지하면서 개선한다.
4. **에이전트 작성** — 대화형 위저드로 요구사항을 수집한 후 프로젝트 패턴에 따른 에이전트 `.md` 파일을 생성한다.
5. **에이전트 리뷰** — 기존 에이전트 `.md` 파일을 8점 에이전트 품질 체크리스트에 따라 분석하고 점수 보고서를 생성한다.

사용자의 의도에서 모드를 판단한다. 모호한 경우 질문한다.

## 스킬 품질 체크리스트 (10항목)

모드 1-3에서 Anthropic 스킬 가이드에서 도출된 이 체크리스트를 사용한다:

| # | 기준 | 최대 점수 |
|---|------|-----------|
| 1 | **명확한 목적** — 스킬이 무엇을 하는지, 언제 사용하는지 설명하는 간결한 설명 | 2 |
| 2 | **트리거 커버리지** — 설명에 영어와 한국어 트리거 키워드 모두 포함 | 2 |
| 3 | **프론트매터** — name, description, version, user-invocable이 포함된 유효한 YAML | 2 |
| 4 | **구조화된 워크플로우** — 번호가 매겨진 단계별 지침 | 2 |
| 5 | **도구 안내** — 사용할 도구를 명시 (Read, Write, Edit, Bash 등) | 2 |
| 6 | **출력 형식** — 예제나 템플릿으로 예상 출력을 정의 | 2 |
| 7 | **에지 케이스** — "사용 시점" / "건너뛸 시점" 가드 조건 포함 | 2 |
| 8 | **실행 가능한 지침** — 명령형 동사와 구체적 행동 사용 | 2 |
| 9 | **참조 자료** — 참조 파일 링크 또는 복잡한 주제에 대한 인라인 예제 제공 | 2 |
| 10 | **완결성** — 자기 완결적이어서 LLM이 외부 지식 없이 전체 워크플로우를 실행 가능 | 2 |

**등급**: A (18-20), B (14-17), C (10-13), D (6-9), F (0-5)

## 에이전트 품질 체크리스트 (8항목)

모드 4-5에서 `docs/05-agents-deep-dive.md`와 Anthropic 에이전트 하네스 가이드에서 도출된 이 체크리스트를 사용한다:

| # | 기준 | 최대 점수 |
|---|------|-----------|
| 1 | **명확한 목적** — 에이전트가 무엇을 하는지 설명하는 한 단락 | 2 |
| 2 | **분석 프로세스** — 에이전트의 워크플로우를 설명하는 번호가 매겨진 단계 | 2 |
| 3 | **출력 형식** — 결과에 대한 명시적 마크다운 구조 | 2 |
| 4 | **분류/범주화** — 구조화된 결정 로직 | 2 |
| 5 | **품질 기준** — 좋은 출력에 대한 기준 | 2 |
| 6 | **가드 조건** — 건너뛸 조건 또는 범위 제한 | 2 |
| 7 | **도구 힌트** — 필요한 도구 목록 (해당되는 경우) | 2 |
| 8 | **완결성** — 자기 완결적이며, 외부 컨텍스트 없이 LLM이 실행 가능 | 2 |

**등급**: A (14-16), B (11-13), C (8-10), D (5-7), F (0-4)

---

## 스킬 작성 워크플로우

### 1단계: 요구사항 수집

AskUserQuestion 도구를 사용하여 수집한다:

1. **스킬 이름** — kebab-case 식별자 (예: `code-reviewer`)
2. **목적** — 이 스킬은 무엇을 하는가? (1-2문장)
3. **트리거 문구** — 활성화할 자연어 문구 (영어 + 한국어)
4. **모드** — 몇 가지 동작 모드가 있는가? 각각 무엇을 하는가?
5. **필요한 도구** — 어떤 Claude Code 도구가 필요한가? (Read, Write, Edit, Bash, Glob, Grep, Task, WebSearch, WebFetch, AskUserQuestion)
6. **출력 형식** — 최종 출력은 어떤 모습이어야 하는가?
7. **참조 파일** — 스킬에 참조 문서가 필요한가?

사용자가 부분적인 정보를 제공하면 합리적인 기본값을 채우고 확인한다.

### 2단계: 참조 자료 읽기

생성 전에 관련 참조 파일을 읽어 모범 사례를 확인한다:

- `references/anthropic-skills-guide/03-skill-structure.md`에서 구조 가이드 읽기
- `references/anthropic-skills-guide/04-writing-instructions.md`에서 지침 품질 읽기
- `references/anthropic-skills-guide/07-patterns.md`에서 일반적인 패턴 읽기

Read 도구를 전체 경로와 함께 사용한다: `${pluginDir}/skills/skill-agent-writer/references/anthropic-skills-guide/{filename}`

### 3단계: 에이전트를 통한 생성

Task 도구를 사용하여 `skill-generator` 에이전트를 subagent_type `skill-agent-writer:skill-generator`로 실행한다. 수집한 요구사항을 프롬프트로 전달한다.

에이전트가 SKILL.md와 SKILL-ko.md 콘텐츠를 모두 생성한다.

### 4단계: 파일 작성

Write 도구를 사용하여 두 파일을 생성한다:

1. 사용자가 지정한 경로에 `SKILL.md` 작성 (기본값: `.claude/skills/{name}/SKILL.md`)
2. 같은 디렉토리에 `SKILL-ko.md` 작성

### 5단계: 검증

10점 스킬 체크리스트에 대해 빠른 리뷰를 실행한다. 0점인 기준이 있으면 플래그하고 수정을 제안한다.

---

## 스킬 리뷰 워크플로우

### 1단계: 대상 식별

사용자에게 어떤 SKILL.md를 리뷰할지 묻는다. 지정하지 않으면 Glob으로 SKILL.md 파일을 찾는다:

```
**/SKILL.md
.claude/skills/*/SKILL.md
plugins/*/skills/*/SKILL.md
```

### 2단계: 파일 읽기

Read 도구를 사용하여 대상 SKILL.md를 로드한다.

### 3단계: 체크리스트 참조 읽기

`references/anthropic-skills-guide/09-quick-reference.md`에서 평가 기준을 읽는다.

### 4단계: 리뷰 에이전트 실행

Task 도구를 사용하여 `skill-reviewer` 에이전트를 subagent_type `skill-agent-writer:skill-reviewer`로 실행한다. SKILL.md 콘텐츠와 체크리스트를 전달한다.

### 5단계: 보고서 제시

점수 보고서를 사용자에게 표시한다. 다음 중 어떤 것을 할지 묻는다:
- **리팩토링** — 자동으로 수정 적용 (스킬 리팩토링 모드로 전환)
- **특정 항목 수정** — 선택한 약점만 처리
- **완료** — 보고서를 그대로 수락

---

## 스킬 리팩토링 워크플로우

### 1단계: 리뷰 존재 확인

리뷰 보고서가 없으면 먼저 스킬 리뷰를 실행한다 (위의 1-4단계).

### 2단계: 원본 파일 읽기

Read 도구를 사용하여 두 파일을 모두 로드한다:
- 원본 SKILL.md
- 원본 SKILL-ko.md (존재하는 경우)

### 3단계: 리팩토러 에이전트 실행

Task 도구를 사용하여 `skill-refactorer` 에이전트를 subagent_type `skill-agent-writer:skill-refactorer`로 실행한다. 원본 콘텐츠와 리뷰 보고서를 전달한다.

### 4단계: 차이점 표시 및 확인

변경 요약을 사용자에게 제시한다. 수정 내용과 예상 점수 향상을 보여준다.

AskUserQuestion 도구를 사용하여 덮어쓰기 전에 확인한다:
- **모든 변경사항 적용** — 리팩토링된 파일 작성
- **선택적 적용** — 사용자가 유지할 변경사항을 선택
- **취소** — 리팩토링 버전을 폐기

### 5단계: 업데이트된 파일 작성

Edit 도구(선호) 또는 Write 도구를 사용하여 업데이트한다:
1. 리팩토링된 콘텐츠로 SKILL.md 업데이트
2. 업데이트된 한국어 번역으로 SKILL-ko.md 업데이트

---

## 에이전트 작성 워크플로우

### 1단계: 요구사항 수집

AskUserQuestion 도구를 사용하여 수집한다:

1. **에이전트 이름** — kebab-case 식별자 (예: `code-reviewer`)
2. **목적** — 이 에이전트는 무엇을 분석하거나 생성하는가? (1-2문장)
3. **스타일** — 순수 마크다운 (session-wrap 스타일) 또는 프론트매터 (youtube-digest 스타일)?
4. **필요한 도구** — 어떤 도구? (Read, Write, Bash, Grep, Glob, WebSearch 등)
5. **워크플로우 단계** — 에이전트 프로세스의 상위 수준 번호 매겨진 단계
6. **출력 형식** — 에이전트의 출력은 어떤 모습이어야 하는가?
7. **모델 힌트** — `sonnet` (분석/생성) 또는 `haiku` (검증/분류)?

사용자가 부분적인 정보를 제공하면 합리적인 기본값을 채우고 확인한다.

### 2단계: 참조 자료 읽기

생성 전에 관련 참조 파일을 읽는다:

- `${pluginDir}/skills/skill-agent-writer/references/anthropic-agent-harness-guide.md`에서 에이전트 설계 패턴 읽기
- 프로젝트의 에이전트 심층 가이드에서 구조적 패턴 읽기:
  - Glob 도구를 사용하여 프로젝트 루트에서 `docs/05-agents-deep-dive.md` 찾기
  - 발견되면 두 가지 에이전트 스타일과 설계 패턴 읽기

선택적으로 기존 에이전트를 템플릿으로 읽기:
- 순수 마크다운 스타일 예제: `plugins/session-wrap/agents/doc-updater.md`
- 프론트매터 스타일 예제: `plugins/youtube-digest/agents/transcript-extractor.md`

### 3단계: 에이전트를 통한 생성

Task 도구를 사용하여 `agent-generator` 에이전트를 subagent_type `skill-agent-writer:agent-generator`로 실행한다. 수집한 요구사항을 프롬프트로 전달한다.

에이전트가 단일 에이전트 `.md` 파일을 생성한다.

### 4단계: 파일 작성

Write 도구를 사용하여 에이전트 파일을 생성한다:
- `plugins/{plugin-name}/agents/{agent-name}.md` (플러그인 에이전트용)
- 또는 사용자가 지정한 경로

### 5단계: 검증

8점 에이전트 체크리스트에 대해 빠른 리뷰를 실행한다. 0점인 기준이 있으면 플래그하고 수정을 제안한다.

---

## 에이전트 리뷰 워크플로우

### 1단계: 대상 식별

사용자에게 어떤 에이전트 `.md` 파일을 리뷰할지 묻는다. 지정하지 않으면 Glob으로 에이전트 파일을 찾는다:

```
plugins/*/agents/*.md
.claude/agents/*.md
```

### 2단계: 파일 읽기

Read 도구를 사용하여 대상 에이전트 `.md` 파일을 로드한다.

### 3단계: 체크리스트 참조 읽기

`${pluginDir}/skills/skill-agent-writer/references/anthropic-agent-harness-guide.md`에서 에이전트 품질 기준을 읽는다.

### 4단계: 리뷰 에이전트 실행

Task 도구를 사용하여 `agent-reviewer` 에이전트를 subagent_type `skill-agent-writer:agent-reviewer`로 실행한다. 에이전트 `.md` 콘텐츠와 8점 체크리스트를 전달한다.

### 5단계: 보고서 제시

점수 보고서를 사용자에게 표시한다. 다음 중 어떤 것을 할지 묻는다:
- **문제 수정** — 발견 사항을 바탕으로 수동으로 개선 적용
- **완료** — 보고서를 그대로 수락

---

## 사용 시점

- 처음부터 새 스킬을 작성할 때
- 처음부터 새 에이전트를 작성할 때
- 기존 SKILL.md의 품질을 평가할 때
- 기존 에이전트 `.md`의 품질을 평가할 때
- 낮은 점수의 SKILL.md를 개선할 때
- SKILL.md 또는 에이전트 모범 사례를 학습할 때
- 이중 언어(영어 + 한국어) 스킬 커버리지를 보장할 때

## 건너뛸 시점

- 사용자가 이미 완전하고 작동하는 파일을 가지고 있고 변경을 원하지 않을 때
- 작업이 스킬/에이전트 파일이 아닌 애플리케이션 코드 편집에 관한 것일 때
- 사용자가 스킬/에이전트 작성이 아닌 일반적인 Claude Code 사용법에 대해 묻고 있을 때

## 참조 파일

### 스킬 작성 참조

스킬 작성에 대한 상세한 가이드는 번들된 Anthropic 스킬 가이드를 참조한다:

- **`references/anthropic-skills-guide/README.md`** — 개요 및 목차
- **`references/anthropic-skills-guide/01-fundamentals.md`** — 스킬이란 무엇이고 어떻게 작동하는지
- **`references/anthropic-skills-guide/02-planning-and-design.md`** — 작성 전 스킬 계획
- **`references/anthropic-skills-guide/03-skill-structure.md`** — 파일 구조와 프론트매터 형식
- **`references/anthropic-skills-guide/04-writing-instructions.md`** — 명확하고 실행 가능한 지침 작성법
- **`references/anthropic-skills-guide/05-testing.md`** — 스킬 테스트 및 검증
- **`references/anthropic-skills-guide/06-distribution.md`** — 스킬 공유 및 배포
- **`references/anthropic-skills-guide/07-patterns.md`** — 일반적인 스킬 패턴과 템플릿
- **`references/anthropic-skills-guide/08-troubleshooting.md`** — 일반적인 문제와 해결책
- **`references/anthropic-skills-guide/09-quick-reference.md`** — 치트 시트와 체크리스트

### 에이전트 작성 참조

- **`references/anthropic-agent-harness-guide.md`** — Anthropic 블로그 요약: 초기화/코딩 에이전트 패턴, 진행 추적, 실패 모드
- **`docs/05-agents-deep-dive.md`** — 프로젝트 수준 에이전트 가이드: 두 가지 스타일, 실행 패턴, 설계 패턴
