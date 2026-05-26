# TrackMD — Format Specification v0.1

> Part of the **LearnSpec** suite  
> Status: Draft — May 10, 2026

---

## Core Principle

TrackMD is the **learning path format** of the LearnSpec suite. It orchestrates LearnMD, QuizMD, and FlashMD files into an ordered pedagogical sequence, with path-level metadata, completion criteria, and progress checkpoints.

TrackMD is the **only format in the suite that can import all content formats**. It gives a corpus its coherence: it transforms a set of independent files into a path with a beginning, an end, and milestones.

A Level 0 TrackMD file is a **human-readable table of contents** in any standard Markdown reader.

| Principle | Description |
|---|---|
| **Markdown-first** | A `.track.md` file is valid Markdown readable in any editor |
| **File-native** | All structure lives in files — no database required |
| **Graceful degradation** | `##` sections + `!import` directives — readable as a syllabus everywhere |
| **AI-native** | Generatable and consumable by an LLM without specific tooling |
| **Orchestrator** | The only format in the suite that imports multiple content types |

---

## Format Levels

| Level | Mechanism | Purpose |
|---|---|---|
| 0 | `##` headings + `!import` directives | Minimal sequence, readable everywhere |
| 1 | YAML frontmatter | Metadata, completion criteria, objectives |
| 2 | Attributes on `!import` | Optional steps, per-quiz passing score |

Each level is a strict superset of the previous one.

---

## Level 0 — Minimal Sequence

A Level 0 TrackMD file is a table of contents: `##` sections grouping `!import` directives pointing to content files.

```markdown
# Learning Python

## The Basics

!import ./01-variables.learn.md
!import ./quiz-variables.quiz.md
!import ./flashcards-variables.flash.md

## Control Flow

!import ./02-conditions.learn.md
!import ./02-loops.learn.md
!import ./quiz-control.quiz.md

## Final Assessment

!import ./quiz-final.quiz.md
```

**Graceful degradation**: in a standard reader, this file displays as a Markdown document with headings and plain-text lines for the directives — perfectly readable as a syllabus.

### Conventions

