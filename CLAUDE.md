# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**minigamegroup** — 브라우저에서 동작하는 5가지 클래식 게임 허브. 빌드 도구·프레임워크·패키지 매니저 없이 순수 HTML/CSS/JS로 구성된 정적 사이트.

- **라이브 사이트**: https://jasonbaek-oss.github.io/minigamegroup/
- **GitHub**: https://github.com/jasonbaek-oss/minigamegroup

## 배포

별도 빌드 과정 없음. `index.html`을 브라우저에서 직접 열거나 로컬 서버를 사용:

```powershell
# PowerShell (Python 설치된 경우)
python -m http.server 8080

# 또는 VS Code Live Server 확장 사용
```

파일 수정 후 자동으로 commit + push → GitHub Pages가 수십 초 내 재배포됨 (PostToolUse 훅).

## Git / 자동 커밋

`.claude/settings.json`에 PostToolUse 훅이 설정되어 있어 **Write 또는 Edit 후 자동으로 `git add -A → commit → push`** 실행됨.

수동으로 의미 있는 커밋을 남길 때는 직접 커밋 메시지를 작성:

```powershell
git add -A
git commit -m "feat: ..."
git push origin master
```

## 파일 구조

```
index.html          # 게임 허브 홈페이지 (5개 게임 카드 + 허브 네비게이션)
minesweeper.html    # 레거시 단독 실행 파일 (허브와 무관)
games/
  minesweeper.html  # 지뢰찾기
  2048.html         # 2048 퍼즐
  snake.html        # 스네이크
  tetris.html       # 테트리스
  breakout.html     # 브레이크아웃
```

## 아키텍처 패턴

**모든 게임 페이지는 동일한 구조를 따름:**

1. **CSS 변수** — 다크 테마 공통 팔레트 (`--bg: #1a1a2e`, `--panel: #16213e`, `--accent: #4fc3f7` 등)
2. **고정 Nav** — `← 허브로` 뒤로가기 링크(`../index.html`) + 게임 제목 (게임별 고유 색상)
3. **게임 로직** — 페이지 하단 `<script>` 태그 내 인라인 JS, 외부 의존성 없음
4. **Canvas 기반 게임** (snake, tetris, breakout) vs **DOM 기반** (2048, minesweeper)

**새 게임 추가 시 체크리스트:**
- `games/[name].html` 생성 — 기존 게임 파일의 nav/CSS 구조 복사
- `index.html` 게임 그리드에 카드 추가 (`.games-grid` 내 `<a>` 태그)
- 카드 색상 클래스 추가 (`.c-[name]` 패턴으로 border, hover, glow, play-btn 색 정의)

## 게임별 핵심 구현

| 게임 | 렌더링 | 주요 상태 변수 | 입력 |
|------|--------|--------------|------|
| 지뢰찾기 | DOM grid | `grid[][]`, `revealed[][]`, `flagged[][]` | 좌/우클릭 |
| 2048 | CSS div tiles | `board[4][4]` (숫자 배열) | 방향키, 터치 스와이프 |
| 스네이크 | Canvas 400×400 | `snake[]` (좌표 배열), `dir`, `food` | 방향키 |
| 테트리스 | Canvas 280×560 + 112×112 (next) | `board[][]`, `current`, `next` piece | 방향키, Space(하드드롭) |
| 브레이크아웃 | Canvas 460×520 | `paddle`, `ball`, `bricks[]` | 마우스, 방향키 |

**테트리스 회전 알고리즘** (`rotateCW`): 행렬 전치 후 역순 — `result[c][r] = matrix[rows-1-r][c]`

**지뢰찾기 첫 클릭 보호**: `placeMines(safeR, safeC)` 는 첫 클릭 칸 및 주변 3×3 영역을 `safe` Set에 등록 후 지뢰 배치.

## 디자인 시스템

모든 파일이 동일한 CSS 변수를 인라인으로 선언:

```css
--bg: #1a1a2e;    /* 페이지 배경 */
--panel: #16213e; /* 카드/패널 배경 */
--blue: #0f3460;  /* 보조 배경 */
--red: #e94560;   /* 지뢰찾기 포인트 컬러 */
--accent: #4fc3f7;/* 기본 강조색 (허브/스네이크) */
--green: #81c784; /* 스네이크 포인트 컬러 */
--text: #e8eaf6;  /* 본문 텍스트 */
--muted: #8892b0; /* 보조 텍스트 */
```

게임별 포인트 컬러: 지뢰찾기 `#e94560` · 2048 `#ff9800` · 스네이크 `#81c784` · 테트리스 `#ce93d8` · 브레이크아웃 `#4fc3f7`
