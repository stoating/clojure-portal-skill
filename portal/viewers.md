# Portal Viewers

## Contents

- Viewer model
- Default viewers with metadata
- `portal.viewer` helpers
- Tables and custom tap lists
- Exceptions and logs
- Text, markdown, hiccup, and binary data

---

## Viewer Model

A viewer renders a selected value. Portal filters viewers by predicate and lets users switch viewers in the UI. Code should only force a viewer when the intended representation is part of the data contract.

Set the default viewer with metadata:

```clojure
(tap> ^{:portal.viewer/default :portal.viewer/hiccup}
      [:h1 "hello"])
```

For computed values:

```clojure
(tap> (with-meta rows
        {:portal.viewer/default :portal.viewer/table}))
```

Values such as strings do not support metadata; wrap them in a container or use `portal.viewer` helpers.

---

## `portal.viewer` Helpers

```clojure
(require '[portal.viewer :as v])

(tap> (v/text "plain text"))
(tap> (v/markdown "# Title\n\nBody"))
(tap> (v/table [{:id 1 :name "Ada"}]))
(tap> (v/hiccup [:div [:strong "Rendered as hiccup"]]))
(tap> (v/json "{\"ok\": true}"))
(tap> (v/edn "{:ok true}"))
(tap> (v/bin (byte-array [0 1 2 3])))
```

Useful viewer keywords include:

| Viewer | Use |
|--------|-----|
| `:portal.viewer/tree` | General nested data |
| `:portal.viewer/table` | Sequences of similar maps or rows |
| `:portal.viewer/hiccup` | HTML or Portal viewer markup |
| `:portal.viewer/markdown` | Markdown documents |
| `:portal.viewer/log` | Structured log events |
| `:portal.viewer/exception` | Datafied exceptions |
| `:portal.viewer/diff` | Structural diffs |
| `:portal.viewer/vega-lite` | Vega-Lite chart specs |

---

## Dynamic Default Viewer

Wrap `p/submit` when values should choose viewers by predicate:

```clojure
(require '[portal.api :as p])
(require '[portal.viewer :as v])

(def defaults
  {string? v/text
   bytes?  v/bin})

(defn viewer-for [value]
  (or (some (fn [[pred viewer]]
              (when (pred value) viewer))
            defaults)
      v/tree))

(defn submit [value]
  (p/submit ((viewer-for value) value)))

(add-tap #'submit)
```

---

## Tables And Custom Tap Lists

Use an atom as the Portal root value when you want to control retention, ordering, or shape:

```clojure
(require '[portal.api :as p])

(def viewer
  {:portal.viewer/default :portal.viewer/table
   :portal.viewer/table {:columns [:id :value :time]}})

(def taps (atom (with-meta [] viewer)))
(def ids (atom 0))

(defn submit [value]
  (let [id (swap! ids inc)]
    (swap! taps
           (fn [xs]
             (conj (if (< (count xs) 25) xs (subvec xs 1))
                   {:id id :value value :time (java.util.Date.)})))))

(add-tap #'submit)
(p/open {:value taps})
```

---

## Exceptions

On the JVM, datafy exceptions before submission:

```clojure
(require '[clojure.datafy :as d])
(require '[portal.api :as p])

(defn submit [value]
  (p/submit
   (if (instance? Exception value)
     (assoc (d/datafy value) :runtime :clj)
     value)))

(add-tap #'submit)
```

For browser ClojureScript, map `js/Error` to data with `:runtime`, `:cause`, `:via`, and `:stack`.

---

## Logs

Portal includes `portal.console` macros that tap log-shaped data with source, time, runtime, and level:

```clojure
(require '[portal.console :as log])

(log/trace ::starting)
(log/debug {:event ::loaded})
(log/info  "ready")
(log/warn  {:slow? true})
(log/error (ex-info "failed" {}))
```

This is useful for REPL investigation. Do not replace application logging solely with Portal taps.
