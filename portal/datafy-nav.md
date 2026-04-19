# Portal Datafy And Nav

## Contents

- When to use datafy/nav
- Datafy custom objects
- Nav relationships
- Metadata protocol extension
- Practical patterns

---

## When To Use Datafy/Nav

Use `clojure.datafy/datafy` when a value has a richer data representation than its printed form. Use `clojure.datafy/nav` when a selected field should navigate to related data.

Good fits:

- Java objects such as files, URLs, exceptions, JDBC results, and system resources
- Domain records where IDs should navigate to entities
- Namespaces, vars, specs, routes, and configuration graphs
- Lazy or remote data that should only load when selected

Avoid hiding side effects in `datafy`; use `nav` for deliberate follow-up lookup.

---

## Datafy Custom Objects

Extend `Datafiable` for types you own or can safely extend:

```clojure
(require '[clojure.core.protocols :refer [Datafiable]])

(extend-protocol Datafiable
  java.io.File
  (datafy [file]
    {:name          (.getName file)
     :absolute-path (.getAbsolutePath file)
     :directory?    (.isDirectory file)
     :file?         (.isFile file)
     :size          (.length file)
     :last-modified (.lastModified file)
     :uri           (.toURI file)
     :children      (seq (.listFiles file))
     :parent        (.getParentFile file)}))
```

Then submit datafied values:

```clojure
(require '[clojure.datafy :as d])
(require '[portal.api :as p])

(def submit (comp p/submit d/datafy))
(add-tap #'submit)
```

---

## Nav Relationships

Attach `nav` behavior when a field points to related data:

```clojure
(require '[clojure.core.protocols :as p])

(def db
  {:book   [#:book{:id 1 :title "1984" :author 10}]
   :person [#:person{:id 10 :fname "George" :lname "Orwell"}]})

(defn book-nav [_coll key value]
  (if (= key :book/author)
    (first (filter (comp #{value} :person/id) (:person db)))
    value))

(tap>
 (map #(with-meta % {`p/nav book-nav})
      (:book db)))
```

When the user navigates into `:book/author`, Portal can show the person instead of just the numeric ID.

---

## Metadata Protocol Extension

Prefer metadata-based `datafy` / `nav` for individual values when global protocol extension would be too broad:

```clojure
(require '[clojure.core.protocols :as p])

(with-meta value
  {`p/datafy (fn [x] ...)
   `p/nav    (fn [coll k v] ...)})
```

This keeps exploratory behavior local and avoids changing all instances of a host type.

---

## Practical Patterns

For database rows:

```clojure
(defn row-nav [_ k v]
  (case k
    :order/customer-id (lookup-customer v)
    :order/id          (lookup-order-lines v)
    v))
```

For Integrant systems, datafy resource handles into maps with stable identifiers and hide secrets. For Polylith workspaces, datafy bricks, projects, and interfaces into plain maps and use `nav` to move from references to definitions.

Never expose credentials, tokens, or large binary payloads by default. Replace secrets with redacted markers before tapping.
