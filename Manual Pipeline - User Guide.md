# PREP Subtitle Translation Pipeline
**Version: 3.1.0 — Refined Pedagogical Standards**

**Who this is for:** Anyone at Prep translating English educational video subtitle files into target languages (e.g., Taiwanese Mandarin).

This document outlines the official pipeline to automatically translate and professionally review educational subtitles with the highest possible accuracy and adherence to IELTS/TOEIC glossary terms.

---

## How to Run (Choose One)

### 🟢 Option A — End-to-End Runner (Recommended)
**`run_pipeline.py`** — handles everything in one go. You just pick files, then pause for the AI review, then it finishes automatically.

```bash
python3 run_pipeline.py
# or:
python3 run_pipeline.py --lang TW
python3 run_pipeline.py --folder "inputs/Writing Inter 2.6" --lang TW
```

**What happens:**
1. A numbered file picker appears — choose which files to process
2. Step 1 (free auto-translate) runs automatically
3. Script pauses and shows you the exact message to paste to **Antigravity** for review
4. Press Enter once Antigravity finishes writing the `_reviewed.json` files
5. Step 3 (build final SRT) runs automatically

> The script is smart about stage — if a file already has a draft or reviewed JSON, it skips straight to the right step.

---

### 🔵 Option B — Run Steps Individually

Use this when you want more control over each step.

```bash
# See all files and their current pipeline stage
python3 run_batch.py --status

# Translate all pending files (auto-batch of 10)
python3 run_batch.py --translate --lang TW

# Translate specific files you choose from a menu
python3 run_batch.py --translate --lang TW --pick

# Build final SRTs from all reviewed JSONs
python3 run_batch.py --build --lang TW

# Build only specific files you choose from a menu
python3 run_batch.py --build --lang TW --pick
```

---

## The 3-Step Workflow

### Step 1 — Free Auto-Translate (Zero Cost)
Generates a raw baseline translation using the Google Translate free web API. Fast but imperfect — glossary terms like "Band" and "Task 1" will be incorrectly translated and are corrected in Step 2.

- **Script:** `core_free_translator.py` (called automatically by `run_pipeline.py` or `run_batch.py`)
- **Output:** `outputs/TW/<file>/<prefix><file>_draft.json`

To run manually on a single file:
```bash
python3 Step_1B_Free_Translate.py --input "inputs/Writing Inter 2.6/L5-D3.srt" --lang TW
```

---

### Step 2 — AI Quality Review (Core Quality Step)
This is the human-in-the-loop step where an AI reviewer enforces:
- IELTS/TOEIC glossary consistency (Band, Task 1, Task Achievement, etc.)
- Taiwan vocabulary (資訊 not 信息, 影片 not 視頻)
- Natural phrasing and semantic accuracy
- Correction of untranslated segments

#### 🌟 The "Gold Standard" for Quality
To assist students in recognizing exam terminology, follow the **Bilingual Parentheses Rule**:
- **Rule:** When a key academic or exam term is mentioned, include the English term in parentheses after its first or primary mention.
- **Example:** "任務反應 (Task Response)", "連貫性與銜接性 (Coherence and Cohesion)".
- **Glossary Focus:** Ensure terms like `Body Paragraphs` (正文段落), `Topic Sentence` (主題句), and `Preppies` (Prep 考生) are correctly localized for Taiwan.

**Option A: Antigravity (Claude Cowork) — Fastest**
Simply tell Antigravity in this chat:
> *"Review `outputs/TW/L5-D3/TW-L5-D3_draft.json` for TW subtitles"*

Antigravity reads the draft, applies all rules from `claude_review_prompt.txt`, and saves the `_reviewed.json` directly to your disk. No browser needed.

**Option B: Google AI Studio or Claude.ai**
1. Open **Google AI Studio** or **Claude.ai**
2. Set the System Prompt to the contents of `claude_review_prompt.txt`
3. Upload the `_draft.json` file
4. Prompt: *"Please review and finalize the attached draft JSON for TW subtitles."*
5. Save the output as `_reviewed.json` in the same output folder

---

### Step 3 — Build the Final SRT
Converts the reviewed JSON back into a standard `.srt` file ready for video embedding.

- **Script:** `Step_3_Build_Final_SRT.py`
- **Output:** `outputs/TW/<file>/<prefix><file>.srt`

To run manually on a single file:
```bash
python3 Step_3_Build_Final_SRT.py --input "outputs/TW/L5-D3/TW-L5-D3_reviewed.json"
```

---

## Pipeline Status Stages

| Icon | Stage | Meaning |
|------|-------|---------|
| ⬜ | Needs translate | No draft JSON yet — run Step 1 |
| 🟡 | Needs AI review | Draft exists — ask Antigravity to review |
| 🟠 | Needs build | Reviewed JSON ready — run Step 3 |
| ✅ | Complete | Final `.srt` is done |

---

## File Structure

```
prep-subtitle-pipeline/
│
├── run_pipeline.py          ← End-to-end runner (pick files → go)
├── run_batch.py             ← Per-step batch runner with --pick flag
├── Step_1B_Free_Translate.py  ← Single-file translate wrapper
├── Step_3_Build_Final_SRT.py  ← Single-file SRT builder
├── core_free_translator.py  ← Translation engine (do not call directly)
├── claude_review_prompt.txt ← Source-of-truth review rules for AI
│
├── inputs/                  ← Place your .srt files here
│   └── Writing Inter 2.6/
│
└── outputs/
    └── TW/
        └── L5-D3/
            ├── TW-L5-D3_draft.json      ← After Step 1
            ├── TW-L5-D3_reviewed.json   ← After Step 2
            └── TW-L5-D3.srt            ← After Step 3 ✅
```

---

## Tracking & Future Plans
- See [CHANGELOG.md](CHANGELOG.md) for version history.
- See [ROADMAP.md](ROADMAP.md) for upcoming features.
