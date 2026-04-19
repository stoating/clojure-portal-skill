# Portal Remote And CLI

## Contents

- Remote API
- Client runtimes
- Encodings
- CLI parsing
- Standalone server

---

## Remote API

Use the remote API when Portal should run in one process while another process sends values.

Host process:

```clojure
(require '[portal.api :as p])
(p/open {:port 5678})
```

JVM or Babashka client process:

```clojure
(require '[portal.client.jvm :as p])

(def submit (partial p/submit {:port 5678}))
(add-tap #'submit)

(tap> {:from :remote})
```

Node ClojureScript:

```clojure
(require '[portal.client.node :as p])
(def submit (partial p/submit {:port 5678}))
(add-tap #'submit)
```

Browser ClojureScript:

```clojure
(require '[portal.client.web :as p])
(def submit (partial p/submit {:port 5678}))
(add-tap #'submit)
```

Tapped values must be serializable by the selected encoding.

---

## Encodings

EDN is the default:

```clojure
(def submit (partial p/submit {:port 5678 :encoding :edn}))
```

Other supported encodings:

```clojure
(def submit (partial p/submit {:port 5678 :encoding :json}))
(def submit (partial p/submit {:port 5678 :encoding :transit}))
```

Use Transit for richer cross-runtime Clojure data. Use JSON only for JSON-compatible values.

---

## CLI Parsing

Add a CLI alias:

```clojure
{:aliases
 {:portal/cli
  {:main-opts ["-m" "portal.main"]
   :extra-deps
   {djblue/portal {:mvn/version "0.63.1"}
    ;; optional yaml support
    clj-commons/clj-yaml {:mvn/version "0.7.0"}}}}}
```

Then pipe data into Portal:

```bash
cat data.edn | clojure -M:portal/cli edn
cat data.json | clojure -M:portal/cli json
cat data.transit | clojure -M:portal/cli transit
cat data.yaml | clojure -M:portal/cli yaml
```

Babashka startup helper:

```bash
bb -cp `clojure -Spath -M:portal/cli` -m portal.main edn
```

---

## Standalone Server

Start a Portal server without embedding it into an application:

```bash
bb -cp `clojure -Spath -Sdeps '{:deps {djblue/portal {:mvn/version "0.63.1"}}}'` \
   -e '(require (quote [portal.api])) (portal.api/open {:port 53755}) @(promise)'
```

Then point remote clients at port `53755`.

---

## Git Dependencies And Local Source

When debugging Portal itself or testing unreleased changes, use a git dependency or local root:

```clojure
{:deps {djblue/portal {:git/url "https://github.com/djblue/portal"
                       :git/sha "..."}}}
```

For this workspace, the source checkout is at `sources/portal`.
