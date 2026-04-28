# Portal Setup And API

## Contents

- Dependencies
- Basic REPL workflow
- `portal.api/open` options
- API functions
- Editor launchers
- ClojureScript notes

---

## Dependencies

Tools.deps:

```clojure
{:aliases
 {:dev
  {:extra-deps {djblue/portal {:mvn/version "0.63.1"}}}}}
```

One-off JVM REPL:

```bash
clj -Sdeps '{:deps {djblue/portal {:mvn/version "0.63.1"}}}'
```

Babashka:

```bash
bb -Sdeps '{:deps {djblue/portal {:mvn/version "0.63.1"}}}'
```

Leiningen:

```clojure
{:profiles {:dev {:dependencies [[djblue/portal "0.63.1"]]}}}
```

If Portal is in a non-`:dev` Leiningen profile, start the REPL with `with-profiles +portal`.

---

## Basic REPL Workflow

```clojure
(require '[portal.api :as p])

(def portal (p/open))
(add-tap #'p/submit)

(tap> {:hello "world"})
(tap> (range 10))

(p/clear)
(remove-tap #'p/submit)
(p/close)
```

Use `#'p/submit` with `add-tap` and `remove-tap` so reloading the namespace updates the function behind the var.

`p/open` returns an atom-like session handle. Deref it to pull the currently selected value back into the REPL:

```clojure
(def portal (p/open))
(prn @portal)
```

---

## `portal.api/open` Options

Common options:

| Option | Use |
|--------|-----|
| `:window-title` | Set the browser/app window title |
| `:theme` | Set the UI theme, such as `:portal.colors/nord` |
| `:value` | Provide the root value atom, useful for custom tap lists |
| `:app` | Open as a Chrome app window when supported |
| `:launcher` | Launch through an editor extension: `:vs-code`, `:intellij`, or `:emacs` |
| `:editor` | Enable editor commands while using a separate UI |
| `:port` | Bind the Portal HTTP server to a known port |
| `:host` | Bind to a specific host, defaulting to localhost |

Use a fixed `:port` when another process will submit remotely:

```clojure
(p/open {:port 5678})
```

Use editor launchers when the user has the matching Portal extension installed:

```clojure
(p/open {:launcher :vs-code})
(p/open {:launcher :intellij})
(p/open {:launcher :emacs})
```

---

## API Functions

```clojure
(p/submit value)  ; send one value to open Portal sessions
(p/tap)           ; convenience tap setup in some workflows
(p/clear)         ; clear submitted values
(p/close)         ; close UI/session
(p/docs)          ; view Portal docs locally, JVM/node
(p/url)           ; get session URL
(p/sessions)      ; inspect open sessions
```

Use `p/set-defaults!` for team or user defaults, but keep project code explicit when reproducibility matters.

---

## ClojureScript Notes

For browser ClojureScript, require `portal.web` instead of `portal.api`:

```clojure
(require '[portal.web :as p])

(def portal (p/open))
(add-tap #'p/submit)
```

Browser Portal windows may require popup permissions. For Node ClojureScript, use `portal.api` similarly to JVM usage.
