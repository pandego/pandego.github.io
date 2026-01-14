---
name: portfolio-project-creator
description: Create new numbered portfolio project/case study entries for this MkDocs site. Use when asked to draft a new portfolio item from notes, transcripts, or draft markdown while following `docs/portfolio/projects/project_template.md` and `docs/portfolio/projects/project_voice.md`, and saving the result as `docs/portfolio/projects/project-N-short-slug.md`.
---

# Portfolio Project Creator

## Overview

Create a new portfolio project file from messy inputs, using the existing template and voice guidelines.

## Inputs to Gather

- Source material: transcript, notes, or draft markdown.
- Optional: architecture diagram filename in `docs/assets/`.
- Optional: metrics, tech stack, client, industry, and role.

If critical fields are missing (title, impact description, or metrics), ask targeted follow-up questions before writing.

## Workflow

1. Read the template: `docs/portfolio/projects/project_template.md`.
2. Read the voice guide: `docs/portfolio/projects/project_voice.md`.
3. Extract structured data from the input: title, one-line description, overview paragraph, challenge, solution details, implementation, architecture caption, tech stack, key learnings, and measurable impact.
4. Choose the next filename:
   - List `docs/portfolio/projects/project-*.md`.
   - Ignore `project_template.md` and `project_voice.md`.
   - Parse the numeric portion after `project-` and set the next number to max + 1 (use 1 if none).
   - Create a 2–3 word slug from the project theme or client context; convert to kebab-case; remove stopwords; avoid duplicates.
   - Final filename format: `project-<num>-<slug>.md`.
5. Draft the markdown by filling the template sections, removing the `??? tip` block, and aligning to the voice guide.
6. Validate formatting:
   - Single H1 matching the frontmatter title.
   - Sentence case headings.
   - `!!! abstract` block with Client/Industry/Role and Impact Metrics (two spaces for line breaks after Client/Industry/Role).
   - Tech stack and metrics as hyphen lists.
   - `<img ... width="600">` for architecture diagram plus italic caption.
   - Close with the CTA card block.
7. Write the new file into `docs/portfolio/projects/`.
8. Update the portfolio index:
   - Add the new card entry at the top of the `<div class="grid cards" markdown>` list (most recent first).
   - Follow this card format:

     ```markdown
     -   [Project Title](projects/project-N-short-slug.md)

         ---

         One-line summary sentence.
     ```

9. Update the navigation in `mkdocs.yml`:
   - Add the new project to `nav > Case Studies > Projects`.
   - Keep the list in most recent-first order.

## Notes on Incomplete Inputs

- Prefer asking questions over inventing numbers or claims.
- If the client name is unknown, use “Confidential”.
- If a diagram is not available, leave the diagram filename out and ask for it before finalizing.

## Output Checklist

- Filename follows `project-<num>-<2-3-words>.md`.
- Portfolio index includes a new card entry.
- `mkdocs.yml` includes the new project in nav.
- No template placeholders remain.
- Voice matches first-person, impact-focused tone.
- Metrics are concrete and consistent with the input.
