# Executive Report: Subtitle Translation Pipeline
**Version 3.2.0 — CEO Approval Brief**
**Date: April 2026**

---

## Executive Summary

Prep has built and successfully piloted an automated subtitle translation pipeline. In the pilot, **22 lessons** were translated into Traditional Chinese (Taiwan), all passing structural validation with zero errors.

The pipeline is producing output of professional quality, but currently relies on **free tools and a manual AI review process** — creating inconsistency and reliability risks that are not acceptable for production content at scale.

This report proposes a **phased approach**:

- **Phase 1 (immediate ask):** A small investment to fix the two critical reliability risks, making output consistent and production-ready every time.
- **Phase 2 (future, separate approval):** A full internal web tool for the broader content team.

The Phase 1 quality gate is handled by **external proofreaders** — not additional software — keeping the investment minimal while maintaining a high standard before any subtitle reaches a student.

---

## 1. Current State & What's Working

The pipeline translates English `.srt` subtitle files into target languages through four stages:

| Stage | What Happens |
|---|---|
| **1 — Auto-Translate** | Generates a raw baseline translation automatically |
| **2 — AI Review** | AI checks every segment against glossary, pedagogical, and language rules |
| **3 — Build** | Reviewed corrections compiled into a final `.srt` subtitle file |
| **4 — Validate** | Automated check confirms the output matches the source structure exactly |

**Pilot results: 22 lessons translated, 100% structural validation pass rate, zero timestamp drift.**

The core logic is proven and the quality of reviewed output is strong. The problem is not the pipeline itself — it is the tools it currently depends on.

---

## 2. The Two Problems That Need to Be Fixed

### Problem 1 — Token Exhaustion (Reliability Risk)
The AI review step is currently done by pasting content into a chat window. Chat windows have a finite context limit. As a session accumulates history, available tokens shrink — large files can be silently truncated mid-review, producing incomplete output with no warning.

**Impact:** We cannot guarantee a complete, reliable review for every file under the current approach.

### Problem 2 — No Pinned Model (Consistency Risk)
The review step has been run across multiple AI interfaces — Claude, Antigravity, and Google AI Studio (Gemini). Different models interpret the review rules differently and produce different output quality, particularly for CJK languages.

**Impact:** Two files reviewed in different sessions may not meet the same quality standard, creating inconsistency across the subtitle library.

### Root cause of both problems
Both issues stem from the same source: the review step runs through a manual chat interface rather than a controlled API call. Fixing this resolves both problems simultaneously.

---

## 3. Phase 1 — The Minimal Investment Ask

### What changes
Replace the two free/manual dependencies with reliable, controlled alternatives:

| Current (Pilot) | Phase 1 |
|---|---|
| Google Translate free web scraper | Google Cloud Translation API (paid, reliable) |
| Manual AI review via chat window | Claude API with pinned model (`claude-opus-4-6`) |

### What stays the same
- The pipeline scripts, validation logic, and file structure are unchanged
- The workflow remains the same — translate, review, build, validate
- The team operates it the same way

### What this delivers
- **Consistent output every time** — same model, same rules, every file
- **No token exhaustion** — each API call is a fresh context; file size is no longer a constraint
- **Audit trail** — every review is logged with the model version and timestamp
- **No manual copy-paste** — the review step runs automatically as part of the pipeline

### Engineering effort
A legacy automated review script (`Step_2_Claude_Review_API.py`) already exists from an earlier version of the pipeline. Phase 1 is primarily updating and integrating that script — **estimated 2–3 days of developer time**, not a new build.

---

## 4. Quality Assurance — External Proofreaders

Automating the AI review does not remove human judgement from the process. The proposed quality model is:

```
Auto-Translate → AI Review (API, pinned) → External Proofreader → Final Product
```

**The pipeline produces a professional first draft.** An external proofreader — a bilingual educator or professional translator familiar with IELTS/TOEIC content — validates the output before it is published to students.

This is industry-standard practice for AI-assisted translation workflows and provides a clear quality gate without requiring Prep to build or staff an in-house review function at this stage.

**Estimated time savings versus fully manual translation: 70–80% reduction per lesson.** The proofreader's role shifts from translating to validating — a significantly faster and lower-cost engagement.

---

## 5. Phase 1 Investment Summary

| Item | Type | Estimated Cost |
|---|---|---|
| Google Cloud Translation API | Ongoing (per character) | Low — volume-dependent |
| Claude API (`claude-opus-4-6`) | Ongoing (per token reviewed) | Low — volume-dependent |
| Developer time to integrate | One-time | ~2–3 days |
| External proofreader (per course suite) | Per engagement | Market rate |

For a team processing one course suite per month, API costs are expected to be in the range of **low hundreds of dollars per month**, depending on volume and providers. Exact figures can be confirmed with a usage estimate before commitment.

There is no infrastructure build, no server, and no new software product required for Phase 1.

---

## 6. Phase 2 — Full Internal Tool (Future)

Once Phase 1 is proven and output quality is validated by proofreaders, the natural next step is a shared internal tool that allows the broader content team to run the pipeline without technical knowledge.

This would include a web interface, user accounts, cloud file storage, and a built-in review and approval workflow. This is a **separate approval and a separate conversation** — Phase 2 does not need to be committed to now.

Phase 1 produces the consistent, reliable foundation that Phase 2 would be built on.

---

## 7. Risk Assessment

| Risk | Likelihood | Mitigation |
|---|---|---|
| API translation quality lower than expected | Low | API-grade Google Translate is equivalent to or better than the free version; proofreader catches any issues |
| Claude API review misses an error | Low–Medium | External proofreader is the quality gate before production |
| API costs exceed estimates | Low | Usage can be monitored and capped; pilot data provides a reliable baseline |
| Proofreader availability or quality | Medium | Standard vendor management; brief against the same review rules used by the AI |
| Phase 1 scope expands unexpectedly | Low | Legacy review script already exists; work is integration, not new development |

---

## 8. Recommendation & Next Steps

**Recommendation:** Approve Phase 1. The investment is small, the engineering work is mostly already done, and it directly resolves the two High-rated risks identified in the current workflow.

**Proposed next steps:**

1. **Approve Phase 1 budget** — API costs + 2–3 days developer time
2. **Identify an external proofreader** — brief them against the existing review guidelines (`claude_review_prompt.txt`)
3. **Integrate the Claude API review step** — update and wire up the existing legacy script
4. **Run a Phase 1 pilot** — process one full course suite end-to-end and submit to proofreader for validation
5. **Review Phase 2** — once Phase 1 output is confirmed production-ready, revisit the internal tool conversation

---

## Appendix: Current Technology Stack

| Component | Technology | Status |
|---|---|---|
| Translation | Google Translate (free web API) | **To be replaced — Phase 1** |
| AI Review | Manual chat (model unpinned; quality varies by session) | **To be replaced — Phase 1** |
| AI Review (target) | Claude API, model `claude-opus-4-6` (pinned) | Phase 1 |
| Validation | Custom Python (`Step_4_Cross_Check.py`) | Production-ready |
| Supported languages | Traditional Chinese (TW), Korean (KR), Japanese (JP), Simplified Chinese (CN), Bahasa Indonesia (ID) | Production-ready |
| Operating environment | Local machine — single operator | Phase 1 unchanged |
| Pipeline version | 3.2.0 | — |
