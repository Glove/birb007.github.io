---
layout: post
title: "Abusing Mathematics to Annoy Nerds"
date: 2019-11-28
---

There are many ways to abuse identities and use obscure tricks to obfuscate what your code does but my favourite method at the moment is abusing lambda calculus. Welcome to your bootleg introduction to lambda calculus. To demonstrate why lambda calculus is so effective at obfuscating code, take a look at this:

```py
def factorial(n: int) -> int:
    if n < 0:
        return 1
    return n * factorial(n-1)

assert factorial(3) == 6
```

is functionally equivalent to

```py
assert (
                                   lambda f:(lambda x: x(x)
                                )(lambda y:     f(lambda x: y(y)(x))))(
                     lambda f: lambda n: (        lambda n: n(
                      lambda f: lambda x:             lambda y: y())(
                     lambda x: lambda y:                 x()))(n)(
                      lambda: lambda f:                     lambda x: f(x))(
                    lambda: (lambda x:                        lambda y: lambda f: y(
                        x(f)))(n)(f((                             lambda n: (lambda p: p(
            lambda x: lambda y: y))(                           n((lambda p: (
                lambda a: lambda b:                         lambda s: s(a)(b))(
    (lambda n: lambda f: lambda x:                       f(n(f)(x)))(
                    (lambda p: p(                     lambda x: lambda y: x))(p)
                  ))((lambda p: p(                 lambda x: lambda y: x))(p))
  ))((lambda a: lambda b: lambda s:             s(a)(b))(lambda f: lambda x: x
            )(lambda f: lambda x: x))        ))(n)))))(
                             lambda f:    lambda x: f(
                             f(f(x))))(lambda x: x + 1)(0) == 6
```
Complicated right? Not really.

## Humble Beginnings
Initially we build trivial, atomic, expressions.

First boolean expressions to allow for predicate logic.
```py
TRUE = lambda x: lambda y: x
FALSE = lambda x: lambda y: y

AND = lambda x: lambda y: x(y)(FALSE)
OR = lambda x: lambda y: x(TRUE)(y)
NOT = lambda x: x(FALSE)(TRUE)

assert AND(TRUE)(TRUE) == TRUE
assert OR(NOT(TRUE))(FALSE) == FALSE
```

Now we represent the natural numbers using Church numerals.
```py
ZERO = lambda f: lambda x: x
SUCC = lambda n: lambda f: lambda x: f(n(f)(x))
```

Next we implement expressions for arithmetic.
```py
ADD = lambda x: lambda y: x(SUCC)(y)
MUL = lambda x: lambda y: lambda f: x(y(f))
```

Clearly we are missing the, arguably, important ability to subtract. Subtraction is finicky in lambda calculus and requires some extra work since we are only able to count up. Consider, $$x-y$$ where $$x, y \in \mathbb{N}$$, if we count up two numbers, $$a, b \in \mathbb{N}$$, to $$x$$ but initially $$a = y$$ and stop when $$a = x$$. Notice than when $$a = x$$, $$b = x-y$$ hence we can perform subtraction! To count up pairs of values we need a structure to store this pair.
```py
CONS = lambda x: lambda y: lambda s: s(x)(y)
CAR = lambda p: p(TRUE)
CDR = lambda p: p(FALSE)
```

I get it, you're looking at this going "_what the fuck_" but let me explain. `CONS` constructs a pair of lists, we combine this with other pairs to build a list. `CAR` and `CDR` grab the first and second element of the pair, respectively. An example of constructing list is as follows:
```py
assert CAR(CONS(ONE)(CONS(TWO)(CONS(THREE)(ZERO)))) == ONE # is equivalent to [1,2,3][0]
```

Now! We construct a pair and perform the computation described above.
```py
T = lambda p: CONS(SUCC(CAR(p)))(CAR(p))
PRED = lambda n: CDR(n(T)(CONS(ZERO)(ZERO)))
```

From `PRED` we can now define subtraction.
```py
SUB = lambda x: lambda y: y(PRED)(x)
```

Before we continue there is a limitation within Python, Python uses strict evaluation which means arguments are evaluated before their evaluation requirement. To demonstrate this:
```py
In [1]: def f(x):
   ...:     return 1
   ...:

In [2]: f(4/0)
---------------------------------------------------------------------------
ZeroDivisionError                         Traceback (most recent call last)
<ipython-input-2-3d43e26063d5> in <module>
----> 1 f(4/0)

ZeroDivisionError: division by zero
```

`x` isn't required to evaluate for `f` but Python evaluates the argument regardless. We define two functions to allow for lazy evaluation which we require.
```py
LAZY_TRUE = lambda x: lambda y: x()
LAZY_FALSE = lambda x: lambda y: y()
```

Almost done! We define `ISZERO` which is close to the final step:
```py
ISZERO = lambda n: n(lambda f: LAZY_FALSE)(LAZY_TRUE)
```

Finally, to have a perfectly recursively function we use the Y-combinator.
```py
Y = lambda f: (lambda x: f(lambda z: x(x)(z)))(lambda x: f(lambda z: x(x)(z)))
FACT = Y(lambda n: ISZERO(n)(lambda: ONE)(lambda: MUL(n)(FACT(PRED(n)))))
```

