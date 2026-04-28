---
name: portal
description: Use when working with Portal - a Clojure/ClojureScript data inspection and navigation tool. Activate when the user asks about portal.api/open, p/submit, tap>, add-tap, viewers, default viewers, portal.viewer metadata, datafy/nav, nREPL middleware, remote clients, CLI parsing of EDN/JSON/Transit/YAML, editor launchers, or debugging Clojure data, exceptions, logs, REPL evals, and tapped values with Portal.
version: 1.0.0
---

# Portal

Portal is a Clojure/ClojureScript tool for inspecting, navigating, and visualizing data from a REPL, app runtime, command line, or remote process. The normal workflow is: open a Portal UI, add `portal.api/submit` as a `tap>` target, then inspect tapped values as data instead of printing strings.

Coordinates: `djblue/portal {:mvn/version "0.63.1"}`.

## Quick Decision: What do you need?

| Task | Go to |
|------|-------|
| Add Portal to a project, open the UI, wire `tap>`, clear/close sessions | [setup-and-api.md](setup-and-api.md) |
| Choose or force viewers, wrap values for markdown/hiccup/table/log/exceptions | [viewers.md](viewers.md) |
| Use `datafy` / `nav` to browse domain objects, files, namespaces, or DB rows | [datafy-nav.md](datafy-nav.md) |
| Capture REPL evals, test reports, stdio, timings, and eval exceptions via nREPL | [nrepl-and-runtime.md](nrepl-and-runtime.md) |
| Send values from another process/runtime or parse data from shell pipelines | [remote-and-cli.md](remote-and-cli.md) |
| Something is leaking memory, not rendering, blocked by popups, or not receiving taps | [anti-patterns.md](anti-patterns.md) |

## Core Mental Model

Portal is not a logging framework or state manager. It is a data browser connected to one or more data sources:

```clojure
runtime value -> tap> -> portal.api/submit -> Portal UI -> viewer + navigation
```

- Prefer `tap>` for exploratory visibility; leave production logging to app logging.
- Keep values data-shaped. Avoid pre-rendering everything to strings unless the text itself is the subject.
- Use metadata or `portal.viewer` helpers to guide the default viewer when the automatic viewer is not the one you want.
- Use `clojure.datafy/datafy` and `clojure.datafy/nav` when domain objects need richer browsing than their raw printed form.
- Clear or close Portal during long sessions; Portal retains submitted objects until cleared.
