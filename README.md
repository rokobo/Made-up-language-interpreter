# MUPL interpreter using Racket
This project has to do with MUPL (Made Up Programming Language). MUPL programs
are written directly in Racket by using the constructors defined by the structs defined at the beginning of `MUPL_interpreter.rkt`. The project's goal is to learn about interpreters, language design, recursion, lexical scope, etc.
## MUPL syntax

* If `s` is a Racket string, then `(var s)` is a MUPL expression (a variable use).

* If `n` is a Racket integer, then `(int n)` is a MUPL expression (a constant).

* If `e1` and `e2` are MUPL expressions, then `(add e1 e2)` is a MUPL expression (an addition).

* If `s1` and `s2` are Racket strings and `e` is a MUPL expression, then `(fun s1 s2 e)` is a MUPL expression (a
function). In `e`, `s1` is bound to the function itself (for recursion) and `s2` is bound to the (one) argument.
Also, `(fun #f s2 e)` is allowed for anonymous non-recursive functions.

* If `e1`, `e2`, and `e3`, and `e4` are MUPL expressions, then `(ifgreater e1 e2 e3 e4)` is a MUPL expression.
It is a conditional where the result is `e3` if `e1` is strictly greater than `e2` else the result is `e4`. Only one
of `e3` and `e4` is evaluated (for performance).

* If `e1` and `e2` are MUPL expressions, then `(call e1 e2)` is a MUPL expression (a function call).

* If `s` is a Racket string and `e1` and `e2` are MUPL expressions, then `(mlet s e1 e2)` is a MUPL expression
(a let expression where the value resulting `e1` is bound to `s` in the evaluation of `e2`).

* If `e1` and `e2` are MUPL expressions, then `(apair e1 e2)` is a MUPL expression (a pair-creator).

* If `e1` is a MUPL expression, then `(fst e1)` is a MUPL expression (getting the first part of a pair).

* If `e1` is a MUPL expression, then `(snd e1)` is a MUPL expression (getting the second part of a pair).

* `(aunit)` is a MUPL expression (holding no data, much like null in Racket). 

* If `e1` is a MUPL expression, then `(isaunit e1)` is a MUPL expression (testing for `(aunit)`).

* `(closure env f)` is a MUPL value where `f` is MUPL function (an expression made from `fun`) and `env` is an environment mapping variables to values. 

## Language implementation

### Problem 1 (Language conversion functions)

* Write a Racket function `racketlist->MUPLlist` that takes a Racket list and produces an analogous MUPL list with the same elements in the same order.

* Write a Racket function `MUPLlist->racketlist` that takes a MUPL list and produces an analogous Racket list (of MUPL values) with the same elements in the same order.

### Problem 2 (MUPL interpreter)
Write a MUPL interpreter `eval-exp`
that takes a MUPL expression `e` and either returns the MUPL value that `e` evaluates to under the empty
environment or calls Racket’s error if evaluation encounters a run-time MUPL type error or unbound
MUPL variable.

A MUPL expression is evaluated under an environment. In the interpreter, use a Racket list of Racket pairs to represent this environment (which is initially empty)
so that you can use without modification the `envlookup` function. Here is a description of
the semantics of MUPL expressions:

* All values (including closures) evaluate to themselves. For example, `(eval-exp (int 17)) `would
return `(int 17)`, not `17`.

* A variable evaluates to the value associated with it in the environment.

* An addition evaluates its subexpressions and assuming they both produce integers, produces the
integer that is their sum. 

* Functions are lexically scoped: A function evaluates to a closure holding the function and the
current environment.

* An `ifgreater` evaluates its first two subexpressions to values `v1` and `v2` respectively. If both
values are integers, it evaluates its third subexpression if `v1` is a strictly greater integer than `v2`
else it evaluates its fourth subexpression.

* An `mlet` expression evaluates its first expression to a value `v`. Then it evaluates the second
expression to a value, in an environment extended to map the name in the `mlet` expression to `v`.

* A `call` evaluates its first and second subexpressions to values. If the first is not a closure, it is an
error. Else, it evaluates the closure’s function’s body in the closure’s environment extended to map
the function’s name to the closure (unless the name field is `#f`) and the function’s argument-name to the result of the second subexpression.

* A `pair` expression evaluates its two subexpressions and produces a new `pair` holding the results.

* A `fst` expression evaluates its subexpression. If the result for the subexpression is a `pair`, then the
result for the `fst` expression is the `e1` field in the pair.

* A `snd` expression evaluates its subexpression. If the result for the subexpression is a `pair`, then
the result for the `snd` expression is the `e2` field in the pair.

* An `isaunit` expression evaluates its subexpression. If the result is an `aunit` expression, then the
result for the `isaunit` expression is the MUPL value `(int 1)`, else the result is the MUPL value
`(int 0)`.

### Problem 3 (More MUPL functions)
Write Racket functions that act like
MUPL macros so that users of these functions feel like MUPL is larger. The Racket functions produce
MUPL expressions that could then be put inside larger MUPL expressions or passed to `eval-exp`. In
implementing these Racket functions, do not use closure (which is used only internally in `eval-exp`).
Also do not use `eval-exp`.

* Write a Racket function `ifaunit` that takes three MUPL expressions `e1`, `e2`, and `e3`. It returns a
MUPL expression that when run evaluates `e1` and if the result is MUPL’s `aunit` then it evaluates `e2`
and that is the overall result, else it evaluates `e3` and that is the overall result. 

* Write a Racket function `mlet*` that takes a Racket list of Racket pairs `((s1, e1) ... (sn, en))` and a final MUPL expression `en+1`. In each pair, assume `si`
is a Racket string and
`ei`
is a MUPL expression. `mlet*` returns a MUPL expression whose value is `en+1` evaluated in an
environment where each `si`
is a variable bound to the result of evaluating the corresponding `ei`
for 1 ≤ i ≤ n. The bindings are done sequentially, so that each `ei`
is evaluated in an environment
where `s1` through `si−1` have been previously bound to the values `e1` through `ei−1`.

* Write a Racket function `ifeq` that takes four MUPL expressions `e1`, `e2`, `e3`, and `e4` and returns
a MUPL expression that acts like `ifgreater` except `e3` is evaluated if and only if `e1` and `e2` are
equal integers. Assume none of the arguments to `ifeq` use the MUPL variables `_x` or `_y`. Use this
assumption so that when an expression returned from ifeq is evaluated, `e1` and `e2` are evaluated
exactly once each.

### Problem 4 (Map functions)
 
* Bind to the Racket variable `MUPL-map` a MUPL function that acts like map. Your function should be curried: it should take a MUPL function and return a MUPL
function that takes a MUPL list and applies the function to every element of the list returning a
new MUPL list. 

* Bind to the Racket variable `MUPL-mapAddN` a MUPL function that takes an MUPL integer `i` and
returns a MUPL function that takes a MUPL list of MUPL integers and returns a new MUPL list of
MUPL integers that adds `i` to every element of the list. Use MUPL-map.
