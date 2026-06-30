# Knowledge Base Agent Instructions

This repository is a knowledge SSOT for reusable engineering principles,
architecture decisions, and implementation guidance.

## Operating Rule

When using this repository as context:

1. Read `indexes/routing-index.md` before reading individual SSOT documents.
2. Select only the documents relevant to the current task.
3. Prefer documents with `status: accepted` over `draft` or `deprecated`.
4. Prefer project-local documentation when it conflicts with this repository.
5. If a conflict matters, report the conflict with both file paths.
6. Cite the selected document paths in answers, reviews, and implementation notes.

## Source Priority

Use this priority order:

1. User instruction in the active conversation.
2. Project-local `AGENTS.md`, specs, ADRs, and implementation docs.
3. This repository's accepted SSOT documents.
4. Draft notes in this repository.
5. General model knowledge.

## Routing Contract

- `kb.manifest.json` is the machine-readable catalog.
- `indexes/routing-index.md` is the human-readable routing table.
- `indexes/summary-index.md` is the compact session-start overview.
- `indexes/tags.json` maps tags to document IDs.
- `docs/**` contains the SSOT documents.

Do not read every document by default. Use the index to choose a narrow set.

