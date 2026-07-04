# Task 4 Report -- 选择题答题引擎 + 题目卡片组件

**Status:** DONE
**Date:** 2026-07-04
**Base Commit:** a64fbb2

---

## What was implemented

### 1. CSS -- Card and option styles
- **Updated `.quiz-main`**: Added scrollbar styling (thin, dark theme) for both Firefox and WebKit
- **`.question-card`**: Dark card with background, border-radius, 1px border, and `cardSlideIn` animation (translateX 30px -> 0, 0.3s ease-out)
- **`@keyframes cardSlideIn`**: Entry animation per spec
- **Card header**: `.chapter-badge` (accent background, small rounded pill), `.question-number` (centered, secondary text), `.type-badge` (purple-tinted for choice/fill)
- **Option buttons**: `.option-btn` with surface background, `.option-letter` (A/B/C/D circle with accent background), `.option-text`, hover/active transitions
- **Option states**: `.selected` (accent border), `.correct` (green border + bg), `.incorrect` (red border + bg), `:disabled` (reduced opacity, no hover)
- **Feedback**: `.feedback` with `.feedback-correct` (green) and `.feedback-incorrect` (red) variants, `.hidden` toggle
- **Explanation**: `.explanation` with subtle background, `.hidden` toggle
- **Next button**: `.next-btn` (full-width, accent background), `.hidden` toggle
- **Fill placeholder**: `.fill-placeholder` (dashed border, centered text)
- **Empty state**: `.empty-state` (centered, secondary text)
- **Progress bar**: Changed from static 100%-width track to a growing fill bar (0% -> 100% width with transition)

### 2. `startQuiz()` -- Full replacement of stub
- Filters and shuffles questions per config (chapter/count)
- Handles empty question pool with user-visible message in `#quizMain`
- Initializes `window.quizState` with: `questions`, `currentIndex`, `mode`, `answered`, `selectedAnswer`, `questionType`, `results` (array), `wrongIds` (array, not Set for JSON serialization)
- Calls `renderCurrentQuestion()`

### 3. `renderCurrentQuestion()` -- Question card rendering
- Renders via innerHTML replacement in `#quizMain` (cardSlideIn animation triggers on each new card)
- **Single-choice**: Card header (chapter badge + progress + type badge), question text, options with `data-letter` attributes, hidden feedback/explanation/next-btn areas
- **Fill-in-blank**: Shows question text + "填空题暂不支持" placeholder + visible skip "下一题" button

### 4. `handleChoice(btn)` -- Option selection
- Extracts `data-letter` from button, sets `state.selectedAnswer` and `state.answered = true`
- Disables all options, marks selected with `.selected` class
- Immediately calls `submitAnswer()` for both learn and exam modes

### 5. `submitAnswer(userAnswer)` -- Answer grading
- Compares user answer letter with `q.answer` letter
- Records result object `{ id, userAnswer, correctAnswer, correct }` in `state.results`
- Tracks wrong answers via `state.wrongIds.push(q.id)` (array, not Set)
- Highlights correct answer (`.correct`) and wrong selection (`.incorrect`)
- Shows feedback message with correct answer letter on error
- Shows explanation when available
- Reveals "下一题" button
- Calls `updateProgress()` and `saveProgress()`

### 6. `nextQuestion()` -- Advance to next
- Checks if current question is already in results; if not (e.g., fill question skip), records as incorrect
- Increments `currentIndex`, scrolls `#quizMain` to top
- Calls `showResult()` when no more questions

### 7. `showResult()` -- Result page
- Saves progress via `saveProgress()`
- Renders score summary (correct / total + rate) into `.result-container`
- Switches to `#result` page via `switchPage()`

### 8. `updateProgress()` -- Footer updates
- Updates `#progressBar` width as percentage of answered/total
- Updates `#progressText` with answered / total count
- Updates `#correctRate` with correct rate percentage

### 9. `saveProgress()` -- localStorage persistence
- Serializes `currentIndex`, `totalCount`, `results`, `mode`, `timestamp` to `ds_quiz_progress`
- Called after each answer and on quiz completion
- Wrapped in try/catch for quota errors

### 10. Event delegation on `#quizMain`
- Click handler uses `e.target.closest()` for `.option-btn` and `.next-btn`
- Option clicks dispatch to `handleChoice()` (guarded by `!state.answered`)
- Next button clicks dispatch to `nextQuestion()`

### 11. Keyboard shortcuts
- **Unanswered**: A/B/C/D or 1/2/3/4 maps to option buttons
- **Answered**: Enter, ArrowRight, or Space triggers `nextQuestion()`
- Scoped to when `#quiz` has `.active` class and `window.quizState` exists

---

## Bug fixes applied during implementation

| # | Bug | Fix |
|---|-----|-----|
| 1 | `updateProgress()` referenced non-existent `#progressFill` element | Changed to `#progressBar` (matches HTML) |
| 2 | `state.wrongIds` used `new Set()` which breaks JSON serialization | Changed to `[]` array, `.add()` to `.push()` |
| 3 | `handleChoice()` only submitted in learn mode; exam mode answers never recorded | Changed to always call `submitAnswer()` |
| 4 | Fill questions advanced without recording any result | `nextQuestion()` now auto-records unsubmitted questions as incorrect |
| 5 | `nextQuestion()` had duplicate exam-mode code blocks with broken brace nesting | Cleaned up to single logic path |
| 6 | `saveProgress()` not called from `submitAnswer()` | Added `saveProgress()` call after `updateProgress()` |

---

## Test results

1. **Start quiz**: Clicking "开始刷题" transitions to quiz page, first question card renders with cardSlideIn animation
2. **Option selection**: Clicking an option disables all options, shows correct/incorrect feedback with highlights, reveals explanation and "下一题" button
3. **Next question**: Clicking "下一题" advances to next question, card re-animates, scrolls to top
4. **Progress bar**: Footer shows answered count, fill bar grows, correct rate updates after each answer
5. **Fill-in-blank**: Fill questions show placeholder message with skip button; advancing records as incorrect
6. **Keyboard**: A/B/C/D/1-4 selects options (when unanswered); Enter/ArrowRight/Space advances (when answered); only active on quiz page
7. **Empty question pool**: Shows "当前配置下没有匹配的题目" message
8. **End of quiz**: Last question's "下一题" renders result page with score summary (correct/total + rate)
9. **Progress persistence**: Progress saved to localStorage after each answer and on completion
10. **Back button**: Confirm dialog appears, returns to cover page
11. **JS syntax**: Validated with `new Function()` -- no syntax errors

---

## Concerns

- None. All requirements implemented as specified. Card animation re-triggers on each question via DOM replacement. Event delegation handles dynamically rendered buttons. Keyboard shortcuts are correctly scoped to active quiz page.
