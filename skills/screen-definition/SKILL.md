---
name: screen-definition
description: >
  This skill should be used when the user asks to "화면정의서 만들어줘",
  "화면정의서 오버레이 구축해줘", "프로토타입에 화면정의서 붙여줘",
  "screen definition overlay", "화면 스펙 오버레이", "화면 명세서 만들어줘",
  "컴포넌트 정의서 만들어줘", or mentions building a screen definition
  system for prototypes. Also triggers when the user discusses prototype
  review workflows, planner mode overlays, component specification tools,
  or "기획-개발 핸드오프" annotation workflows.
---

# Screen Definition Overlay System

프로토타입 화면 위에 **화면정의서 오버레이**를 자동 구축하는 스킬.
기획자가 프로토타입을 보면서 각 컴포넌트의 기능 명세/정책/인터랙션/데이터 요구사항을 확인하고, 수정/삭제/공유할 수 있는 도구를 생성한다.

## Overview

```
+-------------------------------------------+-----------------+
|  Prototype Screen                         |  Screen Def.    |
|                                           |  Panel          |
|  +--[1]----------------+                  |                 |
|  | Component A          |                 | [1] Component A |
|  +---------------------+                  |  Must-Have      |
|                                           |  1-1. Sub item  |
|  +--[2]----------------+                  |  1-2. Sub item  |
|  | Component B          |                 |                 |
|  +---------------------+                  | [Edit] [Delete] |
|                                           | [Share]         |
+-------------------------------------------+-----------------+
```

## Core Capabilities

- **Numbered Overlay**: `data-sd-id` 속성이 있는 DOM 요소에 번호 뱃지 + 보더 하이라이트 자동 표시
- **Right Panel (480px)**: 4탭 상세보기 (기능/정책, 컴포넌트, 인터랙션, 데이터) + 인라인 편집
- **Sub-Items (N-1, N-2)**: 각 섹션 내부 컴포넌트별 기능/정책 상세 정의
- **Select-from-Screen**: 프로토타입에서 직접 클릭하여 서브아이템 추가
- **History + Undo**: 모든 수정/삭제 히스토리 자동 기록, 되돌리기 지원
- **Export**: Markdown / JSON / GitHub Issue 내보내기
- **Persistence**: Next.js는 JSON 파일 + API Route, 순수 HTML은 localStorage

## Execution

To build the overlay system, invoke `/screen-definition`. The build process:

1. Analyze project structure (Next.js vs HTML/JS)
2. Create framework-appropriate file structure
3. Write section data (defaults + sub-items)
4. Implement global state management (with persistence)
5. Build right panel (4-tab detail + 4-tab edit + sub-item CRUD + history)
6. Build numbered overlay (borders + badges + viewport clamping + scroll-to)
7. Create export utilities (with sub-items)
8. Add `data-sd-id` attributes to prototype pages
9. Verify build

### Prerequisites

| Item | Requirement |
|------|-------------|
| Framework | Next.js (App Router) or plain HTML/JS |
| Styling | Tailwind CSS or CSS-in-JS (auto-injected) |
| Extra packages | None (uses existing dependencies only) |

## Critical Rules

- No new npm packages required
- Preserve existing project design system CSS variables
- Overlay border color MUST differ from project key color
- All `data-sd-id` elements get automatic numbered badges + borders
- Edit/delete actions auto-record to history with Undo support
- Borders/badges clamped within viewport boundaries
- MutationObserver watches content container only (not body, to prevent infinite loops)
- Pin mode uses `width: calc(100vw - 480px)` (not margin-right)
- Capture-phase click handler for select-from-screen mode
- `white-space: pre-line` for newline rendering in panel text

## Additional Resources

### Reference Files

- **`references/implementation-spec.md`** - Full implementation specification: data types (ScreenSection, SubItem, HistoryEntry), state management details, panel UI spec, overlay rendering rules, select-from-screen workflow, text newline handling, export utilities, and section ID naming conventions
- **`references/guide.md`** - Usage guide: team role workflows, MoSCoW priority definitions, data persistence flow diagrams, and step-by-step usage instructions for planners