| Syntax | Meaning |
|---|---|
| `# Title` | Track title |
| `## Section title` | Thematic grouping of steps |
| `!import ./file.learn.md` | Educational content step |
| `!import ./file.quiz.md` | Assessment step |
| `!import ./file.flash.md` | Flashcard review step |
| `!ref ./file.media.md` | Declare an additional MediaMD context (the collection's `stock.media.md` is implicit — see *MediaMD §Default Reference Convention*) |
| `!ref ./file.diagram.md` | Declare an additional DiagramMD context (the collection's `stock.diagram.md` is implicit — see *DiagramMD §Default Reference Convention*) |
| `!ref ./file.glossary.md` | Declare a GlossaryMD context |
| `!checkpoint id:slug` | Track-level progress checkpoint |

### Importable File Types

| Extension | Format | Role in the track |
|---|---|---|
| `.learn.md` | LearnMD | Lesson step |
| `.quiz.md` | QuizMD | Assessment step |
| `.flash.md` | FlashMD | Review step |

TrackMD does not directly import leaf formats (DiagramMD, MediaMD, GlossaryMD) — these are embedded in the content files it orchestrates.

---

## Level 1 — YAML Frontmatter

```yaml
---
title: "Learning Python"                      # optional — inferred from the first # H1
lang: en                                       # REQUIRED — BCP-47 code
description: "A complete beginner's path to Python: variables, control flow, data structures, functions, and a final project."
                                               # optional — short summary for catalogues and previews
estimated_time: 10h                            # optional — total estimated track duration
level: beginner                                # optional — beginner | intermediate | advanced
target_audience: "Beginners with no programming experience"  # optional
prerequisites:                                 # optional — informative list (v0.1)
  - "No prior experience required"
tags: [python, programming, beginner]          # optional
author: Jane Smith                             # optional
created: 2026-05-10                            # optional — ISO 8601
updated: 2026-05-10                            # optional — ISO 8601
license: CC-BY-4.0                             # optional — SPDX identifier or custom
spec_version: "0.1"                            # optional
completion:                                    # optional — completion criteria
  require_all_mandatory: true                  # all mandatory steps must be completed
  passing_score: 0.7                           # global minimum score to pass the track
on_completion:                                 # optional — rewards on completion
  badge: ./badge-python-beginner.badge.md      # TODO — BadgeMD not yet specified
  certificate: ./cert-python.cert.md           # TODO — CertMD not yet specified
---
```

### Field Reference

#### Universal LearnSpec Fields

| Field | Required | Description |
|---|---|---|
| `title` | No | Track title — inferred from `# H1` if absent |
| `lang` | **Yes** | BCP-47 code (`en`, `fr`, `en-US`…) |
| `description` | No | Short plain-text summary (typically 1–3 sentences) — used for catalogues, link previews, and `<meta name="description">`. Distinct from `title` and from `target_audience` (which describes *who* the track is for, not *what* it covers). |
| `tags` | No | Thematic tags |
| `author` | No | Track author |
| `created` | No | Creation date, ISO 8601 |
| `updated` | No | Last update date, ISO 8601 |
| `license` | No | SPDX identifier or `custom` |
| `spec_version` | No | Targeted spec version (`"0.1"`) |

#### TrackMD-Specific Fields

| Field | Required | Type | Description |
|---|---|---|---|
| `estimated_time` | No | string | Total estimated track duration (`5h`, `2h30`…). If absent, the player may aggregate `estimated_time` from imported files. |
| `level` | No | enum | `beginner`, `intermediate`, `advanced` |
| `target_audience` | No | string | Description of the intended audience |
| `prerequisites` | No | string[] | Informative prerequisites in v0.1 — not enforced, displayed to the learner |
| `completion.require_all_mandatory` | No | bool | All mandatory steps must be completed (default: `true`) |
| `completion.passing_score` | No | float | Global minimum score to pass the track (0.0–1.0) |
| `on_completion.badge` | No | path | Path to a BadgeMD file awarded on completion |
| `on_completion.certificate` | No | path | Path to a CertMD file awarded on completion |

---

## Level 2 — Attributes on `!import`

Optional attributes may be added on the `!import` line to modify the behaviour of a specific step.

### `optional:true`

Marks a step as optional. The learner can see the content, but it is not required to complete the track.

```markdown
!import ./deep-dive-decorators.learn.md optional:true
!import ./bonus-flashcards.flash.md optional:true
```

All steps are mandatory by default (`optional:false`).

### `passing_score`

Overrides the `passing_score` defined in the imported QuizMD frontmatter. Applicable only to `.quiz.md` files.

```markdown
!import ./quiz-module-1.quiz.md passing_score:0.6
!import ./quiz-final.quiz.md passing_score:0.85
```

If neither the TrackMD nor the QuizMD defines a `passing_score`, no score threshold is applied to that step.

### Combining Attributes

```markdown
!import ./advanced-quiz.quiz.md passing_score:0.75 optional:true
```

---

## Complete Example

````markdown
---
title: "Learning Python — Beginner Track"
lang: en
description: "An end-to-end beginner track covering Python from first variables to a working final project."
estimated_time: 8h
level: beginner
target_audience: "Anyone who wants to learn programming with no prior experience"
prerequisites:
  - "No technical prerequisites"
  - "A computer with a web browser"
tags: [python, programming, beginner]
author: Jane Smith
created: 2026-05-10
license: CC-BY-4.0
spec_version: "0.1"
completion:
  require_all_mandatory: true
  passing_score: 0.65
on_completion:
  badge: ./badge-python-beginner.badge.md
---

# Learning Python — Beginner Track

> [!objectives]
> By the end of this track, you will be able to: write simple Python scripts, use control
> structures, manipulate lists and dictionaries, and read and write files.

!ref ./glossary-python.glossary.md

## Section 1 — First Steps

!import ./00-introduction.learn.md
!import ./01-variables.learn.md
!import ./quiz-variables.quiz.md passing_score:0.6
!import ./flashcards-variables.flash.md optional:true

!checkpoint id:section-1-done label:"Section 1 — First Steps complete"

## Section 2 — Control Flow

!import ./02-conditions.learn.md
!import ./02-loops.learn.md
!import ./quiz-control.quiz.md passing_score:0.65
!import ./flashcards-control.flash.md optional:true

!checkpoint id:section-2-done label:"Section 2 — Control Flow complete"

## Section 3 — Data Structures

!import ./03-lists.learn.md
!import ./03-dictionaries.learn.md
!import ./quiz-structures.quiz.md passing_score:0.65

!checkpoint id:section-3-done label:"Section 3 — Data Structures complete"

## Going Further (Optional)

!import ./04-advanced-functions.learn.md optional:true
!import ./04-decorators.learn.md optional:true

## Final Assessment

!import ./quiz-final.quiz.md passing_score:0.7

!checkpoint id:track-complete label:"Track complete" type:exercise-complete
````

---

## `!checkpoint` at Track Level

The `!checkpoint` directive works the same way in TrackMD as in LearnMD, but at the section or track level rather than within a single module.

```markdown
!checkpoint id:section-1-done label:"Section 1 complete" type:milestone
!checkpoint id:track-complete label:"Track complete" type:exercise-complete
```

Track-level checkpoints are distinct from lesson-level checkpoints — a player may use them to display overall progress across the track.

---

## Interoperability

| Mechanism | Support |
|---|---|
| `!import ./file.learn.md` | ✅ |
| `!import ./file.quiz.md` | ✅ |
| `!import ./file.flash.md` | ✅ |
| `!import ./file.diagram.md` | ❌ — standalone diagrams are embedded in content files |
| `!ref ./file.media.md` | ✅ |
| `!ref ./file.glossary.md` | ✅ |
| Imported by another format | ❌ — TrackMD is a root format |

---

## Validation

### Lenient Mode (default)

| Condition | Level |
|---|---|
| `lang` absent from frontmatter | Warning |
| Title absent (no H1 and no `title`) | Warning |
| No `!import` steps in the file | Warning |
| `!import` pointing to a missing file | Warning |
| `!ref` pointing to a missing file | Warning |
| `passing_score` on a non-QuizMD `!import` | Warning |
| `completion.passing_score` outside [0.0, 1.0] | Error |
| `!checkpoint` missing required `id` attribute | Error |
| Duplicate `!checkpoint` `id` within the document | Error |

### Strict Mode (`--strict`)

All warnings are promoted to errors.

---

## Decisions Deferred to v0.2

| Feature | Reason for deferral |
|---|---|
| Conditional prerequisites between sections | Requires branching logic — out of scope for v0.1 |
| Adaptive branching | Significant player complexity — real-world usage feedback needed first |
| Per-step estimated time (as `!import` attribute) | Available in the frontmatter of imported files — the player can aggregate |

---

*Released under the MIT License. Copyright © 2024-present LearnSpec Contributors*
