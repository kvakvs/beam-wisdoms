BIF and NIF functions ELI5
==========================

BIF Functions
-------------

Unless you work on OTP C source, you will never have to create own BIF function.
If you want to implement own native function for your project,
check out the next section about NIFs!

In the standard Erlang library most functions are implemented in Erlang. But
many features of the virtual machine and internal functions are impossible
to reach from pure Erlang. So they were written in C and are exported as BIF
--- built-in functions.
BIFs are used in standard Erlang libraries and are statically built into
emulator during compile-time.

When reading Erlang standard library modules you often can see functions
with a single call to ``erlang:nif_error(...).``
These are BIF stubs.
BEAM loader finds the native library and replaces these stubs with references
to native implementation in C.
You can also create BIF or NIF functions in other languages like C++ or Rust.
You should also register a new BIF in a special file, called ``bif.tab`` which
connects module:function name and the BIF when you build the emulator.

If you are curious, search for some function in OTP C source, for example:
``lists_reverse_2`` in ``erl_bif_lists.c``.
A BIF function takes ``Process`` pointer and a pointer to registers, where it
can access as many :ref:`Term <def-term>` (``Eterm`` C type) registers as it needs.
A BIF must return a :ref:`Term <def-term>` value or a ``THE_NON_VALUE``
for special execution control features like traps, yields and exceptions.


NIF Functions
-------------

NIFs are a different way of making native functions, more suited for separate
compilation and to be loaded by user modules.
NIF interface and type system is also simplified --- they abstract away and
hide many internal types, bits and fields of the emulator.

There is a good `NIF tutorial`_ in standard documentation and about a million
of NIF functions written by users and available in various Github projects.

.. _NIF tutorial: http://erlang.org/doc/tutorial/nif.html

Still even if it is simplified, one must be careful! A badly written NIF is
able to tip over the whole virtual machine or hog the resources and slow the
execution to a grind.

.. seealso::
    BEAM Wisdoms: :doc:`interfacing` for a list of NIF, interoperability
    libraries, and port drivers for different programming languages.
