# 화면정의서 오버레이 - 구현 스펙

## 1. 데이터 타입

```typescript
type MoSCoW = 'must' | 'should' | 'could' | 'wont';

interface PolicyRule {
  rule: string;
  condition?: string;
  action: string;
}

interface SubItem {
  id: string;           // e.g. "sub-xxxxx"
  name: string;         // 서브 아이템 이름
  type: '기능' | '정책'; // 유형
  selector: string;     // CSS 셀렉터 (화면 요소 매핑, 빈 문자열이면 뱃지 미표시)
  description: string;  // 상세 설명
}

interface ScreenSection {
  id: string;           // e.g. "SEC-GL-01"
  name: string;
  category: string;
  priority: MoSCoW;
  description: string;
  components: ComponentSpec[];    // {name, type, spec}
  interactions: Interaction[];    // {trigger, action, type}
  dataRequirements: DataRequirement[];  // {field, type, source}
  wireframeType: string;
  policy?: PolicyRule[];
  subItems?: SubItem[];          // N-1, N-2 형식 서브 아이템
}

interface HistoryEntry {
  id: string;
  action: 'edit' | 'delete';
  timestamp: string;
  sectionId: string;
  sectionName: string;
  before: ScreenSection;
  after?: ScreenSection;
}
```

## 2. 데이터 영속화

### Next.js 프로젝트
- **`data/screen-definition-state.json`**: 섹션 데이터 + 히스토리를 JSON 파일로 영속 저장
- **`app/api/screen-definition/route.ts`**: Next.js API Route (GET/PUT)
- git 커밋으로 팀원과 상태 공유

### 순수 HTML/JS 프로젝트
- **localStorage** key `sd-state`로 영속 저장
- 수정/삭제/Undo 시 500ms 디바운스 후 저장

## 3. 전역 상태 관리
- `isPlannerMode`, `isPanelOpen`, `selectedSectionId` — UI 상태
- `sections` — 수정 가능한 섹션 데이터
- `history` — 수정/삭제 히스토리 배열
- `_editData` — 편집 중 임시 데이터 (탭 전환 시 보존)
- `_editSubId` — 서브아이템 편집 상태 (null | 'new' | subItemId)
- `_selectMode` — 화면 컴포넌트 선택 모드 활성 여부
- `_selectHandler` — capture-phase 클릭 핸들러 참조
- `_selectedCompName` — 선택된 컴포넌트 이름 (서브아이템 이름 프리필)
- `_selectedCompSelector` — 선택된 컴포넌트 CSS 셀렉터 (서브아이템 셀렉터 프리필)
- `updateSection(id, updates)` — 섹션 수정 (히스토리 자동 기록 + 자동 저장)
- `deleteSection(id)` — 섹션 삭제 (히스토리 자동 기록 + 자동 저장)
- `undoHistory(historyId)` — 특정 히스토리 항목 되돌리기

## 4. 우측 패널 (480px)
- **핀 모드**: 패널 고정 → 본문 width 자동 조정 `calc(100vw - 480px)`
- **MoSCoW 정렬**: 카테고리 내 must → should → could → wont 순

### 상세보기 (Detail View)
- **4탭 구성**: 기능/정책 | 컴포넌트 | 인터랙션 | 데이터
- **서브아이템 표시**: 기능/정책 탭 하단에 N-1, N-2 형식의 번호가 매겨진 서브 아이템 목록
- **서브아이템 CRUD**: 각 서브아이템에 편집/삭제 버튼, 하단에 추가 버튼
- **항목 클릭 → 컴포넌트 이동**: 리스트에서 항목 클릭 시 해당 DOM 요소로 scrollIntoView + 펄스 하이라이트 애니메이션

### 편집 (Edit View) — 4탭 전체 편집
- **기능/정책 탭**: 이름, 설명, 우선순위(MoSCoW), 정책 규칙 추가/삭제
- **컴포넌트 탭**: 컴포넌트 목록(이름/타입/스펙) 추가/삭제
- **인터랙션 탭**: 인터랙션 목록(트리거/액션/타입) 추가/삭제
- **데이터 탭**: 데이터 요구사항(필드/타입/소스) 추가/삭제
- 탭 전환 시 `collectCurrentTabData()`로 임시 저장 → 최종 저장 시 전체 반영
- `_editData` 임시 상태로 탭 간 데이터 보존

