How OTP Source Would Benefit from C++
=====================================

Here are my personal thoughts + results of my experiments with transferring
OTP C source to C++ and listing benefits of doing so for real.

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

*   Macros with code should be replaced with typed inlines or ``constexpr``.
    This helps with type checking and debuggability.
*   Macros with numbers should be replaced with ``const`` and ``constexpr``.
*   Abusive prefixing on ``enum`` should be replaced with ``enum class``
    and namespaces.
*   Abusive prefixing on types and function names should go into namespaces.
*   Code duplication can be reduced by moving code into templated functions.
    Example: Garbage collection sweeps are generic, before R20 they were
    duplicated code.
*   Hiding a lot of small macros with short nondescript names and abstracting
    away things. Example: ``erl_term.h`` and all the macros in it can be
    generalized, hidden and type checked.
*   Size types can be tagged: word and byte counts can be made incompatible to
    reduce confusion and bugs, with conversion functions.
*   To develop the idea of tagging further, memory pointer types can be tagged
    (making them incompatible with other pointer types to reduce confusion
    when passing them as arguments).
*   "Zero abstraction cost" is one ground C++ principle. The idea is that much
    work can be done at the compile-time, based on type checking, inlining
    and optimization. **Amount of used abstractions has little impact on
    resulting machine code**.

How Hard Is To Migrate
----------------------

I've done several attempts with varying degree of success.
One must fix all C++ keywords used as variable and argument names (easy).

Because C++ is stricter to pointers and type casts, one must revisit all places,
where ``erts_alloc/realloc`` are happening and fix the type casts there.

Generated tables also use pointer conversion and the generation scripts must be
mended (several lines changes in few places).

[to be continued]
