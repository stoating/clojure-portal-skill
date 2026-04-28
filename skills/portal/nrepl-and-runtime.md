# Portal nREPL And Runtime Integration

## Contents

- nREPL middleware
- Eval result customization
- Test and stdio capture
- Shadow CLJS
- Uncaught errors

---

## nREPL Middleware

`portal.nrepl/wrap-portal` can submit REPL eval information to Portal:

- source and file info
- runtime type, especially useful in `.cljc`
- eval timings
- test assertion output
- eval exceptions
- stdio

You still need to add Portal as a tap target:

```clojure
(require '[portal.api :as p])
(add-tap #'p/submit)
(p/open)
```

Tools.deps nREPL alias:

```clojure
{:aliases
 {:nrepl
  {:extra-deps
   {cider/cider-nrepl {:mvn/version "0.28.5"}
    djblue/portal     {:mvn/version "0.63.1"}}
   :main-opts ["-m" "nrepl.cmdline"
               "--middleware"
               "[cider.nrepl/cider-middleware,portal.nrepl/wrap-portal]"]}}}
```

Leiningen:

```clojure
{:profiles
 {:dev
  {:dependencies [[djblue/portal "0.63.1"]]
   :repl-options {:nrepl-middleware [portal.nrepl/wrap-portal]}}}}
```

---

## Eval Result Customization

The nREPL middleware marks eval values with metadata. Process them with a custom submitter when you want to split stdio, reports, and results:

```clojure
(require '[portal.api :as p])

(defn submit [value]
  (if (-> value meta :portal.nrepl/eval)
    (let [{:keys [stdio report result]} value]
      (when stdio (p/submit stdio))
      (when report (p/submit report))
      (p/submit result))
    (p/submit value)))

(add-tap #'submit)
```

Return `:portal.api/ignore` from an expression when it should not be tapped by the middleware.

---

## Shadow CLJS

Add middleware in `shadow-cljs.edn`:

```clojure
{:nrepl {:middleware [portal.nrepl/wrap-portal]}}
```

For browser ClojureScript, use `portal.web` in the runtime where values are tapped:

```clojure
(require '[portal.web :as p])
(def portal (p/open))
(add-tap #'p/submit)
```

---

## Uncaught Errors

Browser ClojureScript:

```clojure
(defn error-handler [event]
  (tap> (or (.-error event) (.-reason event))))

(.addEventListener js/window "error" error-handler)
(.addEventListener js/window "unhandledrejection" error-handler)
```

Node ClojureScript:

```clojure
(.on js/process "unhandledRejection" tap>)
(.on js/process "uncaughtException" tap>)
```

JVM:

```clojure
(Thread/setDefaultUncaughtExceptionHandler
 (reify Thread$UncaughtExceptionHandler
   (uncaughtException [_thread error]
     (tap> error))))
```

Datafy exceptions before submission when you want Portal's exception viewer to render structured exception data.
