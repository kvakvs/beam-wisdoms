Processes ELI5
===============

Very high level overview (Explain me like I'm five).

General overview
----------------

A process is a C structure, which refers to heap, stack (an array of
:ref:`Terms <term-label>` inside heap), registers (function args), instruction
pointer, and some other extra fields for exception handling etc. Spawning a
process is basically creating this structure and empty heap (array of terms)
in dynamic memory.

A newly created process is placed on a :ref:`Scheduler <scheduler-label>`.

Sending a message to a process is a simple operation: Lock the process mailbox
(if we're in SMP), copy term being sent to process' own heap, add the resulting
term to its mailbox. Unlock. A wake up signal is sent if process is frozen in
receive, so it gets scheduler time when possible.
If process is waiting for a message (stuck in receive operator) it will never
be queued for execution until a message is found. This is why millions of mostly
idle processes are able to run on a single machine without even reaching high
CPU% usage.

Killing a process is somehow similar to sending it an exit exception. It will
wake up from wait, receive CPU time, discover an exception and either exit
(free its memory, trigger all monitors and links and get out of process queue
and process registry) or catch the exception and process it like a regular
value. An unconditional ``kill`` signal works similarly except it cannot be
caught from Erlang code.

Load balancing ELI5
-------------------

Typically one Erlang scheduler per CPU core is launched. Processes are
assigned to them in some manner (for simplicity you can say it's random).
How many schedulers are started is configured at VM start with ``+S`` flag, also
see flag ``+SP``. Schedulers can be bound to cores in different manners (``+sbt``
flag).

During the runtime schedulers sometimes will eagerly look at other schedulers
(namely at the previous one in scheduler array) and compare their process queue with
the other. If the other queue is longer, the scheduler will steal one or several
processes from the other. Described is the default behaviour. This behavior
depends on the chosen balancing strategy and it can be configured at VM start
with ``+S`` and ``+scl`` flags. You may prefer using up as few cores as possible
to let other CPU cores sleep and save energy, or you may prefer equal spread of
processes to minimize latency.

Stealing is as easy as moving a pointer to a Process struct from one list to
another. This of course damages cache locality a bit when a thread jumps CPU
core, but benefit from it was considered acceptable versus the drawbacks.
