{% (def title "Fibers") %}
Janet has support for single-core asynchronous programming via coroutines, or fibers.
Fibers allow a process to stop and resume execution later, essentially enabling
multiple returns from a function. This allows many patterns such a schedules, generators,
iterators, live debugging, and robust error handling. Janet's error handling is actually built on
top of fibers (when an error is thrown, the parent fiber will handle the error).

A temporary return from a fiber is called a yield, and can be invoked with the `yield` function.
To resume a fiber that has been yielded, use the `resume` function. When resume is called on a fiber,
it will only return when that fiber either returns, yields, throws an error, or otherwise emits
a signal.

Different from traditional coroutines, Janet's fibers implement a signaling mechanism, which
is used to differentiate different kinds of returns. When a fiber yields or throws an error,
control is returned to the calling fiber. The parent fiber must then check what kind of state the
fiber is in to differentiate errors from return values from user defined signals.

To create a fiber, user the `fiber/new` function. The fiber constructor take one or two arguments.
The first, necessary argument is the function that the fiber will execute. This function must accept
an arity of zero. The next optional argument is a collection of flags checking what kinds of
signals to trap and return via `resume`. This is useful so
the programmer does not need to handle all different kinds of signals from a fiber. Any un-trapped signals
are simply propagated to the next fiber.

```janet
(def f (fiber/new (fn []
                   (yield 1)
                   (yield 2)
                   (yield 3)
                   (yield 4)
                   5)))

# Get the status of the fiber (:alive, :dead, :debug, :new, :pending, or :user0-:user9)
(print (fiber/status f)) # -> :new

(print (resume f)) # -> prints 1
(print (resume f)) # -> prints 2
(print (resume f)) # -> prints 3
(print (resume f)) # -> prints 4
(print (fiber/status f)) # -> print :pending
(print (resume f)) # -> prints 5
(print (fiber/status f)) # -> print :dead
(print (resume f)) # -> throws an error because the fiber is dead
```

## Using Fibers to Capture Errors

Besides being used as coroutines, fibers can be used to implement error handling (exceptions).

```janet
(defn my-function-that-errors [x]
 (print "start function with " x)
 (error "oops!")
 (print "never gets here"))

# Use the :e flag to only trap errors.
(def f (fiber/new my-function-that-errors :e))
(def result (resume f))
(if (= (fiber/status f) :error)
 (print "result contains the error")
 (print "result contains the good result"))
```
