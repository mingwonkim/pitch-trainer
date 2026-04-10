# MK.HUB Memory

프로젝트의 주요 의사결정 사항을 기록한 문서. Second brain으로 활용하여 과거 결정의 맥락과 근거를 빠르게 참조할 수 있도록 함. 목차별로 마주했던 패턴, 해결책, 의사결정 이유를 기록한다.

---

## 아키텍처 결정

### 빌드 도구 없음 (Vanilla HTML/CSS/JS)
- **결정**: 빌드 도구(Webpack, Vite 등) 미사용, 파일 직접 수정
- **이유**: 정적 GitHub Pages 배포에 최적화, 복잡도 최소화
- **배포**: GitHub Pages → 민권.com (CNAME)

### SPA 라우팅 방식
- **결정**: URL 변화 없이 `.active` 클래스 토글로 페이지 전환
- **이유**: 정적 호스팅 환경에서 서버 라우팅 불필요
- **함수**: `setPage(p)`, `navHistory` 배열로 뒤로가기 관리

### Firebase 비동기 로드
- **결정**: `Promise.all([import(...)])` 방식, 로컬 파일 우선 → CDN 폴백
- **이유**: CDN 장애 대비 오프라인 보험
- **주의**: `db`, `stg`, `auth`는 로드 전 `null` → 모든 DB 함수 `async/await` 필수

---

## 패턴 및 해결책

### PIN 잠금 & iframe 통신
- `showPin()` / `showTextPin()` 오버레이 표시
- `notifyAllFrames(true)` → `postMessage`로 갤러리 iframe에 해제 상태 전파
- 갤러리(1~5picture.html)는 `{type:'mk_unlock', value:bool}` 메시지 수신

### 갤러리 반응형
- 데스크탑: iframe 폴라로이드 레이아웃
- 모바일: `.mob-gal-item` 썸네일 → `#picExpandModal` fullscreen

---

## 의사결정 로그

| 날짜 | 결정 | 이유 |
|------|------|------|
| 2026-04-10 | memory.md 초기 생성 | 프로젝트 컨텍스트 보존 |
