# Agent instructions

This repository contains the **`portal`** skill - a structured set of Markdown files that teach an AI coding agent how to work with Portal, the Clojure/ClojureScript data inspection and navigation tool.

**Any agent that understands this `AGENTS.md` convention should:**

1. Treat `portal/SKILL.md` as the entry point - it contains a decision table pointing to the right reference file for the task.
2. Load reference files on demand based on that table:
   - `setup-and-api.md` - dependencies, `portal.api/open`, `p/submit`, `tap>`, sessions, editor launchers
   - `viewers.md` - default viewers, metadata, `portal.viewer` helpers, custom tap lists, exceptions, logs
   - `datafy-nav.md` - `clojure.datafy/datafy`, `nav`, metadata protocol extension, domain navigation
   - `nrepl-and-runtime.md` - nREPL middleware, eval metadata, shadow-cljs, uncaught error capture
   - `remote-and-cli.md` - remote clients, encodings, CLI parsing, standalone server
   - `anti-patterns.md` - memory retention, duplicate taps, headless runtimes, secrets, serializability
3. Prefer keeping tapped values as structured data; only convert to strings when text is the subject being inspected.
4. Warn users that Portal retains submitted objects until cleared and that secrets should be redacted before tapping.

This file follows the [agents.md](https://agents.md/) convention and is honored by OpenAI Codex CLI, Cursor, Aider, Zed, Amp, Gemini CLI, Google Jules, Windsurf, Factory, RooCode, and many others.

For Claude Code, the richer native format is `.claude-plugin/` + `portal/SKILL.md`.
