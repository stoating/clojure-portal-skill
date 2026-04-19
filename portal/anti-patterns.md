# Portal Anti-Patterns

## Contents

- Lifecycle and memory
- Tap setup mistakes
- Viewer mistakes
- Runtime mistakes
- Data safety

---

## Leaving Portal Attached Forever

Portal retains submitted objects until they are cleared. This is useful for inspection but can keep large objects from garbage collection.

```clojure
;; During exploratory work:
(p/clear)
(remove-tap #'p/submit)
(p/close)
```

For long-running dev sessions, prefer a bounded custom tap list backed by an atom.

---

## Adding Duplicate Tap Handlers

Repeatedly evaluating `add-tap` can register multiple handlers if each eval creates a new function object.

```clojure
;; Risky when re-evaluated:
(add-tap (fn [x] (p/submit x)))

;; Better:
(defn submit [x] (p/submit x))
(add-tap #'submit)
(remove-tap #'submit)
```

Use vars for tap handlers so you can remove or redefine them predictably.

---

## Opening Before The Runtime Can Launch A UI

Browser ClojureScript may need popup permissions. Headless servers, containers, SSH sessions, and CI often cannot launch a local browser.

Use one of these instead:

- fixed `:port` plus a remote client
- CLI mode for shell data
- editor launchers only when the matching extension is installed
- standalone Portal server

---

## Treating Portal As Production Logging

Portal is a development inspection tool. Do not rely on `tap>` as the only production audit trail, metrics stream, or durable log. Use Portal alongside proper logging, tracing, and metrics.

---

## Pre-Rendering Data To Strings

```clojure
;; Weak: loses structure
(tap> (pr-str {:user/id 1 :roles #{:admin}}))

;; Better: keep data navigable
(tap> {:user/id 1 :roles #{:admin}})
```

Only send strings when the string content itself is what you need to inspect, such as markdown, JSON, SQL, or an HTTP body.

---

## Forcing The Wrong Viewer

Metadata like `:portal.viewer/default` should express intent, not compensate for malformed data. If a table viewer looks wrong, first check whether the value is a sequence of consistent row maps.

For strings that need markdown or code rendering, wrap with `portal.viewer` helpers instead of assuming metadata can be attached to the string.

---

## Sending Non-Serializable Remote Values

Remote clients can only send values supported by EDN, JSON, or Transit. Do not submit raw JVM objects, open streams, sockets, functions, atoms, or browser objects through the remote API. Datafy or summarize them first.

---

## Tapping Secrets

Portal makes data easy to inspect and copy. Redact secrets before tapping:

```clojure
(defn redact [m]
  (cond-> m
    (:password m) (assoc :password ::redacted)
    (:token m)    (assoc :token ::redacted)))

(tap> (redact config))
```

Avoid tapping full environment maps, auth headers, cookies, database URLs with credentials, and production customer data.
