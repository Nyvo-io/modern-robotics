# Modern Robotics Course Instructions

This folder is a persistent, interactive textbook course for a beginner in robotics with prior reinforcement-learning experience. Treat the local textbook PDF as the primary source.

## Resume a Study Session

Before responding to a request to continue or teach this course, read these files in order:

1. `progress.md`
2. `sources.md`
3. `learning_protocol.md`
4. The active chapter folder under `notes/`, including its chapter note and matching `-qa.md` file
5. Relevant files under `attachments/chXX/`

Do not restart from Chapter 1, repeat closed material, or skip the active subsection without the learner asking. The project-level learning route is maintained outside this course package; do not copy it into these files.

## Required Skills

When available, use all three skills together:

- `textbook-study-companion`: restore state; maintain progress, paired note/QA files, attachments, and subsection status.
- `course-study-tutor`: keep claims source-grounded; explain terminology, figures, and formulas; write durable notes.
- `learn-anything`: teach adaptively with one active-recall or application question at a time.

If a named skill is unavailable, follow the equivalent workflow in `learning_protocol.md`; do not silently drop the behavior.

## Non-negotiable Records

- **Before any question, explanation, or note edit, cross-check** `progress.md`, the relevant source-PDF pages, the active chapter note, and its QA file. Confirm the textbook sequence and verify that the current item's concepts, figures, formulas, and QA entry are complete before advancing.
- `learning_protocol.md` is the single source of truth for the teaching loop and note rules.
- Mark only the active subsection with `（进行中）`; remove it when finished and never add `（已完成）`.
- Keep each chapter's note and QA record in the same `notes/chXX/` folder.
- QA records contain only tutor prompts, follow-ups, reference answers, teaching intent, and next steps. Never store the learner's original answer unless they explicitly request it.
- Store extracted textbook visuals in `attachments/chXX/`, with source attribution, and embed them by relative Markdown path.
- For specialized terminology, write the Chinese term with its English equivalent on first use, e.g. 笛卡尔积（Cartesian product） and 构型空间（configuration space, C-space）；keep the terminology consistent afterward.
