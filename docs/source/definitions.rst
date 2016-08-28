BEAM Definitions
=================

For specific constant values, bit positions and bit sizes please always
refer to original beam sources, file: ``erl_term.h``

.. _primary-tag-label:

Primary tag
    When a term value is encoded, several least-significant bits (currently
    2 bits are reserved) to represent type of contained term.

Secondary tag
    When a box is referenced, its first :ref:`Word <word-label>` usually contains
    few least-significant bits (currently 6) with secondary tag, and the
    remaining bits may hold some other information. Often remaining bits are
    used is arity to define following contents size.

.. _box-label:

Box
    A term value, whose :ref:`primary tag <primary-tag-label>` is 2
    (``TAG_PRIMARY_BOXED`` macro) contains a pointer to data on heap. Box values
    are used to store larger values that will not fit into Word size minus
    4 bits (:ref:`immediate-1 <immediate-label>`) or Word size minus 6 bits
    (for immediate-2). Examples of box: bigint, float, fun, export, heap binary,
    external ports and pids with host name in them).

.. _immediate-label:

Immediate
    A term value, whose :ref:`primary tag <primary-tag-label>` is 3 is said to
    contain an immediate value. 2 bits following the primary tag determine what
    the value type is (``TAG_IMMED1_*`` macros). If the immediate-1 tag is 2
    (``TAG_IMMED1_IMMED2`` macro) then 2 more bits are used to interpret the
    value type (``TAG_IMMED2_*`` macros). Examples of immediate: small integers,
    local pids and ports, atoms, empty list ``NIL``. An immediate value FITS
    inside one machine word and does not reference any memory.

.. _word-label:

Word
    Machine-dependent register-sized unsigned integer. This will have width of
    32 bits on 32-bit architecture, and 64 on a 64-bit architecture.
    In BEAM source code Word can be unsigned (UWord) or signed (SWord).

.. _term-label:

Term
    A term is any value in Erlang. Internally a term is a :ref:`Word <word-label>`
    with few least-significant bits reserved (2 to 6 bits depending on the value)
    which define its type. Remaining bits either contain the value itself (for
    :ref:`immediate <immediate-label>` values) or a pointer to data on heap
    (:ref:`box <box-label>` values).

.. _cp-label:
CP, Continuation pointer
    A raw pointer to a location in prepared BEAM code. It is used as return
    value and can only be found on stack, never in registers or on heap.

.. _scheduler-label:

Scheduler
    Scheduler is a loop which runs on a fixed CPU core and it either fetches
    and executes next instruction based on instruction pointer in current
    process, or takes next process in the queue. As soon as a process has been
    running for certain number of :ref:`reductions <reduction-label>` (say 2000
    but number may change), it is scheduled out and put to sleep, and next
    process takes its place and continues running where it left off. This allows
    some sort of fair scheduling where everyone is guaranteed a slice of time,
    no matter how busy some processes are.

.. _reduction-label:

Reduction
    Each instruction or a call has a cost, it uses imaginary units called
    'reductions' where 1 reduction is approximately one function call, and the
    rest are derived from this also approximately.
