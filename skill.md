# 세미나 빌더 — 개발 & Claude 작업 가이드

**파일**: `seminar_builder_v4.html` (단일 파일)  
**배포**: https://seminar-build.vercel.app  
**GitHub**: madamddo-stack/seminar-build (main push → Vercel 자동 배포)

---

## 문서 구조

| 파일 | 내용 |
|------|------|
| [PRD.md](PRD.md) | 제품 개요, 사용자 시나리오, 기능 요구사항, 백로그 |
| [design.md](design.md) | 폰트·색상·안내문·배너 레이아웃 스펙 |
| [skill.md](skill.md) | 기술 스택, JS 함수 맵, html2canvas 원리, 작업 주의사항 |

---

## Claude 작업 시 핵심 주의사항

1. **html2canvas 캡처 전 transform/zoom 반드시 제거**  
   `getBoundingClientRect()`가 시각적 크기로 읽히는 문제 → 제거 후 캡처, 복원 패턴 유지

2. **html2canvas 미지원 CSS 우회**  
   `filter:invert`, `:has()`, `:empty`, `flex align-items:center` (position:absolute 내부) — 아래 섹션 참고

3. **배너 버튼 수직 정렬은 padding-top 방식 고수**  
   flex 대신 `padding-top + line-height:0 + vertical-align:top`

4. **히스토리 자동 저장 트리거**  
   안내문 저장 + 배너 저장 **양쪽 완료** 시 `saveHistory()` 자동 호출

---

## 기술 스택

| 항목 | 상세 |
|------|------|
| 구조 | 단일 HTML 파일 (`seminar_builder_v4.html`) |
| 언어 | 순수 HTML / CSS / JS (프레임워크 없음) |
| 폰트 | Paperlogy (CDN), Noto Sans KR / Lexend / Source Sans 3 (Google Fonts) |
| PNG 변환 | html2canvas v1.4.1 (CDN) |
| 클립보드 | ClipboardItem API |
| 배포 | Vercel (GitHub 연동, main 브랜치 push → 자동 배포) |

---

## 파일 구조

```
seminar-builder/
├── seminar_builder_v4.html   ← 전체 소스 (단일 파일)
├── vercel.json               ← "/" → v4.html 리다이렉트
├── img/
│   └── bi_jk.svg             ← 잡코리아 로고
├── PRD.md
├── design.md
└── skill.md
```

---

## UI 구조

```
[상단 탭바 (#topbar)]
  로고 | 프로젝트명 입력 | 안내문/배너 저장 | 미리보기 | 다운로드

[좌측 레일 (#left-rail)]
  안내문 | 배너 | 히스토리

[안내문 페이지 (#guide-page)]
  좌측 사이드바 (304px) — 서브탭: 로고/뱃지 | 타이틀 | 이미지 | 개요 | 상세 | 디자인
  중앙 미리보기 (#canvas-area) — 900px 실시간 렌더, zoom 자동 조정
  우측 패널 (#right-panel) — 팔레트 / Inspect

[배너 페이지 (#banner-page)]
  좌측 사이드바 — 서브탭: 텍스트 | 이미지 | 버튼 | 저장
  중앙 미리보기 — 4종 세로 나열 (실제 픽셀 scale:1 표시)
```

---

## 주요 JS 함수 맵

### 페이지 전환

| 함수 | 역할 |
|------|------|
| `switchPage(page, btn)` | 안내문 / 배너 / 히스토리 탭 전환 |
| `gTab(id, btn)` | 안내문 사이드바 서브탭 |
| `bTab(id, btn)` | 배너 사이드바 서브탭 |

### 렌더링

| 함수 | 역할 |
|------|------|
| `rg()` | 안내문 전체 렌더 |
| `rb()` | 배너 4종 전체 리렌더 |
| `rbOne(type, html, scale)` | 단일 배너 렌더 + 래퍼 크기 보정 |
| `rV200(d)` | 기업라운지좌측 200×275 렌더 |
| `rV170(d)` | 기업검색좌측 170×235 렌더 |
| `rStrip(d)` | 기업라운지띠 940×80 렌더 |
| `rNews(d)` | 뉴스레터 535×274 렌더 |
| `getBnData()` | 배너 렌더용 데이터 수집 |
| `bnTitleLines(title, fontSize, ...)` | 타이틀 줄별 렌더 HTML 생성 |
| `bnArrow(color, size, valign)` | 배너 화살표 SVG 생성 |
| `fitGuideRoot()` | 안내문 zoom 자동 계산 (ResizeObserver) |

### 색상

| 함수 | 역할 |
|------|------|
| `applyPalette(idx)` | 팔레트 일괄 적용 |
| `applyBg/Main/TitleC/SubC/BtnC(v)` | 개별 색상 적용 + 스와치 동기화 |
| `getOvr(type, field, fallback)` | 배너 개별 오버라이드 값 조회 |

