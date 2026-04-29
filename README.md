# Document-to-Podcast Script Generator

A desktop application that turns a Word, PowerPoint or PDF document into a structured, deterministic Host/Expert podcast script — ready to be edited, reviewed and exported as Word, or fed into an audio synthesis pipeline.

The application was built as part of an enterprise consulting toolkit; the source code is not public. This repository documents what the product does and the engineering decisions behind it.

---

## What it does

The user drops a corporate document (capability deck, case study, methodology overview…) and the application:

1. **Extracts** the document's content, including text from native layers and OCR'd text from infographics and image-heavy PDFs/PPTX slides.
2. **Detects** the document's section structure: standard consulting headings such as *Industry Challenge*, *Key Benefits*, *How We Help*, *Results*, *Approach*, *Methodology*.
3. **Generates** a Host/Expert script deterministically:
   - Headings become Host questions
   - Bullets become Expert points
   - Each Expert turn names and explains every relevant point from the document
4. **Lets the user edit** turns directly in the UI — edit, delete, reorder or insert new turns.
5. **Exports** the final script as a Word `.docx` file, ready for review, publication or hand-off to an audio production pipeline.

The result is a script that *says everything the source document says*, in a conversational format, without inventing content the document does not support.

---

## Why it exists

LLM-only "summarize this deck as a podcast" approaches fail on the use case in three concrete ways:

- **They invent.** Asked to "make a podcast from this deck," the model fills gaps with plausible but unsourced claims. For client-facing content built on top of corporate methodology decks, that is a hard no.
- **They omit.** When the deck has nine bullets in a *Key Benefits* section, the model picks "the most important three." The user wants all nine, in order, named.
- **They flatten structure.** A corporate deck is structured for a reason — Challenge → Approach → Benefits → Results is a narrative the consultant wants preserved. A free-form summary erases it.

This application reverses the relationship: **Python decides *what* to say (parsed straight from the document); the LLM decides *how* to phrase each individual turn.** The model never selects, ranks or omits content — it only renders one bullet at a time as a natural Expert sentence and one heading at a time as a natural Host question.

---

## Functional capabilities

| Capability | What it covers |
|---|---|
| Source formats | `.docx`, `.pptx`, `.pdf` as content sources; `.txt` and `.md` as example scripts; web URLs (optional, off by default) |
| OCR | Optional OCR pass for infographic-heavy slides and scanned PDFs; the app falls back gracefully to text-only mode if OCR is not installed |
| Section detection | Recognizes standard consulting section labels and treats unknown headings as additional sections |
| Deterministic generation | Headings → Host questions, bullets → Expert points; the LLM never picks *which* points |
| Per-turn LLM rendering | Each turn is rendered in isolation, so a re-roll on one Expert sentence does not destabilize the rest |
| Anti-hallucination validation | Generated turns are validated against the source document's named points |
| In-browser segment editor | Edit, delete, reorder or insert turns before export |
| Export | Word `.docx`, script-only (no evidence appendix), ready to ship |

---

## Engineering choices that mattered

- **The LLM does not decide content.** Every "what to say" decision is parsed from the document by Python (headings, bullets, section order). The LLM only handles "how to phrase this single bullet." This is what makes the output trustworthy on regulated client decks.
- **OCR is optional, not required.** Many corporate environments will not allow installing Tesseract. The application detects its absence and runs in degraded mode (native-text only) with a visible warning, rather than refusing to start.
- **Per-turn isolation, not full-script generation.** Asking for "the whole script" in one call produces drift across turns and makes editing destructive. Per-turn calls allow the user to re-render a single Expert sentence without touching the rest.
- **Word export is script-only.** Earlier versions exported the script *plus* an evidence-tracing appendix that mapped each Expert sentence back to the source bullet. Users disabled that view; the current export ships the clean script only.
- **Plug-and-play distribution.** The application is shipped as a one-click installer that provisions Python, the virtual environment and dependencies, then launches the local server. The target user is a consultant, not a developer.
- **Local-only by design.** The application runs on `127.0.0.1`. Source documents (which are often confidential) never leave the user's machine except to call the LLM provider for rendering individual turns.

---

## Position in a broader pipeline

This application is the *script-authoring* stage of a larger document → script → audio pipeline. Its output `.docx` (Host/Expert tagged) is consumable directly by a downstream audio synthesis application that handles voice selection, multi-voice rendering, overlaps and background music. The two stages are deliberately decoupled: the user can review, redline and approve the script as a Word document before any audio is produced.

---

## Status

Active. v1.0 distributed as a self-contained installer. Section detection, deterministic generation, OCR fallback, in-browser editor and Word export are in production use.

---

## What this repository contains

This README. The code lives in a private corporate repository and is not redistributable.
