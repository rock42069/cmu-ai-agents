---
name: ingest-learning-source
description: Ingest a paper, article, essay, technical post, or other learning reference into the user's persistent Obsidian learning wiki. Use when the user explicitly sends a source link or source file to add, save, ingest, remember, summarize, brief, or incorporate into their course or learning vault. Store the source in the raw layer, write a source-grounded summary with key ideas, and update the wiki index and log. Do not trigger for a URL merely mentioned for navigation, troubleshooting, or casual discussion when the user has not indicated it belongs in the knowledge base.
---

# Ingest Learning Source

Build durable understanding from one user-selected source while preserving a
clear trail back to the original.

## Locate the vault

Use the current workspace when it contains `AGENTS.md`, `raw/`, and `wiki/`.
Read `AGENTS.md` completely and follow its local conventions. If no unique vault
can be identified safely, ask the user for its location before writing.

## Ingest the source

1. Inspect the URL or file and identify the canonical title, author or
   organization, publication date when available, source type, and canonical
   URL. For a paper, prefer its publisher, conference, arXiv, or author-hosted
   primary page over third-party summaries.
2. Read the complete source when accessible. For a PDF whose figures or layout
   carry meaning, inspect the relevant rendered pages as well as extracted text.
   State any access or completeness limitation; never imply a full reading when
   only an abstract or excerpt was available.
3. Preserve the raw layer:
   - Save an openly downloadable paper PDF or user-provided file unchanged in
     `raw/`, using a stable descriptive filename.
   - For a web article, paywalled source, or source that should not be
     redistributed, create an immutable Markdown source record in `raw/` with
     title, author, publication date, canonical URL, access date, and access
     limitations. Do not copy an entire copyrighted article into the vault.
   - Never overwrite or edit an existing raw artifact. Reuse it when identical;
     otherwise add a distinct version.
4. Search `wiki/index.md` and existing wiki pages before creating anything.
   Update an existing source page when it represents the same work; otherwise
   create a descriptively named page under `wiki/sources/`.
5. Write the source page using only sections that add value:

   ```markdown
   # Source title

   ## Brief
   One compact paragraph explaining what this is and why it matters.

   ## Summary
   A faithful explanation of the source's argument, method, evidence, and
   conclusions at a depth appropriate to the material.

   ## Key ideas to keep in mind
   - Durable ideas stated clearly and with necessary qualifications.

   ## Connections
   Links to genuinely related [[wiki pages]]; omit when none exist yet.

   ## Questions and caveats
   Limitations, uncertainties, objections, or useful follow-up questions.

   ## Source
   Relative link to the raw artifact or record plus the canonical web URL.
   ```

6. Separate the source's claims from inference or commentary. Preserve important
   nuance, assumptions, limitations, and negative results. Do not inflate a
   preliminary result into an established fact.
7. Integrate durable concepts into existing wiki pages only when the source
   materially improves them. Create a new concept page only when it is already a
   useful navigation or understanding target; avoid noun-page sprawl.
8. Add or update the source entry in `wiki/index.md` with a wikilink and one-line
   description. Keep the existing organization unless the user approves a
   consequential restructuring.
9. Append `## [YYYY-MM-DD] ingest | Source title` to `wiki/log.md`, listing raw
   and wiki files created or materially updated. Never rewrite older entries.
10. Check links and the Git diff. Report what was added, the principal takeaway,
    and any access limitation. Commit or push only when the user requests it or
    the vault has an explicit standing instruction to do so.

## Handle ambiguity

When the user sends several links, do not silently assume whether to batch them.
If their wording clearly requests all of them, ingest each source separately and
then connect them. Otherwise ask which source to start with.

If the source is long, still prioritize fidelity over speed. Ask the user about
desired emphasis only when different choices would materially change the saved
synthesis; otherwise complete the ingest autonomously.