### 공유 (Share)
- Copy as Markdown / Copy as JSON / Download .md / GitHub Issue 복사
- 서브아이템도 내보내기에 포함 (N-1, N-2 번호 포함)

### 삭제 (2단계)
- 첫 클릭: "삭제?" 확인 표시 (3초 타임아웃)
- 두번째 클릭: 실제 삭제 (히스토리에 기록, Undo 가능)

### 히스토리
- 패널 헤더의 Clock 아이콘 → 수정/삭제 이력 + Undo

## 5. 번호 오버레이
- `isPlannerMode` 활성 시 `[data-sd-id]` 요소 스캔
- 각 요소에 **보더 하이라이트** (sky-500 `#0EA5E9`) + **번호 뱃지** (1, 2, ...)
- **서브아이템 뱃지**: `selector` 필드가 있는 서브아이템은 부모 섹션 내에서 해당 요소를 찾아 **서브 번호 뱃지** (1-1, 1-2, ...) + **점선 보더** 표시
- 선택된 항목은 rose-500 `#F43F5E`로 강조
- **뷰포트 클리핑 방지**: 보더/뱃지 위치를 뷰포트 경계 내로 클램핑
- 스크롤/리사이즈/DOM 변경 시 위치 재계산 (requestAnimationFrame + MutationObserver)
- 섹션 뱃지 클릭 → 패널에서 해당 섹션 상세 열림, 서브아이템 뱃지 클릭 → 부모 섹션 상세 열림
- 상태 관리: `subBorderEls`, `subBadgeEls` 객체로 서브아이템 오버레이 추적 (key: `sdId::subId`)

## 6. 컴포넌트 이동 (Scroll-to-Component)
- 패널 리스트에서 항목 클릭 시 `scrollIntoView({behavior:'smooth', block:'center'})`
- 뱃지 클릭 시에도 동일하게 해당 컴포넌트로 스크롤 이동
- 이동 후 CSS `@keyframes sd-pulse` 애니메이션으로 box-shadow 펄스 효과 (0.8s × 2회)
- 스크롤 컨테이너 내부 요소도 올바르게 스크롤되도록 처리

## 7. 서브아이템 (Sub-Items) 기능
기획자가 각 섹션의 내부 컴포넌트별로 기능/정책을 상세히 정의할 수 있다.

### 표시 형식
```
[1] 토픽 분석 위젯 (상위 섹션)
  1-1. 토픽 목록 테이블 [기능] — 뉴스 토픽을 기사수 기준...
  1-2. 토픽 상세 패널 [기능] — 선택된 토픽의 대표 기사...
  1-3. 날짜 탭 전환 [기능] — 어제/오늘 2개 탭...
  1-4. 초기 로드 정책 [정책] — 위젯 로드 시...
```

### 서브아이템 속성
| 속성 | 설명 |
|------|------|
| id | 고유 ID (sub-xxxxx) |
| name | 이름 (예: 토픽 목록 테이블) |
| type | 유형: `기능` 또는 `정책` |
| selector | CSS 셀렉터 (예: `.topic-list`). 화면 요소 매핑용. 값이 있으면 프로토타입 화면에 서브아이템 번호 뱃지 표시 |
| description | 상세 설명 (구현 스펙, 조건, 동작 등) |

### CRUD
- **추가**: 기능/정책 탭 하단 "서브 아이템 추가" 버튼
- **편집**: 각 서브아이템의 연필 아이콘 → 인라인 편집 폼
- **삭제**: 각 서브아이템의 휴지통 아이콘 → 즉시 삭제 (히스토리에 기록)
- **내보내기**: Markdown/JSON/GitHub Issue에 서브아이템 포함

## 8. 화면에서 컴포넌트 선택 (Select-from-Screen)
프로토타입 화면에서 직접 컴포넌트를 클릭하여 서브아이템으로 추가하는 기능.

### 상태 변수
- `_selectMode` — 선택 모드 활성 여부 (boolean)
- `_selectHandler` — capture-phase 클릭 핸들러 참조
- `_selectedCompName` — 선택된 컴포넌트 이름 (프리필 용도)

