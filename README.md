# chezscheme-define-macro

Sometimes you just want define-macro to get on with it!

## Quickstart

### Example: checking properties on constants at compile time

```scheme
(import (define-macro))

(define-macro (check f x)
  `(unless (,f ,x)
     (assertion-violationf 'check "could not verify property (~a ~a)" (quote ,f) ,x)))
```

#### Usage

```scheme
(check even? 5)
```

This yields the following compilation error

```
Exception in check: could not verify property (even? 5)
```

### Example: instrumentation of code

```scheme
(import (define-macro))

(define-macro (instrument code)
  (if (pair? code)
      (case (car code)
        [let
         (let ([bindings (cadr code)] [body (cddr code)])
           `(let ,(map (lambda (var/val)
                        (let ([var (car var/val)] [val (cadr var/val)])
                          `(,var (let ([tmp ,val]) (break ',val "~a" tmp) tmp))))
                      bindings)
                ,@(map (lambda (b) `(let ([tmp ,b]) (break ',b "~a" tmp) tmp))
                       body)))]
        [else code])
      code))
```

#### Usage

```scheme
(instrument
  (let ([x (add1 4)] [y (sub1 6)] [z 2])
    (+ x y)
    (- y z)
    (* x y z)))
```

Inserts break statements into a `let` form such that you can step trough it at runtime and see it evaluating step by step. _I actually didn't know that `let` evaluated its bindings in reverse order before I made this example!_

```
Break in 2: 2
break> e

Break in (sub1 6): 5
break> e

Break in (add1 4): 5
break> e

Break in (+ x y): 10
break> e

Break in (- y z): 3
break> e

Break in (* x y z): 50
break> e
50
```

Imagine if you instrumented the code with a communication mechanism with an editor like Visual Studio Code such that the programmer can step through the code expression by expression!
