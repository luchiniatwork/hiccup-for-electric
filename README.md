# hiccup-for-electric

[![Clojars][clojars-badge]][clojars]
[![License][license-badge]][license]
![Status][status-badge]

`hiccup-for-electric` is a library to use the [Hiccup][hiccup] format
for representing HTML in [Electric Clojure][electric]. In traditional
Hiccup fashion, it uses vectors to represent elements, and maps to
represent an element's attributes but instead of Hiccup's generation
of HTML, `hiccup-for-electric` bridges Electric Clojure's macros that
can be used in Electric apps.

## Installation

Add the following dependency to your `deps.edn` file:

    luchiniatwork/hiccup-for-electric {:mvn/version "TBD"}


## Usage

Here is a basic example of Hiccup syntax in practice:

```clojure
(ns app
  (:require [hyperfiddle.electric :as e]
            [hiccup-for-electric.core :as h]))

(e/defn Component []
  (e/client
    (h/electric
      [:span {:class :foo} "bar"])))
```

Given the snippet above, `Component` would be equivalent to the
following in basic Electric:

```clojure
(ns app
  (:require [hyperfiddle.electric :as e]
            [hyperfiddle.electric-dom2 :as dom]))

(e/defn Component []
  (e/client
    (dom/span
      (dom/props {:class :foo})
      (dom/text "bar"))))
```

Which ultimately means the following in HTML:

```html
<span class=\"foo\">bar</span>
```

The first element of the vector is used as the element name. Any map
element will be folded left to supply the element's attributes. The
default behavior is that every other element is either a vector
representing another element part of the tag's body or, if it's
anything else than a vector, it will be considered a `dom/text`
node. This behavior can be changed (see below).

Hiccup provides a CSS-like shortcut for denoting `id` and `class`
attributes and these are also supported:

```clojure
(h/electric [:div#foo.bar.baz "bang"])
```
Is equivalent to:

```clojure
(dom/div
  (dom/props {:id :foo
              :class [:bar :baz]})
  (dom/text "bang"))
```

Any DOM element can be used - including web components. For instance,
if you are using [Shoelace][shoelace] for example, the following will
render a gear icon:

```clojure
(h/electric [:sl-icon {:name "gear"}])
```

Electric sometimes favors macros for certain use cases. For instance,
the `hyperfiddle.electric-ui4` namespace has a bunch of useful
macros. In order to use them via `hiccup-for-electric` you can require
and then use the namespace on the keyword:

```clojure
(ns app
  (:require [hyperfiddle.electric :as e]
            [hyperfiddle.electric-ui4 :as ui]
            [hiccup-for-electric.core :as h]))

(e/defn Component []
  (e/client
    (h/electric
      [:div
        ^{:text-mode :last-one}
        [::ui/button (e/fn []
                       (prinln "Clicked"))
          "Click Me"]]
```

Special attention to `^{:text-mode :last-one}` which is explained in
the next session.


## Text behavior

A lot of the magic behind Electric is wrapped in its many macros. The
`dom/text` macro in particular is pretty useful. In order to extract
as much value from it as possible the `hiccup-for-electric` compiler
has three operating modes: `:default`, `:last-one` and `:none`.

In the `:default` mode, only a single element in the Hiccup vector can
be not a map (it does not represent the element's attributes) and a
not a vector (it does not represent a sub element). That single
element is wrapped in the `dom/text` macro. In practice it means that
you can do things like this:

```clojure
(e/client
  (let [<st e/system-time-ms]
    (h/electric [:div (str "reactive client time: " <st)])))
```

The mode can be set via `:text-mode` in the element's vector's
meta. The following is an example of the `:last-one` mode. In this
mode, only the last non-map/non-vector element will be wrapped in a
`dom/text`. This is very useful for many of the macros from
`hyperfiddle.electric-ui4`. In the following example, the `e/fn`
remains a first parameter sent to the `button` macro and `"Click Me"`
(the last entry) is wrapped in `dom/text`:

```clojure
(e/client
  (h/electric
    ^{:text-mode :last-one}
    [::ui/button (e/fn []
                   (prinln "Clicked"))
      "Click Me"]))
```

The `:none` mode is definitely the simplest and it turns off any
`dom/text` wrapping. This is useful when several components are
part of a stack. For instance:

```clojure
(e/defn Page []
  (e/client
    (h/electric
      ^{:text-mode :none}
      [:div
        (ComponentA.)
        (ComponentB.)
        (ComponentC.)])))
```

This approach makes it very natural to mix Hiccup and Electric code.


## Differences to Hiccup

In `hiccup-for-clojure` a map does not have to be the second
element. All maps are folded left during the compilation.

`hiccup-for-clojure` treats the element's body differently to
accomodate `dom/text`. This may make some Hiccup code incompatible
with `hiccup-for-clojure`.

Hiccup is intelligent enough to render different HTML elements in
different ways, in order to accommodate browser
quirks. `hiccup-for-clojure` does not deal with HTML directly.

`hiccup-for-clojure` does not expand content out based on specific
symbols such as how Hiccup does with `for`.


## Further References

* [Hiccup's original Wiki][hiccup-wiki]
* [Hiccup's original API Docs][hiccup-api]
* [Electric Clojure][electric]


## License

Distributed under the MIT Public License. Use it as you
will. Contribute if you have the time.

[hiccup]: https://github.com/weavejester/hiccup
[hiccup-wiki]: https://github.com/weavejester/hiccup/wiki
[hiccup-api]: http://weavejester.github.io/hiccup
[electric]: https://github.com/hyperfiddle/electric
[shoelace]: https://shoelace.style

[license-badge]: https://img.shields.io/badge/license-MIT-blue.svg
[license]: #license

[clojars-badge]: https://img.shields.io/clojars/v/luchiniatwork/hiccup-for-electric.svg
[clojars]: http://clojars.org/luchiniatwork/hiccup-for-electric

[status-badge]: https://img.shields.io/badge/project%20status-prod-brightgreen.svg
