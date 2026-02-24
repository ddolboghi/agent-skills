---
name: decision-first
description: |
  A skill that extracts all details requiring user decisions before executing a task, presents them as a numbered list with recommended options, then executes the task strictly following the user's selections.

  Use when:
  - The user asks to "extract decision points", "list choices", "identify what needs to be decided", or similar
  - The user invokes "/decision-first <prompt>"
  - A task prompt contains ambiguous or undecided implementation details and the user wants to choose before execution
---

# Decision First

Extract all details requiring user decisions before executing a task, collect selections, then execute strictly following those choices.

## Language Rule

**Respond in the same language the user used to invoke or interact with this skill.** Default is English.
- If the user's prompt or message is in Korean, all output (decision list, confirmation summary, execution commentary) must be in Korean.
- If the user's prompt or message is in Japanese, respond in Japanese. Same for any other language.
- Technical terms, code, and file paths remain as-is regardless of language.

## Workflow

### Phase 1: Extract and Present Decision Points

1. **Analyze the task prompt** and any referenced files or context.
2. **Detect the user's language** from their prompt. Use that language for all output in every phase. Default to English if unclear.
3. **Extract every ambiguous or undecided detail** — implementation approach, UX patterns, data structures, output format, scope, etc. Include anything where the choice changes the outcome.
4. **Output a numbered list** in this format (translated to the user's language):

```
Please decide on the following details:

1. [Decision title]:
   a. [Option description]
   b. [Option description] (recommended)
   c. [Option description]

2. [Decision title]:
   a. [Option description] (recommended)
   b. [Option description]
```

**Format rules:**
- Items are numbered (1, 2, 3...), options use lowercase letters (a, b, c...)
- Mark one recommended option per item with `(recommended)`
- Minimum 2 options per item; up to 3–4 when appropriate
- Descriptions should be concise but make the differences between options clear
- Include inline code examples where helpful (e.g. `` `⋮` ``)

5. After the list, **append this instruction prompt** (translated to the user's language):

```
Select an option for each item (e.g. 1-b, 2-a, 3-c).
You may also type a custom option not in the list — it will be adopted as-is.
```

### Phase 2: Collect User Selections

When the user responds:
- `1-b` — adopt option b for item 1
- `1-b, 2-a` — adopt multiple selections at once
- `2-<custom text>` (number + text not matching any listed option) — adopt that custom text literally as the selection for that item
- Number only (e.g. `2`) — auto-apply the recommended option for that item
- Partial response — auto-apply the recommended option for any unspecified items (state this explicitly)

Output a **selection confirmation summary**:

```
Selection confirmed:
1. [Decision]: [Adopted option]
2. [Decision]: [Adopted option] (recommended auto-applied)
...

Proceeding with the above selections.
```

### Phase 3: Execute the Task

Execute the task **strictly following** the adopted selections.

**Execution rules:**
- Never deviate from the adopted selections
- If a new undecided detail is discovered mid-task, stop immediately and ask the user for an additional decision
- Custom options typed by the user must be reflected literally

## Example

**User input:**
> `/decision-first` Referring to @spec.md, write a `### 4.2 Workspace Page` requirements table in @spec.md.

**Phase 1 output:**
```
Please decide on the following details:

1. UX for edit / delete / invite action buttons on workspace cards:
   a. Individual action buttons visible on each card
   b. Actions accessed via kebab menu (e.g. `⋮`) on the card (recommended)
   c. Actions accessed via accordion on the card

2. Default sort order for the workspace list:
   a. Creation date ascending
   b. Last modified date descending (recommended)
   c. Name alphabetical order

Select an option for each item (e.g. 1-b, 2-a, 3-c).
You may also type a custom option not in the list — it will be adopted as-is.
```

**User selection:** `1-b, 2-c`

**Phase 2 output:**
```
Selection confirmed:
1. Action button UX: Actions accessed via kebab menu (⋮)
2. Sort order: Name alphabetical order

Proceeding with the above selections.
```

**Phase 3:** Write the requirements table in spec.md reflecting the selected options.

### Example (Korean)

**User input:**
> `/decision-first` @spec.md 를 참고해서 `### 4.2 워크스페이스 페이지` 요구사항 테이블을 @spec.md 에 작성해줘.

**Phase 1 output:**
```
아래 항목에 대해 결정해 주세요:

1. 워크스페이스 카드의 편집/삭제/초대 액션 버튼 UX:
   a. 각 카드에 개별 액션 버튼 노출
   b. 카드의 케밥 메뉴(`⋮`)를 통해 액션 접근 (recommended)
   c. 카드의 아코디언을 통해 액션 접근

2. 워크스페이스 목록 기본 정렬 순서:
   a. 생성일 오름차순
   b. 최종 수정일 내림차순 (recommended)
   c. 이름 가나다순

각 항목의 옵션을 선택해 주세요 (예: 1-b, 2-a, 3-c).
목록에 없는 커스텀 옵션을 직접 입력할 수도 있습니다.
```

**User selection:** `1-b, 2-c`

**Phase 2 output:**
```
선택 확인:
1. 액션 버튼 UX: 케밥 메뉴(⋮)를 통해 액션 접근
2. 정렬 순서: 이름 가나다순

위 선택 사항으로 진행합니다.
```

**Phase 3:** 선택된 옵션을 반영하여 spec.md에 요구사항 테이블을 작성한다.
