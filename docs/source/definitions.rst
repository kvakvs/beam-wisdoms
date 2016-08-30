BEAM Definitions
=================

For specific constant values, bit positions and bit sizes please always
refer to original beam sources, file: ``erl_term.h``

.. _def-box:

Boxed Value
    A term value, whose :ref:`primary tag <def-primary-tag>` is
    (``TAG_PRIMARY_BOXED=2``), contains a pointer to data on heap.

    On the heap first word of the box always has header tag
    (``TAG_PRIMARY_HEADER``) in 2 least-significant bits
    with subtag (following 4 bits) specifying what is inside the block. Knowing
    this allows scanning heap and interpreting found data.

    Box values are used to store larger values that will not fit into
    :ref:`Word <def-word>` size minus 4 bits (:ref:`immediate-1 <def-immed>`)
    or :ref:`Word <def-word>` size minus 6 bits
    (for immediate-2). Examples of box:
    **bigint**, **float**, **fun**, **export**, heap and refcounted **binary**,
    external **ports** and **pids** (with host name in them).

.. _def-cache-locality:

Cache Locality
    A mythical creature used to scare naughty developers. It may have either
    huge impact on your code performance or very little. Always run
    performance tests before you start optimizing for this.

.. _def-cp:

CP, Continuation pointer
    A raw pointer to a location in prepared BEAM code. It is used as return
    value and can only be found on stack, never in registers or on heap.

.. _def-header:

Header tag
    When a box is referenced or when a block of memory is stored in heap,
    its first :ref:`Word <def-word>` usually has few least-significant bits
    reserved (currently 6) with header tag (4 bits)
    and primary tag (2 bits ``TAG_PRIMARY_HEADER=0``), and the
    remaining bits may hold some other information. Often remaining bits are
    used as arity to define following contents size.

.. _def-immed:

Immediate
    A term value, whose :ref:`primary tag <def-primary-tag>` is 3 is said to
    contain an immediate value. 2 bits following the primary tag determine what
    the value type is (``TAG_IMMED1_*`` macros). If the immediate-1 tag is 2
    (``TAG_IMMED1_IMMED2`` macro) then 2 more bits are used to interpret the
    value type (``TAG_IMMED2_*`` macros).

    Examples of immediate:
    **small** integers, local **pids** and **ports**, **atoms**,
    empty list ``NIL``.
    An immediate value FITS in one :ref:`Word <def-word>`
    and does not reference any memory.

.. _def-heap:

Heap
    Dynamically allocated block of :ref:`Words <def-word>` used by a process.
    Heap can be resized with help of either ``ERTS_REALLOC*`` macro or by
    allocating a new fragment and moving data there using garbage collector.
    Data is placed onto heap sequentially from its start.

.. _def-primary-tag:

Primary tag
    When a term value is encoded, several least-significant bits (currently
    2 bits) are reserved to represent type of contained term.

    Term tag can be:
    a :ref:`box <def-box>` (``TAG_PRIMARY_BOXED=2``),
    a list (``TAG_PRIMARY_LIST=1``),
    a :ref:`header <def-header>` (``TAG_PRIMARY_HEADER=0``)
    or an :ref:`immediate <def-immed>` (``TAG_PRIMARY_IMMED1=3``).

.. _def-reduction:

Reduction
    Each instruction or a call has a cost, it uses imaginary units called
    'reductions' where 1 reduction is approximately one function call, and the
    rest are derived from this also approximately.

.. _def-registers:

Registers
    An array of :ref:`Words <def-word>` used to pass arguments in a function
    call. When a recursive call is made, affected registers are also saved onto
    the stack.

.. _def-scheduler:

Scheduler
    Scheduler is a loop which runs on a fixed CPU core and it either fetches
    and executes next instruction based on instruction pointer in current
    process, or takes next process in the queue. As soon as a process has been
    running for certain number of :ref:`reductions <def-reduction>` (say 2000
    but number may change), it is scheduled out and put to sleep, and next
    process takes its place and continues running where it left off. This allows
    some sort of fair scheduling where everyone is guaranteed a slice of time,
    no matter how busy some processes are.

.. _def-stack:

Stack
    A section of young heap of a process, which is used as temporary storage and
    return stack by a process. Stack begins at heap end and grows back until it
    meets heap write position (heap top). At this moment heap is considered full.

.. _def-term:

Term
    A term is any value in Erlang. Internally a term is a :ref:`Word <def-word>`
    with few least-significant bits reserved (2 to 6 bits depending on the value)
    which define its type. Remaining bits either contain the value itself (for
    :ref:`immediate <def-immed>` values) or a pointer to data on heap
    (:ref:`box <def-box>` values).

.. _def-terminate:

Terminating a Process
    An exit or kill signal is sent to a process which works similar to an
    exception. If process was able to catch an exit signal (``trap_exit``), then
    nothing else happens.

    Process that is going to die will free its memory, trigger all monitors
    and links, leave the process queue and get unregistered from the process
    registry.

.. _def-nonvalue:

THE_NON_VALUE
    Internal value used by emulator, you will never be able to see it from Erlang.
    It marks exception or special type of return value from BIF functions, also
    it used to mark memory during garbage collection.

    Depending on whether ``DEBUG`` macro is set and HiPE is enabled,
    ``THE_NON_VALUE`` takes value of primary float header
    (6 least-significant bits are ``0110-00``) with remaining bits set
    to either all 0 or all 1. Or it is all zero-bits :ref:`Word <def-word>`
    (marking a zero arity tuple on Heap), which never can appear in a register,
    thus marking it useful to be the special return value.

.. _def-word:

Word
    Machine-dependent register-sized unsigned integer. This will have width of
    32 bits on 32-bit architecture, and 64 on a 64-bit architecture.
    In BEAM source code Word can be unsigned (UWord) or signed (SWord).
