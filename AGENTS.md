# Workspace Agent Guide

This workspace is primarily a research and preparation repo. Treat it as a mixed knowledge base plus reference-code workspace.

## Project Structure

- `paper.md`: primary reference for the original research paper. Prefer this first when paper details are needed.
- `paper presentation/`: secondary knowledge base containing presentation materials in Markdown and LaTeX (`.md`, `.tex`, `.sty`). Use these files for background, summaries, speech notes, and Q&A material.
- `code/original/PersonalWAB/`: upstream reference implementation and paper artifact. It includes the `PersonalWAB` benchmark/runtime, the `PUMA` pipeline, scripts, and docs.

## Current Development Status

- We may build a new code project based on `code/original/PersonalWAB` later.
- For now, `code/original/PersonalWAB` is reference-only. Read it to understand architecture, data flow, and experiment setup.
- Do not treat the reference code as the active implementation target unless the user explicitly asks for changes there.

## Source Priority

When gathering information, use sources in this order:

1. `paper.md`
2. Markdown and LaTeX files under `paper presentation/`
3. Source code and docs under `code/original/PersonalWAB/`

## Retrieval Restrictions

- Never retrieve, read, summarize, or cite PDF files in this workspace.
- Assume any needed PDF content is also available in `.md` or `.tex` sources.
- Prefer `.md`, `.tex`, and code files for all analysis.

## Working Style

- Keep summaries brief and grounded in the available source files.
- When discussing future implementation, clearly separate reference behavior from new planned code.
