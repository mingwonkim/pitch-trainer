# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**MK.HUB** — 개인용 정적 웹 앱. 빌드 도구 없음. HTML/CSS/JS 파일을 직접 수정하면 바로 반영됨.  
배포: GitHub Pages → 민권.com (CNAME.txt 참조)

## 파일 구조

```
index.html          ← 메인 SPA (~3,800줄). 모든 페이지·CSS·JS가 이 파일 안에 있음
pitch-trainer.html  ← 절대음감 트레이너 (music iframe)
chord-dictionary.html ← 코드 사전 (music iframe)
quotes.html / code-editor.html ← 독립 서브 앱
1picture.html ~ 5picture.html  ← 갤러리 서브 앱 (hub에서 iframe으로 삽입)
firebase/           ← Firebase SDK 로컬 복사본 (CDN fallback용)
*.backup / *.working ← 수정 시 생성되는 버전 파일 (원본 보존용)
```

## 아키텍처

### SPA 라우팅 (index.html)
- 모든 페이지는 `<div class="page" id="page-X">` 형태로 DOM에 공존
- `setPage(p)` 함수가 `.active` 클래스 토글로 페이지 전환 (URL 변화 없음)
- 페이지 ID: `hub`, `memo`, `files`, `music`, `settings`, `todo`
- `navHistory` 배열로 뒤로가기 스택 관리

### Firebase 연동
- **Firestore** (DB) + **Storage** (파일) + **Auth** (이메일 링크 로그인)
- Firebase 모듈은 `Promise.all([import(...)])` 방식으로 **비동기 로드** (로컬 파일 우선 → CDN 폴백)
- `db`, `stg`, `auth` 변수는 Firebase 로드 전까지 `null` → 모든 DB 함수는 `async/await` 필수
- Firestore 경로: `mk_app/data/{collection}` (memos, files, folders, todos 등)

### PIN 잠금 시스템
- `isUnlocked` (변수) + `sessionStorage('mk_unlocked')` 로 잠금 상태 관리
- `showPin(title, callback)` — 숫자 PIN 오버레이 표시
- `showTextPin(callback)` — 텍스트 입력 방식 PIN
- PIN 해제 후 `notifyAllFrames(true)` 로 `postMessage`를 통해 갤러리 iframe들에게 해제 상태 전파
- 갤러리 iframe (1~5picture.html)은 `message` 이벤트로 `{type:'mk_unlock', value:bool}` 수신

### Music 앱 (iframe 오버레이)
- `openMusicApp('pitch'|'chord')` 호출 시 `#musicAppView`(position:fixed, inset:0)가 전체화면으로 열림
- 내부 iframe (`#musicFrame`)이 `pitch-trainer.html` 또는 `chord-dictionary.html` 로드
- `pitch-trainer.html` / `chord-dictionary.html`: `body { height:100dvh; overflow:hidden }` + `.app { height:100% }` + `.sc { flex:1; overflow-y:auto }` 구조

### 갤러리 (Hub 페이지)
- 데스크탑: `1~5picture.html`이 iframe으로 삽입되어 폴라로이드 레이아웃 표시
- 모바일: `.mob-gal-item`으로 썸네일 표시, 클릭 시 `#picExpandModal`(fullscreen) 에서 열림
- iframe 간 통신: `notifyEdFrame / notifyLtFrame / notifyMbFrame / notifyP4Frame / notifyP5Frame`

### 로컬 저장소 패턴
- `localStorage`: 비민감 설정 (카드 순서, 폴더 순서, 마스터 라벨, PIN 해시)
- `sessionStorage('mk_unlocked')`: 탭 세션 중 잠금 해제 상태
- 민감 데이터(메모, 파일, 할 일)는 Firestore에만 저장

## 코드 수정 규칙 (CLAUDE.md 글로벌 설정)

- 원본 파일 최초 1회 `.backup` 복사 후 `.working` 파일에서 작업
- `.working` 파일을 반복 수정하며 완성 → 원본에 덮어쓰기
- `index.html`은 258KB+ 대용량 — 전체 읽기 대신 `grep`/`Read(offset+limit)` 활용

## 개발 & 배포

빌드 도구 없음. 로컬 테스트는 브라우저에서 직접 열거나 `python -m http.server` 사용.  
배포는 GitHub에 push → GitHub Pages 자동 반영.

## 주의사항

- `index.html` 내 CSS는 파일 상단 `<style>` 블록에 모두 집중 (Tailwind 유틸리티와 혼용)
- Firebase SDK 버전: `FB_VER` 변수로 관리 (로컬 복사본과 CDN 버전 동일하게 유지)
- `--nav-h` CSS 변수: JS로 실측 (`nav.offsetHeight`) — `var(--nav-h, 64px)` 폴백 있음
- 갤러리 iframe에 `will-change:transform` 금지 (position:fixed 자식의 containing block 오염)
