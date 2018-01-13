Efficiency ELI5
===============

Data copying
------------

Here are general rules regarding data copying and performance.

*   Data is stored in process heap most of the time.
*   As long as data is moved **inside** the owning process, it is **not copied**,
    only a pointer is passed.
*   If the data ever leaves the process, a message for example, it will be
    **copied** to its new owner.
*   If the data is moved to or from ETS, it behaves similarly, it will be
    **copied** (ETS has own heap separate from any process).
*   If the data is moved between a process and a port, it will be **copied**
    (ports behave similar to processes, and own resources in a similar way).
*   Large binaries >64 bytes are **never** copied, instead they are reference
    counted and reference is shared between processes. This is why you want to
    GC all processes which touched a large binary periodically, otherwise it
    may take a long time before the binary is freed.


Literals (Constants)
--------------------

All Erlang values in your module (literals) are saved in the module file and
then loaded when your module loads.
These values are stored in special memory area called constant pool and later
when you reload your code the garbage collector will move old constants to your
process heap (see note below, this has changed in OTP 20.0).

Accessing a literal value of any size is cheap. You can use this trick to define
large constant structures, lookup tuples with cheap element access,
store large binaries in code etc.

.. note::

    In Erlang versions prior to 20.0 a constant value will be copied when
    crossing process boundary (explained above in "Data Copying"). This has changed
    in version 20.0
    (OTP-13529 in `Release Notes <http://erlang.org/download/otp_src_20.0.readme>`_ )
