# Document-to-Podcast Script Generator

A desktop application that turns a Word, PowerPoint or PDF document into a Host/Expert podcast script. The output is a Word file the user can review, edit, and either send out as written content or hand to a downstream audio synthesis tool.

The application is part of a private corporate toolkit and the source code is not published. This README explains what the tool does and the choices behind it.

## What it does

The user drops in a corporate document. Capability deck, case study, methodology overview, that kind of thing. The application reads it, finds the section structure, and writes a conversational script where every section becomes a Host question and every bullet inside that section becomes a point the Expert addresses by name.

Reading the document is the first thing that has to be reliable. Native text comes out of the file directly. Image-heavy slides and scanned PDFs go through OCR. If OCR is not installed on the machine, the application says so and runs in a degraded mode against native text only, instead of refusing to start. Many corporate environments will not let you install Tesseract; the tool needs to keep working in those.

Detecting structure is the second thing. The application recognises the standard consulting heading set (Industry Challenge, Key Benefits, How We Help, Results, Approach, Methodology, and so on) and treats anything else it finds as an additional section. Headings drive the Host's questions; bullets drive the Expert's answers.

Generating the script is the part where most of the design effort went. The model never decides what to say. Python parses the document, figures out the sections and the bullets, and feeds the model one bullet at a time with the instruction to phrase it as a single Expert sentence, and one heading at a time with the instruction to phrase it as a single Host question. The model handles wording; it never picks, ranks, or omits content.

Once the script is generated, the user sees it in the browser as a sequence of editable turns. Each turn can be edited, deleted, reordered, or replaced. Re-rolling a single Expert sentence does not touch the rest of the script, because every turn was generated in isolation in the first place.

Export is a Word file containing only the script. No appendix, no evidence trace, no source mapping; an earlier version shipped all of those and nobody used them.

## Why it works the way it does

The shortest way to describe the design is that Python decides what to say and the model decides how to phrase it. That inversion of the usual relationship is what made the tool usable for client-facing content.

Letting a language model summarise a corporate deck into a podcast script fails in three ways that are hard to fix with better prompting. The model invents, filling small gaps with plausible but unsourced claims. The model omits, picking three of nine bullets when the user wanted all nine in order. The model flattens, erasing the Challenge → Approach → Benefits → Results structure that the deck was built around. None of those failures happen when the model is never given the choice in the first place.

Per-turn isolation is the second piece. A "render the whole script" prompt makes the model drift across turns, which means editing one turn destabilises the rest. A "render this single sentence" prompt keeps every turn independent, which is what makes the in-browser editor safe to use.

Local-only execution is the third. The application runs on 127.0.0.1. The source documents, which are usually confidential, never leave the user's machine except for the individual rendering calls to the model provider. That is the only way this gets approved for use on real client material.

## Where it sits in the bigger picture

This tool authors the script. It does not produce audio. The Word file it exports (Host and Expert turns, tagged) is the input format expected by a separate audio synthesis application that handles voice selection, multi-voice rendering, overlaps, and background music. The two stages are kept apart on purpose: the user reviews and approves the script as a written document before any voice acting happens, which is also when the consulting team can have legal or the client sign off.

## Distribution

The application ships as a one-click installer. Double-click sets up Python, creates a virtual environment, installs the dependencies, and starts the local server. The user is a consultant, not a developer.

## Status

Version one is in production. Section detection, deterministic generation, the OCR fallback, the in-browser editor, and Word export are all shipped.

## About this repository

You will only find this README here. The code is not public.
