# Lazy

`Lazy` is a Carp module that provides support for lazy computations. The main
entry point exposed by the module is the `Lazy.lazy` macro which takes a body of
some computation and returns a corresponding lazy computation. Here's an example
of `Lazy` in action:

```clojure
(use Lazy)

(defn add-one [x] (+ x 1))

;; Traditional, strict evaluation. The following function application is
;; evaluated immediately.
(add-one 2)
=> 3

;; A lazy variant of the prior computation.
(defn lazy-add-one [x] (Lazy.lazy (+ x 1)))

;; The original computation is wrapped in a thunk, which defers the evaluation
;; of the underlying function application.
(lazy-add-one)
=> ƛ
;; Force a lazy computation to compute by making an additional call.
((lazy-add-one 1))
=> 2
```

## Typed Laziness

The `Lazy` module also exposes a submodule called `Typed`. The `Lazy.Typed`
module is functionally equivalent, except that it wraps all lazy computations in
a `Lazy` type. Computations wrapped in this type must be evaluated using an
explicit call to `force`.

While the untyped variant of the module is more flexible, the additional wrapper
type exposed by `Lazy.Typed` communicates more to callers--you can tell
precisely where lazy computations are expected or used by looking at a
function's signature.

```clojure
(use Lazy.Typed)

(defn lazy-add-one [x] (Lazy.Typed.lazy (+ x 1)))

;; Typed lazy computations return a member of the `Lazy` type, which contains
;; the underlying computation.
(lazy-add-one 2)
=> (Lazy ƛ)

;; Use the `force` function to compute a typed lazy computation.
;; Note that force expects a reference to a Lazy type value.
(Lazy.Typed.force &(lazy-add-one 2))
```

## Memoization

Lazy computations support memoization--any lazy computation that's bound to a
lexical environment won't be recomputed after it's computed for the first
time--instead, the return value is stored and returned immediately upon
subsequent invocations of the lazy computation.

```clojure
(defn add-one [x] (+ x 1))

;; This call isn't memoized, since two distinct lazy computations are
;; instantiated, even though their underlying computation is equivalent.
(+ ((Lazy.lazy (add-one 2))) ((Lazy.lazy (add-one 2))))
=> Computed Value (3)
=> Computed Value (3)
=> 6

;; Contrarily, bound computations are memoized:
(def lazy-add-one (Lazy.lazy (add-one 2)))
(+ (lazy-add-one) (lazy-add-one))
;; The call to lazy-add-one is only computed once. The second application of the
;; lazy computation reuses the computed value.
=> Computed Value (3)
=> 6
```

### Let over Lazy

It's often convenient to use ad-hoc memoization to store the result of a partial
computation. You can use let bindings to bind and memoize lazy computations in
function bodies:

```clojure
(defn lazy-plus-plus [x y]
  (let [m (Lazy.lazy (+ x y))]
    (+ (m) (m))))

(lazy-plus-plus 2 2)
=> Computed Value (4)
=> 8
```

## Limitations

This library does not play well with the Carp REPL. You'll find that a lot of
its functionality, such as memoization, doesn't work as expected when
definitions are called directly in a REPL environment.
