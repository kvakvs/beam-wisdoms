BEAM Definitions
=================

For specific constant values, bit positions and bit sizes please always
refer to original beam sources, file: ``erl_term.h``

.. _def-box:

Boxed Value
    A term value, whose :ref:`primary tag <def-primary-tag>` is
    (``TAG_PRIMARY_BOXED=2``), contains a pointer to data on heap.

    The first word of the box always has header tag (``TAG_PRIMARY_HEADER=0``)
    in its 2 least-significant bits. Then goes the subtag (following 4 bits)
    which determines type of the value inside the box. Knowing this allows
    scanning heap and interpreting found data.

    VM uses boxes to store larger values, which do not fit into
    :ref:`Word <def-word>` size minus 4 bits (:ref:`immediate-1 <def-immed>`)
    or :ref:`Word <def-word>` size minus 6 bits
    (for immediate-2).

    Examples of box:
    **bigint**, **float**, **fun**, **export**, heap and refcounted **binary**,
    external **ports** and **pids** (with host name in them).

.. _def-cache-locality:

Cache Locality
    A mythical creature used to scare naughty developers. It may have either
    huge impact on your code performance or very little. Always run
    performance tests before you start optimizing for this.

Context
    Virtual machine context is a small structure which contains a copy of
    active registers, instruction pointer, few other pointers from the
    currently running process. It is stored in CPU registers or on stack
    (based on C compiler judgement when building the emulator).

    When a process is ready to run, its fields are copied into the context,
    i.e. *swapped in*.
    When a process is suspended or scheduled out, the context is stored back
    into the process. This operation is called *swapping out*.
    This is not a cheap operation, so VM tries to minimize context switches.

    A context switch is also performed when changing from BEAM code to native
    compiled code by HiPE or JIT. This is why mixing HiPE and normal Erlang
    code in your program, which calls each other, is an expensive luxury.

.. _def-cp:

CP, Continuation pointer
    A raw pointer to a location in prepared BEAM code. It is used as return
    value and can only be found on stack, never in registers or on heap.

.. _def-header:

Header tag
    When a box is referenced or when a block of memory is stored in heap,
    its first :ref:`Word <def-word>` usually has few least-significant bits
    reserved (currently 6). First goes the primary tag (2 bits of
    ``TAG_PRIMARY_HEADER=0``). Then follows the header tag (4 bits) which
    defines the type of contents. Remaining bits may hold other information,
    often these bits store the arity (contents size).

.. _def-immed:

Immediate
    A term value, whose :ref:`primary tag <def-primary-tag>` is
    ``TAG_PRIMARY_IMMED1=3`` contains an immediate value. Two bits follow the
    primary tag and determine the value type (``TAG_IMMED1_*`` macros).
    If the immediate-1 tag equals ``TAG_IMMED1_IMMED2=2`` then two more bits
    are used to interpret the value type (``TAG_IMMED2_*`` macros).

    Examples of immediate:
    **small** integers, local **pids** and **ports**, **atoms**,
    empty list ``NIL``.
    An immediate value fits into one :ref:`Word <def-word>`
    and does not reference any memory.

.. _def-heap:

Heap
    Dynamically allocated block of :ref:`Words <def-word>` used by a process.
    Heap can be resized with help of either ``ERTS_REALLOC*`` macro or by
    allocating a new fragment and moving data there using garbage collector.
    Data is placed onto heap sequentially from its start.

.. _def-port:

Port
    A special value which you receive when you call ``erlang:open_port``.
    It is hooked to a port driver (built-in or custom). You can send it commands
    and receive messages from it, you can close it, link to it and monitor it
    (monitoring added in v.19).

    A port driver manages some resource, such as a file, a socket, a ZIP
    archive etc. Ports provide your process with a stream of events from
    some resource, and you can write commands and data to them as well.

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

.. _def-roots:

Roots
    During garbage collection, the roots are all known to be live values, found
    on the stack and in the registers. Anything that can be traced by following
    references in roots is considered to be reachable data. This data is moved
    to the new heap. Previous heap is discarded, because no data can be
    reached on it anymore.

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

Slot
    A special tagged value which points at a register, float register, or a
    stack slot. It is used internally by instructions and never appears in
    Erlang programs as data.

.. _def-stack:

Stack
    A section of young heap of a process, which is used as temporary storage and
    return stack by a process. A new process creates a stack which has zero size
    and begins at the ``heap_end``.
    Stack grows back (decrementing memory address) until it meets
    heap write position (``heap_top``). Then heap is considered full
    and garbage collection will trigger.

    Data on stack is grouped into :ref:`Stack Frames <def-stack-frame>`.

.. _def-stack-frame:

Stack Frame
    Functions can create a stack frame by pushing a :ref:`CP <def-cp>` value
    and reserving several extra words on stack. Sometimes, when code throws an
    exception, VM scans the stack to build a stacktrace and uses these CP values
    as markers.

    Each frame corresponds to a function call. A frame always begins with a
    CP value which marks a return address can be used to find a frame boundary.
    Rest of the frame is used to store any temporary variables and register
    values between the calls.

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