### 동작 흐름
1. 기능/정책 탭 하단 "화면에서 선택" 버튼 클릭
2. `enableSelectMode()` → `body.sd-select-mode` 클래스 추가 (crosshair 커서, hover 아웃라인)
3. capture-phase 클릭 핸들러로 모든 클릭 인터셉트
4. 프로토타입 화면의 요소 클릭 시:
   - `getComponentInfo(el)` — 클래스명/태그 + 텍스트 힌트로 이름 자동 생성
   - `_selectedCompName`에 저장
   - `disableSelectMode()` → 선택 모드 해제
   - `_editSubId = 'new'` → 새 서브아이템 폼 오픈 (이름 프리필)
5. 서브아이템 저장/취소 시 `_selectedCompName` 초기화

### 핵심 규칙
- `addEventListener('click', handler, true)` — **capture phase** 사용 (일반 이벤트보다 우선)
- `e.stopImmediatePropagation()` — 다른 핸들러 차단
- 패널/토글/뱃지 클릭은 무시 (선택 대상에서 제외)
- 배너에 "화면에서 컴포넌트를 클릭하세요" 안내 + 취소 버튼 표시
- **`findMeaningfulEl(el)`** — 클릭된 요소가 `span`, `small`, `strong` 등 인라인 태그면 클래스/ID가 있는 부모 요소까지 자동 탐색 (더 정확한 셀렉터 생성)
- **SVG 차단** — 선택 모드에서 `svg *`의 `pointer-events: none`으로 차트 내부 hover 원 등이 클릭을 가로채지 않도록 처리

## 9. 텍스트 개행 처리
모든 텍스트 필드(설명, 정책 액션, 컴포넌트 스펙, 서브아이템 설명 등)에서 `\n` 줄바꿈이 올바르게 표시되어야 한다.

### 렌더링 (패널 표시)
- CSS `white-space: pre-line` 적용 대상:
  - `.sd-tab-section p` — 설명 텍스트
  - `.sd-tab-section td` — 테이블 셀 (정책/컴포넌트/인터랙션/데이터)
  - `.sd-sub-desc` — 서브아이템 설명
- `esc()` 함수로 HTML 이스케이프 후 `\n`이 CSS에 의해 줄바꿈으로 표시
- textarea는 기본적으로 `\n`을 줄바꿈으로 처리하므로 편집 시 추가 처리 불필요

### 내보내기 (Markdown/GitHub)
- `mdCell(s)` 헬퍼: Markdown 테이블 셀 내 `\n` → `<br>`, `|` → `\|` 이스케이프
- `toMarkdown()` — 테이블 외 텍스트(설명, 서브아이템)는 `\n` 그대로 유지 (Markdown 줄바꿀)
- `toGitHubIssue()` — 리스트 항목 내 `\n` → 공백으로 치환 (한 줄 유지)
- `toJSON()` — `\n` 원본 그대로 보존

## 10. 내보내기 유틸
- `toMarkdown(section)` — 서브아이템 포함 마크다운
- `toJSON(section)` — 전체 데이터 JSON
- `toGitHubIssue(section)` — 서브아이템별 체크리스트 포함
- `downloadMd(section)` — .md 파일 다운로드

## 11. 프로토타입 페이지에 data-sd-id 속성 추가
프로토타입의 각 화면 컴포넌트 DOM에 `data-sd-id="SEC-XX-XX"` 속성을 추가하면 자동으로 오버레이에 표시됨.

### 섹션 ID 네이밍 규칙
```
SEC-{카테고리}-{번호}

카테고리 예시:
  GL = Global Layout       SR = Search
  DB = Dashboard           MT = Mention Trend
  CH = Channel             SA = Sentiment Analysis
  SK = Sentiment Keyword   DA = Deep Analysis
  RK = Related Keyword     IF = Influencer
  HI = Hot Issue           RP = Report
  CP = Compare             NW = Network
  CM = Community           DT = Data Table
  SB = Sidebar             HD = Header
  CT = Chat Content        IN = Input
  FT = Footer
```

프로젝트 특성에 맞게 카테고리를 추가/변경하여 사용한다.
