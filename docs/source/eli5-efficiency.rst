Efficiency ELI5: Data copying
=============================

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