### 캡처 / 저장

| 함수 | 역할 |
|------|------|
| `saveCtx()` | 현재 페이지 저장 (안내문/배너 분기) |
| `saveGuideCapture()` | 안내문 PNG → `_pvData.guide` |
| `saveBannerCapture()` | 배너 4종 PNG → `_pvData.banners` (순차 처리) |
| `toBlob(type)` | 개별 배너 html2canvas → Blob |
| `pngSaveBanner(type)` | 배너 PNG 파일 저장 |
| `pngCopyBanner(type)` | 배너 PNG 클립보드 복사 |
| `pngSaveAll()` | 배너 4종 순차 저장 |
| `dlGuidePng()` | 안내문 PNG 다운로드 |
| `openPreview()` | 미리보기 새 탭 오픈 |

### 히스토리 / Draft

| 함수 | 역할 |
|------|------|
| `saveHistory()` | 현재 상태 localStorage 저장 (`sb_history_v1`, 최대 50건) |
| `renderHistory()` | 히스토리 카드 렌더 |
| `restoreHistory(id)` | 히스토리 불러오기 |
| `deleteHistory(id)` | 히스토리 항목 삭제 |
| `saveDraft()` | 편집 상태 자동저장 (`sb_draft_v1`) |
| `scheduleAutoSave()` | 1.5초 debounce 후 saveDraft |
| `restoreDraft()` | 이전 작업 복원 |

---

## html2canvas 캡처 원리

### 안내문 캡처 (`saveGuideCapture`)

```
① _buildInvertedLogoMap()
   다크모드 로고 filter:invert → html2canvas 미지원
   → canvas2d로 미리 반전 data URL 생성

② guide-root zoom 제거 (live DOM)
   → getBoundingClientRect()가 900px 그대로 읽힘
   → 캡처 완료 후 zoom 복원

③ requestAnimationFrame × 2 — 레이아웃 반영 대기

④ onclone 콜백:
   - canvas-area overflow:visible (클립 방지)
   - guide-root box-shadow 제거
   - 다크 로고 src → 반전 data URL 교체
   - 에디터 UI 요소 숨김 (.ov-type-bar, .img-overlay 등)
   - att-wrap 빈 경우 DOM 물리 제거 (CSS :has 미지원 대응)
   - contenteditable 속성 제거
```

### 배너 캡처 (`saveBannerCapture`, `toBlob`)

```
① bn-stage transform:scale 제거 (live DOM)
   → getBoundingClientRect()가 자연 크기로 읽힘 (200×275 등)
   → 캡처 완료 후 transform 복원

② bn-stage.firstElementChild (bn-canvas) 캡처

③ scale:1 → 스펙 픽셀 크기 그대로 출력

④ 4종 순차 처리 (transform 제거/복원 충돌 방지)
```

### html2canvas 알려진 한계 및 해결책

| 현상 | 원인 | 해결책 |
|------|------|--------|
| `filter:invert()` 이미지 미적용 | html2canvas 미지원 | canvas2d pre-invert |
| `:has()`, `:empty` 미작동 | html2canvas 미지원 | JS DOM 직접 조작 |
| `position:absolute` 내 `flex align-items:center` 오작동 | 렌더러 한계 | `padding-top + line-height:0 + vertical-align:top` |
| `transform:scale` 캡처 크기 왜곡 | getBoundingClientRect 시각적 크기 반환 | 캡처 전 transform 제거 후 복원 |
| `box-shadow` 과도 렌더 | 브라우저와 렌더링 차이 | onclone에서 box-shadow 제거 |

---

## 배너 버튼 수직 정렬 원칙

고정 높이 버튼(v200: 38px, v170: 36px)은 **flex 사용 금지**, 아래 방식 사용:

```html
<div style="height:38px; padding-top:9px; line-height:0;
            text-align:center; box-sizing:border-box; overflow:hidden;">
  <span style="display:inline-block; vertical-align:top; line-height:20px;">텍스트</span>
  <svg style="display:inline-block; vertical-align:top;">...</svg>
</div>
```

**계산식**: `padding-top = (버튼 높이 - SVG 높이) / 2`
- v200: (38 - 20) / 2 = **9px**
- v170: (36 - 18) / 2 = **9px**

pill 버튼(strip, news)은 `padding:9px 14px`으로 자연 중앙 정렬.

---

## 로컬 실행 / 배포

```bash
# 로컬
python3 -m http.server 3000

# 배포 (Vercel 자동)
git add seminar_builder_v4.html
git commit -m "..."
git push origin main
# → https://seminar-build.vercel.app (1~2분 소요)
```
