How OTP Source Would Benefit from C++
=====================================

Here are my personal thoughts + results of my experiments with transferring
OTP C source to C++ and listing benefits of doing so for real.

Here is a brilliant case study on converting a C library to C++ with benefits
listed:
J. Daniel Garcia and B. Stroustrup:
`Improving performance and maintainability through refactoring in C++11 <http://www.stroustrup.com/improving_garcia_stroustrup_2015.pdf>`_.
Isocpp.org. August 2015. They managed to reduce instruction count, improve
branch miss ratio, thus improving the performance.


Portability Benefits
--------------------

Currently compiling Erlang/OTP on Windows is pain. It is built using MSVC for
the C89 part, except that ``beam_emu.c`` is built using GCC.
This is happening because "jump to label address" extension is not supported on
MSVC which falls back to switch dispatch and creates much slower code.

Microsoft stated, that they will not be investing any time into supporting
modern C standards with MSVC.
So there are only two ways out: use another compiler on Windows (gcc or Clang)
or switch to C++ and use MSVC natively.

Code Benefits
-------------

Compile-Time Everywhere
```````````````````````

Macros with code should be replaced with typed inlines or ``constexpr``.
This helps with type checking and debuggability.
Stepping into a function in a debugger works much better than into a macro.
Also type safety applies.

Macros with numbers should be replaced with ``const`` and ``constexpr``.
This way they will have a type, a namespace, and possibly may change into
an enum.

Abusive prefixing on ``enum`` should be replaced with ``enum class``
and namespaces. Moving enums with smaller scope into relevant structs and
classes (serves as a namespace).

Type Safety
```````````

Size types can be tagged: word and byte counts can be made incompatible to
reduce confusion and bugs, with conversion functions.

To develop the idea of tagging further, memory pointer types can be tagged
(making them incompatible with other pointer types to reduce confusion
when passing them as arguments).

Divide and Rule
```````````````

Abusive prefixing on types and function names should go into namespaces.
Structs and classes also serve as namespaces (unlike in C).

Benefits are immense: names become shorter at the cost of specifying namespace
at the beginning of module or a scope (``using namespace X`` or
``namespace X = Y::Z``).

Hiding a lot of small macros with short nondescript names and abstracting
away things. Example: ``erl_term.h`` and all the macros in it can be
generalized, grouped under the name of eterm and type checked.
In modern C++ all functions in the class header file are implicitly inline.

Generic Code
````````````

Code duplication can be reduced by moving code into templated functions.
Long functions can and should be broken into smaller, reducing the variable
scope and amount of code to read.
Example: Garbage collection sweeps are generic, before R20 they were
duplicated code.

This applies to all the aggressively macroed code, again.

"Zero abstraction cost" is the ground C++ principle.
Much work can be done at the compile-time, based on type checking, inlining
and optimization. **Amount of abstractions used should have little impact on
resulting machine code**.

For example: wrapping ``Eterm`` into a class will
not affect its memory and code footprint, but give full control to the developer
on how the type is convertable, what goes in and how it comes out, how to copy
it, what to do when the type is destroyed, also will allow to group all tool
macros and functions inside that class.

Many loops can be remade generic using higher order functions.
C++ compiler often is smart enough to inline them without degrading machine
code quality.

Performance and Safety
```````````````````````

Smart pointers and RAII should be used for temporary memory (resource)
allocations reducing manual resource management. Let compiler remember, when
you have to reduce that refcount or free that temporary buffer.

With const-correctness we gain better control on what can be changed where.
Current OTP source is not const-correct and safety is enforced through
convoluted locking and thorough thinking + debugging.
Const-correct code is transparent on what memory it touches, and is
easier to parallellize. Pure const-correct code is even better.


How Hard Is To Migrate
----------------------

I've done several attempts with varying degree of success.
One must fix all C++ keywords used as variable and argument names (easy).

Because C++ is stricter to pointers and type casts, one must revisit all places,
where ``erts_alloc/realloc`` are happening and fix the type casts there.

Generated tables also use pointer conversion and the generation scripts must be
mended (several lines changes in few places).

[to be continued]
