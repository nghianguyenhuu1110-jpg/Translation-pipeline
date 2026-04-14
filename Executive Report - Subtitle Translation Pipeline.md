# Executive Report: Subtitle Translation Pipeline
**Version 3.2.0 — Pilot Results & Scaling Strategy**
**Date: April 2026**

---

## Executive Summary

Prep has successfully built and piloted an automated subtitle translation pipeline for its IELTS/TOEIC video course library. The system translates English subtitle files into target languages, applies AI-assisted quality review, and validates structural integrity before delivery.

The pilot covered **22 lessons** translated into Traditional Chinese (Taiwan), all of which passed automated structural validation with zero timestamp drift. The pipeline is operational and producing output of sufficient quality to move toward production — provided a final human review round is included before publishing.

**The recommended next step is to build this into an internal tool** accessible to the broader content team, with a structured human review stage as the quality gate before any subtitle file goes live.

---

## 1. What the Pipeline Does

The pipeline converts English `.srt` subtitle files into translated, production-ready subtitle files through four automated stages:

| Stage | What Happens | Human Involvement |
|---|---|---|
| **1 — Auto-Translate** | Google Translate generates a raw baseline translation | None |
| **2 — AI Review** | An AI reviewer (currently Claude) checks every segment against glossary rules, pedagogical standards, and linguistic guidelines. An Google AI Studio/Gemini runs another round of review | Triggers the review; pastes the output back |
| **3 — Build SRT** | Reviewed corrections are compiled into a final `.srt` file | None |
| **4 — Validation** | Automated check confirms segment count and timestamps match the source exactly | None |

The process is designed to be fast, consistent, and scalable. A single operator can process an entire course suite in a working day.

---

## 2. Pilot Results

| Metric | Result |
|---|---|
| Lessons processed | 22 |
| Target language | Traditional Chinese (Taiwan) |
| Structural validation pass rate | 100% (22/22) |
| Timestamp drift | 0 files |
| Segment recovery (large files) | 100% (e.g. L21S1 restored from 127 → 179 segments) |

The pipeline has demonstrated reliability under real course content, including complex large-format lessons that previously caused data loss with earlier automation approaches.

---

## 3. Output Quality & Production Readiness

### What the pipeline guarantees
- **Structural integrity**: Every output file has the exact same segment count and timestamps as the source English file. Subtitles will never be out of sync with video.
- **Glossary compliance**: Key IELTS/TOEIC terms, brand names, and pedagogical vocabulary are handled according to defined rules.
- **Consistency**: Every file is processed against the same ruleset, eliminating the variance of manual translation.

### What the pipeline does not guarantee
- **Nuance and flow**: Machine translation produces accurate but sometimes unnatural phrasing. AI review significantly improves this, but it is not equivalent to a trained human translator.
- **Cultural appropriateness**: Idiomatic expressions, tone, and register require human judgement.
- **Domain expertise**: A reviewer familiar with IELTS/TOEIC pedagogy will catch errors the AI may not flag.

### Recommendation: Two-Round Quality Model

> **It is strongly recommended that no pipeline output goes directly to production without a final human review.**

The proposed quality model is:

```
Auto-Translate → AI Review → [Pipeline Output] → Human Review → Production
```

The pipeline output should be treated as a **high-quality first draft**, not a final deliverable. A qualified reviewer — ideally a bilingual educator familiar with the exam content — should perform a final pass before any subtitles are published to students.

This is consistent with industry standards for AI-assisted translation workflows and significantly reduces per-file review time compared to translating from scratch.

**Estimated time savings with pipeline vs. fully manual translation: 70–80% reduction in production time per lesson.**

---

## 4. Scaling to an Internal Tool

The current pipeline runs locally on one machine. To deploy it as a shared internal tool accessible to the content team, the following investments are required.

### Core Requirements

