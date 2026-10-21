# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A personal, long-lived programming knowledge base written almost entirely in Markdown, published as a GitHub Pages/Jekyll site (`index.md` is the portal, `_includes/head-custom.html` injects the favicon and MathJax). There is no application build, package manager configuration, formatter, linter, or automated test suite — edit Markdown directly and preview it in a Markdown viewer.

- `notes/` — original knowledge notes; the single home for explanatory content
- `demos/` — minimal runnable/compilable code and reproductions
- `resources/` — external links, reference material, historical archive
- `assets/`, `_includes/` — site-only files; not learning resources
- Every content directory has a `README.md` index (the one exception to kebab-case filenames)

## Commands

Run from the repository root:

- `git diff --check` — detect whitespace errors in tracked changes
- `git diff --stat` — review scope of edits and moves
- `git status --short` — see modified/new files before committing

Java demos are compiled and run manually with `javac`/`java`; document the required JDK, commands, and expected output in the demo's `README.md`. `demos/java/linkage-error/` is intentionally not a clean build — its class-name case mismatch and committed `.class` fixtures are the experiment itself. Those `.class` files are explicitly un-ignored in `.gitignore`; do not delete them.

## Content placement

Decision rule: content that runs/reproduces → `demos/`; external links, templates, or non-original material → `resources/`; your own knowledge write-up → `notes/`, choosing the one domain where you'd look for it later.

Boundaries that determine the location between similar-looking options:

- Java API usage/behavior → `notes/java/`; JDK class implementation details → `notes/source-code-analysis/jdk/`
- Spring/MyBatis usage, config, and concepts → `notes/java/frameworks/`; source call chains and internals → `notes/source-code-analysis/`
- Reusable algorithm knowledge → `notes/algorithms/topics/`; individual solutions → `notes/algorithms/problems/` grouped by source (leetcode, luogu, nowcoder, interview, standalone)

One primary copy per topic — cross-link instead of duplicating files. Unclassifiable notes go to `notes/inbox/` temporarily and should be moved out once their home is clear. Create directories only when content exists (roughly 3–5 related files before adding a subdirectory); keep nesting within four levels unless a stable identifier (platform name, spec chapter, project name) justifies more.

## Editing conventions

- New Markdown files use lowercase kebab-case (e.g. `method-resolution.md`); stable identifiers may prefix names (problem numbers, JLS chapter numbers)
- Preserve UTF-8 and existing Chinese content; match the surrounding file's language for new notes
- Language-tag every code fence; use relative links for internal references
- Moving or renaming a file requires updating its directory `README.md`, the domain index, root `README.md`, and `index.md`
- MathJax is loaded globally; `$...$` inline math is enabled in addition to default delimiters

## Commits

Follow the dominant history style: `docs(scope): 中文描述` or `refactor(scope): 中文描述`, with scopes such as `java`, `notes`, `algorithms`, `resources`, `database`. Keep each commit focused; write descriptions in Chinese to match.
