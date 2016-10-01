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

Another success story: Porting MAME (multiple arcade machine emulator) from C to modern C++ https://www.youtube.com/watch?v=wAUnUWYaA5s


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

Compatible with C
`````````````````

C runs perfectly fine when compiled as C++, the transition can happen gradually.
Does not have to use certain features like RTTI and exceptions, they can be
disabled at compiler-level.

Compile-Time Everywhere
```````````````````````

Macros with code should be replaced with typed inlines or ``constexpr``.
This helps with type checking and debuggability.
Stepping into a function in a debugger works much better than into a macro.
Also type safety applies. [Better view in debugger]

Macros with numbers should be replaced with ``const`` and ``constexpr``.
This way they will have a type, a namespace, and possibly may change into
an enum.
[Type checking = less bugs]

Abusive prefixing on ``enum`` should be replaced with ``enum class``
and namespaces. Moving enums with smaller scope into relevant structs and
classes (serves as a namespace).
[Less collisions and confusions in naming]

Type Safety
```````````

C++ is more strict about types, pointer types and alignment. Example: Assigning
an ``Eterm*`` to a ``char*`` will change alignment requirement from 8 to 1,
possibly hiding a bug. [Prevents accidents]

Size types can be tagged: word and byte counts can be made incompatible to
reduce confusion and bugs, with conversion functions.
[Less word-byte-confusion-related bugs]

To develop the idea of tagging further, memory pointer types can be tagged
(making them incompatible with other pointer types to reduce confusion
when passing them as arguments).
[Less memory-type related bugs]

Things like ``memcpy`` invite errors, you have to be mindful about types and
sizes. C++ allows to rewrite this in a safe way, they even have ``std::copy``
that does the same in a safe way (often this compiles into a ``memcpy``
but with correct sizes and bounds).
The C alternative having a function per type with separate names is highly
impractical. [Safer memory bounds]

Things like ``erts_alloc`` invite errors because you have to remember data
sizes. C++ can make sizes deducted automatically and return a correct
pointer type. [Safer memory allocations]
[Safer deallocations if they check types too]

Divide and Rule
```````````````

Abusive prefixing on types and function names should go into namespaces.
Structs and classes also serve as namespaces (unlike in C).

Benefits are immense: names become shorter at the cost of specifying namespace
at the beginning of module or a scope (``using namespace X`` or
``namespace X = Y::Z``).
[Shorter names easier to read]

Hiding a lot of small macros with short nondescript names and abstracting
away things. Example: ``erl_term.h`` and all the macros in it can be
generalized, grouped under the name of eterm and type checked.
In modern C++ all functions in the class header file are implicitly inline.
[Fast code at no cost]

Generic Code
````````````

Code duplication can be reduced by moving code into templated functions.
Long functions can and should be broken into smaller, reducing the variable
scope and amount of code to read.
Example: Garbage collection sweeps are generic, before R20 they were
duplicated code.
[Less mistakes with copies of almost identical code]

This applies to all the aggressively macroed code, again.
[Stricter type control, less bugs, code is as fast as before]

"Zero abstraction cost" is the ground C++ principle.
Much work can be done at the compile-time, based on type checking, inlining
and optimization. **Amount of abstractions used should have little impact on
resulting machine code**.
[Code is fast as in C]

For example: wrapping ``Eterm`` into a class will
not affect its memory and code footprint, but give full control to the developer
on how the type is convertable, what goes in and how it comes out, how to copy
it, what to do when the type is destroyed, also will allow to group all tool
macros and functions inside that class.
[Code is as fast as in C]

Many loops can be remade generic using higher order functions.
C++ compiler often is smart enough to inline them without degrading machine
code quality.
[Code is as fast as in C] [Less bugs when using generic algorithms]

Performance and Safety
```````````````````````

Smart pointers and RAII should be used for temporary memory (resource)
allocations reducing manual resource management. Let compiler remember, when
you have to reduce that refcount or free that temporary buffer.
[Automatic resource management at little to no cost]

With const-correctness we gain better control on what can be changed where.
Current OTP source is not const-correct and safety is enforced through
convoluted locking and thorough thinking + debugging.
Const-correct code is transparent on what memory it touches, and is
easier to parallellize. Pure const-correct code is even better.
[Harder to create bad code]

How Hard Is To Migrate
----------------------

I've done several attempts with varying degree of success.
One must fix all C++ keywords used as variable and argument names (easy).

Because C++ is stricter to pointers and type casts, one must revisit all places,
where ``erts_alloc/realloc`` are happening and fix the type casts there.

Generated tables also use pointer conversion and the generation scripts must be
mended (several lines changes in few places).

Note: https://www.youtube.com/watch?v=w5Z4JlMJ1VQ
CppCon 2016: Tim Haines “Improving Performance Through Compiler Switches..."