**Infrastructure**
- A server or cloud environment to run translation jobs (not tied to one person's laptop)
- Cloud file storage (e.g. Google Cloud Storage or AWS S3) to replace local input/output folders
- A database to track job status, file history, and user activity

**User Interface**
- A web-based dashboard so non-technical staff can upload files, monitor progress, and download outputs without using the command line
- A built-in review interface where human reviewers can read, edit, and approve segments directly in the browser

**Authentication & Access Control**
- User accounts with defined roles: Administrator, Reviewer, Viewer
- Audit trail: who approved what, and when

**Translation Service**
- The current free Google Translate integration will not scale under concurrent use
- A paid translation API (Google Cloud Translation, DeepL, or equivalent) is required for production reliability

**AI Review Integration**
- Currently, AI review requires a human to manually paste a prompt into a chat interface
- For an internal tool, this step would be called automatically via API, removing the manual handoff entirely
- This is the single most significant architectural change

### Options for the AI Review Step

| Approach | Description | Trade-off |
|---|---|---|
| **Fully automated** | Claude API called automatically; no human in the loop for AI review | Faster, lower cost per file; less oversight |
| **Human-in-the-loop** | Reviewer triggers AI review and approves output before proceeding | Higher quality control; adds time |
| **Hybrid (recommended)** | AI review runs automatically; human reviewer approves before production | Balances speed and quality |

The hybrid model aligns with the two-round quality model described above and is the recommended approach.

---

## 5. Resource Requirements

### To build the internal tool

| Area | Scope |
|---|---|
| Backend development | Pipeline steps refactored into a web service; job queue for async processing |
| Frontend development | Web UI: file upload, status dashboard, review and approval interface |
| Infrastructure setup | Cloud hosting, file storage, database, CI/CD pipeline |
| API integration | Claude API for automated AI review; paid translation API |
| QA & testing | End-to-end testing with real course content |

**Estimated scope:** 6–10 weeks for a small development team (2–3 engineers) to deliver a functional internal tool covering the core workflow.

### Ongoing operational costs (estimated)

| Service | Basis |
|---|---|
| Translation API | Per character translated (volume-dependent) |
| Claude API (AI review) | Per token reviewed (volume-dependent) |
| Cloud hosting & storage | Per month, based on usage |

For a team processing one course suite per month, API costs are expected to be modest (low hundreds of dollars). Exact figures depend on volume and chosen providers.

---

## 6. Risks & Mitigations

| Risk | Likelihood | Mitigation |
|---|---|---|
| AI review misses a terminology error | Medium | Final human review round before production |
| Translation API rate limits or downtime | Low–Medium | Retry logic in pipeline; fallback provider |
| Segment count mismatch in output | Low | Step 4 validation fails hard; file cannot be marked production-ready until resolved |
| Staff unfamiliar with review process | Medium | Reviewer training guide; in-app review guidelines |
| Sensitive course content exposed to third-party APIs | Low–Medium | Review data retention policies of translation and AI providers; consider on-premise options for sensitive content |

---

## 7. Recommended Next Steps

1. **Adopt the two-round quality model now** — treat all current pipeline output as first drafts pending human review before publishing
2. **Define the reviewer role** — identify who in the content team will own the final review step and establish a sign-off process
3. **Decide on AI review approach** — automated, human-in-the-loop, or hybrid (recommendation: hybrid)
4. **Commission a technical scoping exercise** — a short discovery sprint to produce detailed specifications and a delivery timeline for the internal tool
5. **Evaluate translation API providers** — compare Google Cloud Translation, DeepL, and others on cost, quality, and data privacy terms

---

## Appendix: Current Technology Stack

| Component | Technology |
|---|---|
| Translation | Google Translate (free web API — pilot only) |
| AI Review | Claude (manual chat interface — pilot only) |
| Validation | Custom Python (`Step_4_Cross_Check.py`) |
| Supported languages | Traditional Chinese (TW), Korean (KR), Japanese (JP), Simplified Chinese (CN), Bahasa Indonesia (ID) |
| Operating environment | Local machine (macOS) |
| Pipeline version | 3.2.0 |
