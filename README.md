# coffi
[![cljdoc badge](https://cljdoc.org/badge/org.suskalo/coffi)](https://cljdoc.org/d/org.suskalo/coffi/CURRENT)
[![Clojars Project](https://img.shields.io/clojars/v/org.suskalo/coffi.svg)](https://clojars.org/org.suskalo/coffi)

Coffi is a foreign function interface library for Clojure, using the new
[Project Panama](https://openjdk.java.net/projects/panama/) that's a part of the
incubator in Java 17. This allows calling native code directly from Clojure
without the need for either Java or native code specific to the library, as e.g.
the JNI does. Coffi focuses on ease of use, including functions and macros for
creating wrappers to allow the resulting native functions to act just like
Clojure ones, however this doesn't remove the ability to write systems which
minimize the cost of marshaling data and optimize for performance, to make use
of the low-level access Panama gives us.

## Installation
This library is available on Clojars. Add the following entry to the `:deps` key
of your `deps.edn`:

```clojure
org.suskalo/coffi {:mvn/version "0.1.0"}
```

## Usage
There are two major components to coffi and interacting with native code:
manipulating off-heap memory, and loading native code for use with Clojure.

In the simplest cases, the native functions you call will work exclusively with
built-in types, for example the function `strlen` from libc.

```clojure
(require '[coffi.ffi :as ffi :refer [defcfn defcstruct]])

(defcfn strlen
  "Given a string, measures its length in bytes."
  strlen [::ffi/c-string] ::ffi/long)

(strlen "hello")
;; => 5
```

The first argument to `defcfn` is the name of the Clojure var that will hold the
native function reference, followed by an optional docstring and attribute map,
then the C function identifier, including the name of the native symbol, a
vector of argument types, and the return type.

Coffi defines a basic set of primitive types:
- byte
- short
- int
- long
- long-long
- char
- float
- double
- pointer

Each of these types maps to their C counterpart. Values of any of these
primitive types except for `pointer` will be cast with their corresponding
Clojure function (with `long-long` mapping to the `long` function) when they are
passed as arguments to native functions. Additionally, the `c-string` type is
defined, although it is not primitive.

In addition, some compound types are also defined in coffi, including struct and
union types. For an example c struct and function:

```c
typedef struct point {
    float x;
    float y;
} Point;

Point zero(void) {
    Point res = {};

    res.x = 0.0;
    res.y = 0.0;

    return res;
}
```

The corresponding coffi definition is like so:

```clojure
(defcfn zero-point
  "zero" [] [::ffi/struct [[:x ::ffi/float] [:y ::ffi/float]]])

(zero-point)
;; => {:x 0.0,
;;     :y 0.0}
```

Writing out struct definitions like this every time would get tedious, so the
macro `defalias` is used to define a struct alias.

```clojure
(defalias ::point
  [::ffi/struct
   [[:x ::ffi/float]
    [:y ::ffi/float]]])

(defcfn zero-point
  "zero" [] ::point)
```

In addition to structs, there is also support for passing Clojure functions as
callbacks to native functions, as well as calling function pointers returned
from native functions as Clojure functions.

```clojure
(defcfn higher-order
  "higher_order" [[::ffi/fn [::ffi/c-string] ::ffi/int]] ::ffi/int)
```

TODO Talk about writing your own serdes

## License

Copyright © 2021 Joshua Suskalo

Distributed under the Eclipse Public License version 1.0.
