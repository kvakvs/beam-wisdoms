Types ELI5
==========

Erlang is dynamically typed language. It means that any variable,
field or function argument can contain any allowed value.

Type Specs
----------

There is the type description language, which you can use to
describe what YOU think your functions should take and return.
These type specs are optional and the compiler will run your program like
nothing happened even if a type violation exists
To ensure that your type specs do not contradict each other, there is
another tool called Dialyzer.

A type spec is a directive in source ERL file, which looks like
``-spec func_name(Arg :: integer(), _, Y :: any()) -> float().`` This is not
code, it does not run.

You can read more about specifying types of your program in
`Reference Manual <http://erlang.org/doc/reference_manual/typespec.html>`_
and other existing books.

Dialyzer
--------

Dialyzer takes compiled BEAM or ERL source files as input, then
tries to guess types (type inference). If some types collapse to no value
(``none()``) or incompatible types are found, Dialyzer considered it a
discrepancy and error is reported.

How this works in few words
```````````````````````````

Say you have written a function, ``f`` which takes one argument ``X`` and
returns something.
Dialyzer first does a general assumption that ``f`` is a ``fun()``, which
takes any value (``X :: any()``) and returns whatever (``any()``).
Then Dialyzer analyzes usage of this function and its code and
tries to reduce argument and return types to more narrow
specific set of types.

For example, if Dialyzer discovers that the only two places
which call your function ``f`` pass an integer or an atom
argument, then ``X``'s type is reduced to ``integer()|atom()``
(a set of two types).

Typer
-----

Typer is another optional tool, found in your OTP distribution.
Using same algorithm as Dialyzer, it infers the types for a given module and
prints them. You can take them and insert into your program for future use
or for better understanding.
