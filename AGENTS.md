# Wiki maintainer instructions

This repository is an Obsidian vault and a persistent, LLM-maintained learning
wiki. The human curates sources, directs exploration, and decides what matters.
The agent handles organization, synthesis, cross-linking, and maintenance.

## Boundaries

- Do not crawl, import, or pre-structure the course unless explicitly asked.
- Do not create assignment content or solutions unless explicitly asked.
- Ask before making a domain-level organizational choice that is not supported
  by material already present in the vault.
- Never modify files under `raw/` during ingestion. They are source-of-truth
  artifacts. Moving an inbox item into `raw/` is allowed only when requested or
  clearly approved.
- Treat the wiki as an evolving synthesis, not a collection of isolated
  summaries.
- Preserve the user's writing and existing edits.

## Layers

- `inbox/`: unprocessed thoughts, links, snippets, and temporary notes.
- `raw/`: immutable original sources and `raw/assets/` attachments.
- `wiki/`: agent-maintained pages derived from sources and conversations.
- `wiki/index.md`: content-oriented catalog of every maintained wiki page.
- `wiki/log.md`: append-only chronological record of meaningful operations.

## Ingest workflow

When the user asks to ingest a source:

1. Read the selected raw source completely enough to represent it faithfully.
2. Discuss ambiguity or emphasis with the user when it would materially affect
   the result.
3. Create or update the appropriate pages in `wiki/`.
4. Integrate new information into existing concept pages; add useful Obsidian
   `[[wikilinks]]` in both directions where appropriate.
5. Distinguish sourced claims from the agent's synthesis. Record disagreement,
   uncertainty, and superseded claims rather than flattening them.
6. Update `wiki/index.md`.
7. Append an ingest entry to `wiki/log.md` using the documented format.

Do not mechanically create a page for every noun. Create a concept page when it
is useful as a durable destination for understanding or navigation.

## Query workflow

Read `wiki/index.md` first, then the relevant pages and raw sources. Answer with
traceable source links. Only file an answer back into the wiki when the user
asks, or when they clearly agree it is durable knowledge worth keeping. Log
filed queries.

## Page conventions

- Use Markdown and Obsidian `[[wikilinks]]`.
- Prefer clear, descriptive filenames and one durable subject per page.
- Keep prose concise, but preserve nuance and unresolved questions.
- Link claims to their raw source using a relative Markdown link when possible.
- Use YAML frontmatter only when it has an immediate use; do not add metadata
  merely because it might become useful later.
- Avoid duplicate pages. Search the index and filenames before creating one.

## Log format

Use one of these parseable headings:

`## [YYYY-MM-DD] ingest | Title`

`## [YYYY-MM-DD] query | Title`

`## [YYYY-MM-DD] lint | Scope`

Under the heading, briefly record files created or materially updated and why.
Never rewrite older log entries except to correct a factual error.

## Maintenance

When asked to lint, check for broken links, orphaned pages, duplicated concepts,
contradictions, stale synthesis, missing source traceability, and gaps worth
investigating. Propose consequential reorganizations before applying them.

