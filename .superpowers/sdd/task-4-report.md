# Task 4 Report -- 选择题答题引擎 + 题目卡片组件

**Status:** DONE
**Date:** 2026-07-04
**Base Commit:** a64fbb2

---

## What was implemented

### 1. Quiz engine - `startQuiz()` replaced
Replaced the stub with a full implementation:
- Reads config via `getConfig()`, filters with `filterQuestions()`, shuffles with `shuffleArray()`
- Respects `config.count` limit when > 0 and less than pool size
- Initializes `window.quizState` with all quiz state (questions, currentIndex, mode, answered, selectedAnswer, questionType, results, wrongIds)
- Calls `updateProgress()` and `renderCurrentQuestion()`

### 2. `renderCurrentQuestion()` - Question card rendering
For **single-choice** questions (type === 'single'):
- Card header with chapter badge (`第X章 章节名`), progress (`current / total`), type badge (单选)
- Question text in `.question-text`
- Options list with buttons: `<button class="option-btn" data-letter="X"><span class="option-letter">X</span><span class="option-text">text</span></button>`
- Empty feedback div (`#questionFeedback`), explanation div (`#questionExplanation`), hidden next button (`#btnNext`)

For **fill-in-blank** questions (type === 'fill'):
- Shows the question text and a placeholder message "填空题暂不支持"
- Sets `state.answered = true` and shows a "下一题" button immediately

### 3. `handleChoice(btn)` - Option selection handler
- Extracts `data-letter` from clicked button
- Sets `state.selectedAnswer` and `state.answered = true`
- Disables all option buttons and adds `.selected` class to chosen option
- In **learn mode**: immediately calls `submitAnswer()` for feedback
- In **exam mode**: shows the "下一题" button without revealing correct/incorrect

### 4. `submitAnswer(userAnswer)` - Answer checking
- Compares user answer with `q.answer`
- Pushes result object `{ id, userAnswer, correctAnswer, correct }` to `state.results`
- Tracks wrong answers in `state.wrongIds` Set
- Highlights correct option (`.correct` class) and incorrect selection (`.incorrect` class)
- Shows feedback message (correct/incorrect) and explanation if available
- Reveals the "下一题" button

### 5. `nextQuestion()` - Advance to next question
- **Exam mode**: silently records the current answer (without UI feedback) before advancing
- **Fill questions**: advances directly since they are pre-marked as answered
- Advances `state.currentIndex`, resets `answered`/`selectedAnswer`, calls `renderCurrentQuestion()`
- On last question, calls `showResult()`

### 6. `showResult()` - Result page stub
- Saves progress to localStorage via `saveProgress()`
- Renders basic result summary to `.result-container` (correct/total count, correct rate)
- Switches to `#result` page via `switchPage('quiz', 'result')`

### 7. `updateProgress()` - Footer progress bar
- Updates `#progressText` with current question index and total count
- Updates `#progressFill` width as percentage of questions completed
- Updates `#correctRate` with correct answer rate from `state.results`

### 8. Event delegation on `#quizMain`
- Click handler uses `e.target.closest()` to detect clicks on `.option-btn` and `.next-btn`
- Option clicks delegate to `handleChoice()` (only when `!state.answered`)
- Next button clicks delegate to `nextQuestion()`

### 9. Keyboard shortcuts
- **Unanswered**: A/B/C/D or 1/2/3/4 selects the corresponding option
- **Answered**: Enter, ArrowRight, or Space triggers `nextQuestion()`
- Only active when `#quiz` page is visible (`classList.contains('active')`)

### 10. CSS additions/fixes
- Fixed `.progress-bar` CSS to act as a track (gray background, full width)
- Added `.progress-fill` CSS for the colored fill indicator (width transition)
- Question card CSS (`.question-card`, card slide-in animation, `.card-header`, `.chapter-badge`, `.question-number`, `.type-badge`, `.question-text`)
- Option button CSS (`.options-list`, `.option-btn`, `.option-letter`, `.option-text`, states: hover, active, `.selected`, `.correct`, `.incorrect`, `:disabled`)
- Feedback CSS (`.feedback`, `.feedback-correct`, `.feedback-incorrect`, `.hidden`)
- Explanation CSS (`.explanation`, `.hidden`)
- Next button CSS (`.next-btn`, `.hidden`)
- Fill placeholder CSS (`.fill-placeholder`)
- Empty state CSS (`.empty-state`)

---

## Test results

1. **Start quiz**: Clicking "开始刷题" transitions to quiz page, first question card renders with cardSlideIn animation
2. **Learn mode**: Clicking an option immediately shows correct/incorrect feedback, explanation, and highlights correct answer
3. **Exam mode**: Clicking an option shows it as selected, "下一题" button appears, correct/wrong is not revealed until the result page
4. **Next question**: Clicking "下一题" advances to next question with smooth card animation
5. **Progress bar**: Footer shows current progress (N / total), fill bar percentage, correct rate
6. **Fill-in-blank**: Fill questions show placeholder message + question text + skip button
7. **Keyboard**: A/B/C/D keys select options; Enter/ArrowRight advances to next question
8. **Return to cover**: Back button shows confirm dialog, returns to cover page
9. **End of quiz**: Last question's "下一题" switches to result page with score summary

---

## Concerns

- None. All requirements implemented as specified.
