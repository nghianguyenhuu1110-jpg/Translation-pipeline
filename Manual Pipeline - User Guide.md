# PREP Subtitle Translation Pipeline
**Version: 3.2.0 — Robustness & Validation**

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
6. **Step 4 (Validation)** runs automatically to check for structural parity and drift

> The script is smart about stage — if a file already has a draft, reviewed, or validated marker, it skips straight to the right step.

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

# Run Step 4: Cross-check ALL finalized SRTs for structural parity
python3 run_batch.py --check --lang TW

# Run Step 4 and AUTO-REPAIR timestamp drifts
python3 run_batch.py --check --repair --lang TW
```

---

## The 4-Step Workflow

### Step 1 — Free Auto-Translate (Zero Cost)
Generates a raw baseline translation using the Google Translate free web API. Fast but imperfect.

- **Script:** `core_free_translator.py`
- **Output:** `outputs/TW/<file>/<prefix><file>_draft.json`

---

### Step 2 — AI Quality Review (Core Quality Step)
This is the human-in-the-loop step where an AI reviewer enforces glossary consistency and Taiwan-specific vocabulary.

#### 🌟 The "Gold Standard" for Quality
To assist students in recognizing exam terminology, follow the **Bilingual Parentheses Rule**:
- **Rule:** When a key academic or exam term is mentioned, include the English term in parentheses (e.g., "任務反應 (Task Response)").

**Precision Review Mode (Large Files):**
For files >50 segments, divide the `_draft.json` into chunks of ~30 segments and review them individually to prevent AI truncation.

---

### Step 3 — Build the Final SRT
Converts the reviewed JSON back into a standard `.srt` file.

- **Script:** `Step_3_Build_Final_SRT.py`
- **Output:** `outputs/TW/<file>/<prefix><file>.srt`

The build summary will warn you if any segments had a Google Translate failure during Step 1 (they will contain English as a fallback). Check the segment indices listed and fix them manually in the reviewed JSON before distributing.

---

### Step 4 — Cross-Check & Verification
Validates that the translated subtitle file has the exact same segment count as the source and that no timestamp drift has occurred.

- **Script:** `Step_4_Cross_Check.py` (called by `run_batch.py --check`)
- **Action:** Compares `inputs/source.srt` with `outputs/target.srt`.
- **Repair:** Use the `--repair` flag to automatically sync timestamps if minor shifts are detected.

> **Note:** A segment count mismatch between source and target is a hard failure — the script will exit immediately and the file will not be marked as verified. This must be corrected manually (by editing the reviewed JSON and rebuilding) before validation can pass.

---

## Pipeline Status Stages

| Icon | Stage | Meaning |
|------|-------|---------|
| ⬜ | Needs translate | No draft JSON yet — run Step 1 |
| 🟡 | Needs AI review | Draft exists — ask Antigravity to review |
| 🟠 | Needs build | Reviewed JSON ready — run Step 3 |
| 🛡️ | Needs verify | SRT is built but needs Step 4 cross-check |
| ✅ | Verified | Final `.srt` is checked and production-ready |

---

## File Structure

```
prep-subtitle-pipeline/
│
├── run_pipeline.py          ← End-to-end runner (pick files → go)
├── run_batch.py             ← Per-step batch runner (status/check/repair)
├── Step_1B_Free_Translate.py  ← Single-file translate wrapper
├── Step_3_Build_Final_SRT.py  ← Single-file SRT builder
├── Step_4_Cross_Check.py      ← Structural parity & drift checker
├── claude_review_prompt.txt ← Source-of-truth review rules for AI
│
├── inputs/                  ← Place your .srt files here
└── outputs/
    └── TW/
        └── L5-D3/
            ├── TW-L5-D3_draft.json      ← After Step 1
            ├── TW-L5-D3_reviewed.json   ← After Step 2
            ├── .validated               ← Marker for Step 4 ✅
            └── TW-L5-D3.srt            ← Final Output
```

---

## Tracking & Future Plans
- See [CHANGELOG.md](CHANGELOG.md) for version history.
- See [ROADMAP.md](ROADMAP.md) for upcoming features.
