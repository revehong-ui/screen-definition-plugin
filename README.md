# Screen Definition Plugin for Claude Code

프로토타입 화면 위에 **화면정의서(Screen Definition) 오버레이**를 자동 구축하는 Claude Code 플러그인입니다.

## 주요 기능

- **번호 오버레이**: 프로토타입 화면의 각 컴포넌트에 번호 뱃지 + 보더 하이라이트
- **우측 패널**: 4탭 상세보기 (기능/정책, 컴포넌트, 인터랙션, 데이터)
- **인라인 편집**: 패널에서 바로 수정/삭제, 히스토리 + Undo 지원
- **서브아이템**: 각 섹션 내부 컴포넌트별 기능/정책 상세 정의 (N-1, N-2 형식)
- **화면 선택**: 프로토타입에서 직접 클릭하여 서브아이템 추가
- **내보내기**: Markdown / JSON / GitHub Issue

## 설치

```bash
claude plugin add github:your-username/screen-definition-plugin
```

## 사용법

Claude Code에서:

```
/screen-definition
```

이 명령을 실행하면 현재 프로젝트에 화면정의서 오버레이 시스템이 자동 구축됩니다.

## 지원 프레임워크

| 프레임워크 | 영속화 방식 |
|-----------|-----------|
| Next.js (App Router) | JSON 파일 + API Route (git 공유 가능) |
| 순수 HTML/JS | localStorage |

## 전제 조건

- 추가 npm 패키지 불필요 (React, Lucide, Tailwind 등 기존 의존성만 사용)
- 프로토타입 DOM에 `data-sd-id="SEC-XX-XX"` 속성 추가 필요

## 라이선스

MIT
