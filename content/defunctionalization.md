+++
date = 2019-06-17
title = 'Defunctionalization'
tags = ['reasonml']
keywords = ['reasonml', 'defunctionalization']

[extra]
author = 'Gustavo Aguiar'
+++
Defunctionalizing is a technique to use high-order
functions in a language that does not support it.
For example, in a language that supports high-order
functions, we could write `fold` as:

```reason
let rec fold = (fn, initial, l) =>
  switch (l) {
  | [] => initial
  | [hd, ...tail] => fn(hd, fold(fn, initial, tail))
  };
let sum = l => fold((x, y) => x + y, 0, l);
let _ = sum([1, 2, 3]);
let add = (n, l) => fold((x, l) => [x + n, ...l], [], l);
let _ = add(2, [1, 2, 3]);
```

But in a language that do not, we have to abstract
somehow the `fn` argument, we could do that by
creating a type called `arrow(_, _)` which is a GADT
that holds information for our `sum` and `add` helper functions.

```reason
type arrow(_, _) =
  | FnPlus: arrow((int, int), int)
  | FnPlusCons(int): arrow((int, list(int)), list(int));
```

Here `FnPlus` is abstracting `(x, y) => x + y`, and
`FnPlusCons(int)` is abstracting `(x, l) => [x + n, ...l]`.
The first type argument for `FnPlus` is `(int, int)` which
will hold our function arguments, the second type argument is
a `int` that represents the result of the function. The same structure
applies to `FnPlusCons(int)`, but with an adittional `int` which will hold
what was the `n` in `let add = (n, l) => fold((x, l) => [x + n, ...l], [], l);`

Now we can write an `apply` helper function as:

```reason
let apply: type a b. (arrow(a, b), a) => b =
  (appl, v) => {
    switch (appl) {
    | FnPlus => {
      let (x, y) = v;
      x + y;
    }
    | FnPlusCons(n) => {
      let (x, l') = v;
      [x + n, ...l'];
    }
    }
  };
```

And use it in our new `fold` function:

```reason
let rec fold: type a b. (arrow((a, b), b), b, list(a)) => b =
  (f, u, l) => 
    switch (l) {
    | [] => u
    | [x, ...xs] => apply(f, (x, fold(f, u, xs)))
    }
```

Then it's done! We have successfully avoided the use of high order functions.

```reason
let sum = l => fold(FnPlus, 0, l);
let add = (n, l) => fold(FnPlusCons(n), [], l);

let _ = sum([1, 2, 3]); /* 6 */
let _ = add(2, [1, 2, 3]); /* [3, 4, 5] */
```

This concept is a building block towards understanding this [paper](https://www.cl.cam.ac.uk/~jdy22/papers/lightweight-higher-kinded-polymorphism.pdf).

More on this topic can be found here:  
[Defunctionalization at Work](https://www.brics.dk/RS/01/23/BRICS-RS-01-23.pdf)
[Defunctionalization](https://en.wikipedia.org/wiki/Defunctionalization)
[Definitional interpreters for higher-orderprogramming languages](https://surface.syr.edu/cgi/viewcontent.cgi?article=1012&context=lcsmith_other)