If you made it this far, congratulations! Expand all the terms until you're left with `lambda`s et voila! To do this yourself simply use the building blocks above to construct a function and expand it back out. If you want to learn more I highly recommend [Beazley's talk](https://www.youtube.com/watch?v=pkCLMl0e_0k), and for anyone really interested [this book](https://www.cs.kent.ac.uk/people/staff/sjt/TTFP/ttfp.pdf) is a great introduction to functional programming from a mathematical perspective. May I present, the Ackermann function.
```py
assert (
    lambda f: lambda m: lambda n: (
        lambda n: n(lambda x: lambda x: lambda y: y)(lambda x: lambda y: x)
    )(m)((lambda n: lambda f: lambda x: f(n(f)(x)))(n))(
        (lambda n: n(lambda x: lambda x: lambda y: y)(lambda x: lambda y: x))(n)(
            f(
                (
                    lambda n: (lambda p: p(lambda x: lambda y: y))(
                        n(
                            lambda p: (lambda x: lambda y: lambda s: s(x)(y))(
                                (lambda n: lambda f: lambda x: f(n(f)(x)))(
                                    (lambda p: p(lambda x: lambda y: x))(p)
                                )
                            )((lambda p: p(lambda x: lambda y: x))(p))
                        )(
                            (lambda x: lambda y: lambda s: s(x)(y))(
                                lambda f: lambda x: x
                            )(lambda f: lambda x: x)
                        )
                    )
                )(m)
            )(lambda f: lambda x: f(x))
        )(
            f(
                (
                    lambda n: (lambda p: p(lambda x: lambda y: y))(
                        n(
                            lambda p: (lambda x: lambda y: lambda s: s(x)(y))(
                                (lambda n: lambda f: lambda x: f(n(f)(x)))(
                                    (lambda p: p(lambda x: lambda y: x))(p)
                                )
                            )((lambda p: p(lambda x: lambda y: x))(p))
                        )(
                            (lambda x: lambda y: lambda s: s(x)(y))(
                                lambda f: lambda x: x
                            )(lambda f: lambda x: x)
                        )
                    )
                )(m)
            )(
                f(m)(
                    (
                        lambda n: (lambda p: p(lambda x: lambda y: y))(
                            n(
                                lambda p: (lambda x: lambda y: lambda s: s(x)(y))(
                                    (lambda n: lambda f: lambda x: f(n(f)(x)))(
                                        (lambda p: p(lambda x: lambda y: x))(p)
                                    )
                                )((lambda p: p(lambda x: lambda y: x))(p))
                            )(
                                (lambda x: lambda y: lambda s: s(x)(y))(
                                    lambda f: lambda x: x
                                )(lambda f: lambda x: x)
                            )
                        )
                    )(n)
                )
            )
        )
    )(
        lambda f: (lambda x: f(lambda z: x(x)(z)))(lambda x: f(lambda z: x(x)(z)))
    )(
        lambda f: lambda m: lambda n: (
            lambda n: n(lambda x: lambda x: lambda y: y)(lambda x: lambda y: x)
        )(m)((lambda n: lambda f: lambda x: f(n(f)(x)))(n))(
            (lambda n: n(lambda x: lambda x: lambda y: y)(lambda x: lambda y: x))(n)(
                f(
                    (
                        lambda n: (lambda p: p(lambda x: lambda y: y))(
                            n(
                                lambda p: (lambda x: lambda y: lambda s: s(x)(y))(
                                    (lambda n: lambda f: lambda x: f(n(f)(x)))(
                                        (lambda p: p(lambda x: lambda y: x))(p)
                                    )
                                )((lambda p: p(lambda x: lambda y: x))(p))
                            )(
                                (lambda x: lambda y: lambda s: s(x)(y))(
                                    lambda f: lambda x: x
                                )(lambda f: lambda x: x)
                            )
                        )
                    )(m)
                )(ONE)
            )(
                f(
                    (
                        lambda n: (lambda p: p(lambda x: lambda y: y))(
                            n(
                                lambda p: (lambda x: lambda y: lambda s: s(x)(y))(
                                   (lambda n: lambda f: lambda x: f(n(f)(x)))(
                                        (lambda p: p(lambda x: lambda y: x))(p)
                                    )
                                )((lambda p: p(lambda x: lambda y: x))(p))
                            )(
                                (lambda x: lambda y: lambda s: s(x)(y))(
                                    lambda f: lambda x: x
                                )(lambda f: lambda x: x)
                            )
                        )
                    )(m)
                )(
                    f(m)(
                        (
                            lambda n: (lambda p: p(lambda x: lambda y: y))(
                                n(
                                    lambda p: (lambda x: lambda y: lambda s: s(x)(y))(
                                        (lambda n: lambda f: lambda x: f(n(f)(x)))(
                                            (lambda p: p(lambda x: lambda y: x))(p)
                                        )
                                    )((lambda p: p(lambda x: lambda y: x))(p))
                                )(
                                    (lambda x: lambda y: lambda s: s(x)(y))(
                                        lambda f: lambda x: x
                                    )(lambda f: lambda x: x)
                                )
                            )
                        )(n)
                    )
                )
            )
        )
    )(
        lambda f: lambda x: f(x)
    )(
        lambda f: lambda x: f(f(x))
    )(
        lambda n: n + 1
    )(
        0
    )
    == 4
)
```
I cannot test this given Python explodes since it near instantly hits the recursion depth. However, if you can, please let me know!

