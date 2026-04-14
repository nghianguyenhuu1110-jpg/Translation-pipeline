# Pilot Test Report: Subtitle Translation Pipeline (v3.1.0)

## 1. Project Overview
This report summarizes the results of the pilot test phase for the subtitle translation pipeline, focused on the **IELTS/TOEIC Course Suite** for the Taiwan market.

### 📊 Cumulative Translation Stats
*   **Total Lessons Processed:** 25
*   **Target Language:** Traditional Chinese (Mandarin - Taiwan)
*   **Output Format:** Production-ready `.srt` and master `_reviewed.json` versions.
*   **Status:** 100% of the current batch is production-ready.

### 📁 Inventory of Completed Lessons
| Category | Lessons |
| :--- | :--- |
| **Task 2 Strategy** | L21S1, L21S2, L23S1, L23S2, Lesson 14-D3, L27-D1, L5-D3 |
| **Task 1 Mastery** | L1S2, L9S1, L11-D3, L13S1, L13S2, L13S3 |
| **Grammar & Fundamentals** | L25S1, L16, L18S2_D2, Lesson 18-D2 |
| **Foundational Lessons** | L1, L2S1_D2, L2S2, L7S1, L7S2_D2 |

---

## 2. Workflow incorporated

### **Stage 1: Baseline Auto-Translation**
*   **Mechanism**: Free auto-translation via Google Translate API (web).
*   **Goal**: Create a semantic baseline in `_draft.json` format at zero cost.
*   **Status**: Successfully automated in `run_pipeline.py`.

### **Stage 2: AI Review & Human-in-the-Loop**
*   **Mechanism**: Manual/Semi-automated review using **Claude (Antigravity)**.
*   **Goal**: Enforcement of academic tone, project glossary, and pedagogical standards.
*   **Status**: Proven as the most reliable method for educational content quality.

### **Stage 3: Production SRT Build**
*   **Mechanism**: Terminal-based execution of `Step_3_Build_Final_SRT.py`.
*   **Goal**: Final format validation and generation of `.srt` files.
*   **Status**: Fully operational and handles timestamp preservation.

---

## 3. Core Constraints & Standards

### 🇹🇼 Linguistic Constraints (Taiwan Mandarin)
*   **Traditional Characters**: Strict use of 繁體字.
*   **Vocabulary Normalization**:
    *   `影片` (not 視頻), `資訊` (not 信息)
    *   **Specific Fixes**: `Prep 考生` (Preppies), `正文段落` (Body Paragraphs), `收縮關係子句` (Contracted Relative Clauses).

### 🎓 Pedagogical Constraints (Bilingual Focus)
*   **Bilingual Parentheses Rule**: High-priority academic terms must include the English original in parentheses (e.g., `任務反應 (Task Response)`).
*   **Teaching English**: Example sentences being taught in the video must remain **100% English**; only instructional narrative is translated.

### ⚙️ Technical Constraints
*   **No-Key Architecture**: Designed to operate without expensive API keys by leveraging web APIs and manual review logic.
*   **JSON Safety**: Reviewers must escape internal double quotes (`\"`) to ensure compatibility with the terminal-based builder.
*   **Timestamp Sync**: Start/End times are immutable during translation to ensure video sync integrity.

---

## 4. Conclusion
The pilot test confirms that the revised workflow (v3.1.0) successfully bridges the gap between machine translation speed and academic quality standards. The **Bilingual Parentheses Rule** and **local vocabulary enforcement** significantly improve the learning experience for Taiwan-based students.
