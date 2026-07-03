# Task 3 Report -- 配置面板 + 页面切换动画

**Status:** DONE
**Date:** 2026-07-03
**Base Commit:** 7751869

---

## What was implemented

### 1. Quiz page HTML shell
Replaced the `<section id="quiz">` placeholder with the full quiz page structure:
- **Toolbar** (`<header class="quiz-toolbar">`): Back button (arrow icon), title ("配置面板"), three selectors (chapter, type, count)
- **Main area** (`<main class="quiz-main" id="quizMain">`): Placeholder for Task 4 question rendering
- **Footer** (`<footer class="quiz-footer" id="quizFooter">`): Progress bar with fill indicator, progress stats (current/total, correct rate)

Chapter selector covers all 9 chapters from `questions.json`:
1. 绪论, 2. 线性表, 3. 栈和队列, 4. 串, 5. 数组和广义表, 6. 树和二叉树, 7. 图, 8. 查找, 9. 排序

Type selector: 全部题型 / 选择题 (single) / 填空题 (fill)
Count selector: 全部 / 10 / 20 / 30 / 50 / 100

### 2. CSS additions
- `#quiz.page.active` override: uses flex column with stretch alignment instead of centered
- `.quiz-toolbar`: fixed top bar with surface background, border, z-index
- `.btn-icon`: 40x40 transparent button with border, hover/active states
- `.toolbar-title`: white-space nowrap, flex-shrink 0
- `.toolbar-selectors`: flex row with wrapping, center alignment
- `.sel`: styled select dropdowns matching dark theme
- `.quiz-main`: flex column with auto scroll, max-width 700px
- `.quiz-footer`: fixed bottom bar mirroring toolbar style
- `.progress-bar` / `.progress-bar-fill`: 4px height with purple fill, smooth width transition
- `.progress-stats`: space-between layout with secondary text
- `.correct-rate`: green color for correct rate display
- Responsive breakpoints: 768px (hide title, compact toolbar), 480px (compact everything, smaller fonts)

### 3. JavaScript
- **`allQuestions` global variable**: stores fetched questions
- **`loadQuestions()` IIFE**: fetches `questions.json` on page load, stores in `allQuestions`
- **`switchPage(fromId, toId)`**: Replaces `navigateTo()`. Adds `exit` class to from-page (keeping `active` for visible animation), then after 400ms removes exit/active from old page and adds active to new page. Smooth CSS transition via opacity + translateY.
- **`getConfig()`**: Reads from `localStorage` key `ds_quiz_config`, returns default `{ chapter: 'all', type: 'all', count: 'all' }`
- **`saveConfig(config)`**: Writes config to `localStorage` key `ds_quiz_config`
- **`shuffleArray(arr)`**: Fisher-Yates shuffle returning a new shuffled copy
- **`filterQuestions(config)`**: Filters `allQuestions` by chapter number and question type
- **`startQuiz()`**: Stub that filters and shuffles questions per config, logs to console, updates progress text. Ready for Task 4.
- **`onConfigChange()`**: Selector change handler that saves config and logs filtered question count
- **DOMContentLoaded**: Wires btnStart click to `switchPage('cover', 'quiz')` + `startQuiz()`, wires btnContinue, wires btnBack with confirm dialog, restores saved config to selectors, registers selector change events

### 4. Preserved existing code
All existing functionality remains intact:
- Cover page with canvas tree icon drawing
- PWA icon generation (`generateAppIcon`)
- `setupIcons()`, `setupManifest()`, `registerSW()`
- Saved progress display (lastProgress)

---

## Concerns
- None. All requirements implemented as specified.

---

## Test results
1. **Cover page**: Tree icon renders correctly, "开始刷题" button visible with pulse animation
2. **Page switch**: Clicking "开始刷题" transitions to quiz page with smooth opacity + translateY animation
3. **Quiz page shell**: Toolbar with back button, three selectors, empty main area, progress footer all present
4. **Back button**: Confirm dialog appears, returns to cover page on confirm
5. **Config persistence**: Selector values persist across page reloads via localStorage
6. **questions.json loading**: Console logs "[DS刷题] 已加载 511 道题目"
7. **Responsive**: At 768px toolbar title hides; at 480px toolbar compacts further
8. **Dark theme**: All components use existing CSS variables, dark background with proper contrast

