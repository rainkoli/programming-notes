# Repository Guidelines

## Project Structure & Module Organization

This repository is a personal programming knowledge base, primarily Markdown rather than an application.

- `notes/`: original explanations organized by Java, databases, algorithms, source-code analysis, career, and tools. Use `notes/inbox/` for temporarily unclassified material.
- `demos/`: runnable code and minimal reproductions; the current Java example lives in `demos/java/linkage-error/`.
- `resources/`: external links, reference material, and archives.
- `assets/` and `_includes/`: site assets and Jekyll includes; `index.md` is the GitHub Pages entry point.
- Directory `README.md` files provide navigation. Keep one primary copy of each topic and cross-link related areas.

## Build, Test, and Development Commands

There is no repository-wide build script, package manager configuration, or automated test runner. Edit Markdown directly and preview it in a Markdown viewer.

Run these commands from the repository root:

- `git diff --check`: detect whitespace errors in tracked changes.
- `git diff --stat`: review the scope of tracked edits and moves.
- `git status --short`: identify modified and newly added files before committing.

For Java experiments, document the required JDK, compilation commands, and expected behavior in the demo README. Do not assume the linkage-error example is a conventional clean build: its class-name mismatch and precompiled fixtures are intentional.

## Coding Style & Naming Conventions

Use lowercase kebab-case for new Markdown files, such as `method-resolution.md`; use `README.md` for directory indexes. Preserve Java class-name capitalization and match the existing four-space Java indentation.

Use descriptive headings, language-tagged code fences, and relative internal links. Preserve UTF-8 text, including Chinese content. No formatter or linter is configured. When moving content, update affected indexes, the root README, and `index.md`.

## Testing Guidelines

No automated test framework or coverage threshold is configured. Preview changed Markdown, verify links and images, and check technical examples against their stated runtime. For demo changes, record reproduction steps and observed results. Preserve `demos/java/linkage-error/*.class`: these are intentional fixtures explicitly retained by `.gitignore`.

## Commit & Pull Request Guidelines

Recent commits commonly use `docs(scope): description` and `refactor(scope): description`, with scopes such as `java`, `notes`, and `algorithms`. Follow that pattern and keep changes focused.

In pull requests, summarize the content or behavior changed, list affected sections and validation performed, and link related issues when applicable. Include screenshots for visible site-layout changes.
