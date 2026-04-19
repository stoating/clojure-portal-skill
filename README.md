# Clojure Portal Skill

A structured Markdown skill that teaches AI coding agents how to work with Portal - a Clojure/ClojureScript tool for inspecting, navigating, and visualizing data from REPLs, apps, remote runtimes, and shell pipelines.

Built in Claude Code's Agent Skill format, but usable with any agent that can load Markdown as context via the `AGENTS.md` convention.

## Installation

Claude Code via this plugin marketplace:

```text
/plugin marketplace add stoating/clojure-portal-skill
/plugin install clojure-portal@clojure-portal-skill
```

Claude Code via the aggregate marketplace:

```text
/plugin marketplace add stoating/plugins
/plugin install clojure-portal@stoating
```

Manual context usage:

```bash
aider --read clojure-portal-skill/portal/SKILL.md
codex --ask-for-approval on-request
```

## Repository Layout

```text
.
.claude-plugin/
  marketplace.json
  plugin.json
AGENTS.md
README.md
portal/
  SKILL.md
  setup-and-api.md
  viewers.md
  datafy-nav.md
  nrepl-and-runtime.md
  remote-and-cli.md
  anti-patterns.md
```

Only `portal/` contains the skill content. The rest is metadata and cross-agent entry points.