---

## Task 3 Spec Compliance Fix Report

**Date:** 2026-07-03  
**Status:** DONE

### Issues Fixed

| # | Issue | Fix Applied |
|---|-------|-------------|
| 1 | Toolbar title was "配置面板" | Changed to "数据结构刷题" |
| 2 | Missing mode selector; had selType (single/fill) not in spec | Removed selType entirely; added selMode with learn/exam options |
| 3 | Chapter selector "all" value | Changed to value="0" |
| 4 | Count selector "all" value | Changed to value="0" |
| 5 | Extra "100题" option in count selector | Removed; also fixed spacing ("10 题" to "10题", etc.) |
| 6 | CSS class names didn't match spec | Renamed `.quiz-toolbar` to `.toolbar`, `.toolbar-selectors` to `.toolbar-controls` (both CSS and HTML) |
| 7 | getConfig() returned string "all" values from localStorage | Rewrote to read from DOM selectors and parseInt chapter/count |
| 8 | saveConfig/onConfigChange parameter mismatch | onConfigChange now calls getConfig() for consistency |
| 9 | Selector change handlers didn't call startQuiz() | Added startQuiz() call inside onConfigChange() |
| 10 | filterQuestions() filtered by config.type (removed) | Removed type filtering; only filters by config.chapter (0 = all) |
| 11 | switchPage() used setTimeout + class toggling | Replaced with requestAnimationFrame + inline style animation per spec |
| 12 | Back button confirm text | Changed to "确定要返回封面吗？当前进度将丢失。" |
| 13 | Header used class="quiz-toolbar" | Changed to class="toolbar" |
| 14 | Button title was "返回" | Changed to "返回封面"; also reordered attributes |
| 15 | progress-bar had inner progress-bar-fill div | Removed inner div; kept simple `<div class="progress-bar" id="progressBar">` |
| 16 | startQuiz() used string "all" comparisons | Updated to use number 0 for "all", removed config.type references |
| 17 | onConfigChange() referenced selType | Updated to reference selMode and call getConfig() |

### Additional Cleanup
- Removed unused `.progress-bar-fill` CSS rule (no longer needed without the inner div)
- Updated `DOMContentLoaded` config restoration to use localStorage directly (since getConfig() now reads DOM selectors which may not have default values at that point)
- Updated all `selType` references in `DOMContentLoaded` to `selMode`

### Remaining Concerns
- None. All 17 spec compliance issues have been addressed.

---

## Review Fix Report -- 2026-07-03

### Fix 1: switchPage inline styles removed

**Problem:** `switchPage()` was setting `display: flex`, `flexDirection: column`, `alignItems: center`, and `justifyContent: center` as inline styles on every target page. These overrode the CSS rules for `#quiz.page.active` which requires `align-items: stretch; justify-content: flex-start`. This caused the quiz main area to be vertically centered instead of sitting below the toolbar.

**Fix applied:**
- Removed the four inline layout style lines (`display: flex`, `flexDirection: column`, `alignItems: center`, `justifyContent: center`) from `switchPage()`.
- Moved `toPage.classList.add('active')` to before the `requestAnimationFrame` block, so the CSS `.page.active { display: flex; }` handles visibility instead of inline `display`.
- Removed the now-redundant `toPage.classList.add('active')` from the `setTimeout` callback.
- `switchPage()` now only sets `opacity`, `transform`, and `transition` as inline styles for the animation, leaving layout entirely to the CSS stylesheet.

### Fix 2: currentQuestions module-level variable

**Problem:** The filtered/shuffled `questions` array in `startQuiz()` was a local variable with no way for Task 4's question rendering code to access it.

**Fix applied:**
- Added `let currentQuestions = [];` declaration next to `let allQuestions = [];` at module scope.
- Added `currentQuestions = questions;` assignment in `startQuiz()` after the shuffle logic, so Task 4 and subsequent tasks can access the current question set via `currentQuestions`.

### Remaining Concerns
- None. Both review findings have been addressed.
