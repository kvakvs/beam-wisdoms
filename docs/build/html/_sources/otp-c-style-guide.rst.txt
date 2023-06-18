Erlang/OTP Source Style Guide
=============================

General
-------

Source language is C89.

Source uses mix of tabs and spaces. Tab equals 8 spaces, indent is 4 spaces.
It seems that spaces are rather accepted way to indent contributions.

Version Control System
----------------------

Erlang/OTP uses Git as its version control system.

Commits should have max 60 characters in the first line. Long commit description
follows after one empty line.

The source tree perfectly should be compilable at any time between commits (it does
not have to pass all tests between commits though), to simplify git bisect searches.

Headers
-------

The #define Guard
`````````````````

All headers should have #define guards to prevent multiple inclusion. The format of
the symbol name is ``__MODULENAME_H__`` or ``__MODULENAME_H``.
Pragma once is not accepted as of Sept 2016.

Naming Macros
`````````````

All macro names are defined using uppercase letters, numbers and underscores.

For new macros a general suggestion for type safety is: if it's possible to write
it as inline function, one should not create a macro and use inline
(``ERTS_INLINE`` or ``ERTS_FORCE_INLINE``) instead.

Prefer to not convert existing macros to inline without a reason (they are well
tested as is now and their compiled representation may slightly change with
inline, that is not always desirable).
Also when going from a macro to a function, its name should be changed
accordingly (lowercase + prefix), this may have some undesired consequences
for dependent modules and external code using Erlang API.

Naming Types
````````````

A new type should be named in CamelCase. A publicly visible type must have
longer prefix. For example garbage collection functions have prefix ``erts_*``,
``erts_gc_*``, and types have prefix ``ErtsGC*``

Do not rename existing types without a good reason, especially if it was a
part of the public API.

Naming Functions
````````````````

Functions are named using lowercase letters, numbers and underscores.

Function arguments are named using lowercase letters, numbers and underscores.

A globally visible non-inline function should have an ``erts_`` prefix to its
name.

All inline functions should use ``ERTS_INLINE`` or ``ERTS_FORCE_INLINE`` macro.

A local function inside module should be declared ``static``.

Naming Variables
````````````````

Local variables are named using lowercase letters, numbers and underscores.

It is suggested to have local variables defined as close to their first use as
possible.
A new variable can be declared at the beginning of any scope
(inside an ``if``, ``for``, ``do`` etc), not just at the start of the function
body.

Consider using ``const`` if variable is only set once at the moment of its
declaration.

Consider using a strict sized type (``Uint64`` or ``Uint32`` for example) when
you definitely know limits of your data. Otherwise consider using sizable
(``Word``) and similar.

Coding conventions
------------------

Argument Passing
````````````````

If a function takes a pointer to a memory block, consider using ``const``
pointer type to pass it as read only (``const char *`` as a mutable pointer
to an immutable array of ``char``).

If a function should return more than one value, pass pointer to the
container for the result or pass values using their address.
It is suggested to either mark such arguments in comment as ``in/out`` or have a
prefix ``_out``.

To avoid mistakes, try not having similar types following each other in
argument list.
This way you'll get compiler warning if you miss one argument position.
