Processes ELI5
===============

This is a high level overview (explain me like I'm five).

General overview ELI5
---------------------

A process is a simple C structure, which contains
a :ref:`heap <def-heap>`,
a :ref:`stack <def-stack>`,
:ref:`registers <def-registers>`,
and an instruction pointer. Also, there are some extra fields for exception
handling, tracing etc. A new process is this structure created with a minimal
size heap.

Every new process is assigned to a :ref:`Scheduler <def-scheduler>`.

Messages
````````

Sending a message to a process is simple — this is how VM does it:

1.  Lock the process mailbox (or don't, if running on a single core).
2.  Copy message to destination process heap.
3.  Add the resulting term to process mailbox.
4.  Unlock the mailbox.
5.  If the process was sleeping in a receive, it would return back to
    scheduling queue and wake up when possible.

A process waiting for a message (in receive operator) is never queued for
execution until a message arrives. This is why millions of idle processes can
exist on a single machine without it breaking a sweat.

Messages are stored in the heap or in a heap fragment, and are chained together
using a single linked list.

Killing and Exiting
```````````````````

Killing a process is like sending it an exit exception. The process wakes up
from sleep, receives CPU time, and discovers an exception. Then it will either
:ref:`terminate <def-terminate>` or catch the exception and process it like
a regular value. An unconditional ``kill`` signal works similarly except that
Erlang code cannot catch it.

Load balancing ELI5
-------------------

Typically one Erlang scheduler per CPU core is launched. Processes are
assigned to them in some manner (for simplicity you can say it is random).
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

Process Registry ELI5
---------------------

A global process table maps process identifier (pid) to a Process structure.
The reverse mapping is done via ``Process.common.id`` field which is
an :ref:`Eterm <def-term>` field containing pid. This way a process can always
be uniquely identified by its local pid. Remote pids are larger and contain node
name and internal node id, and they will have to be resolved on the node which
owns them.

Another global table (process registry) maps names to pid. You can reach it from
Erlang by using ``erlang:register``, ``erlang:unregister`` and ``erlang:whereis``
function calls.

Message Queues ELI5
-------------------

Message queue is a C structure which belongs in Process struct,
it contains :ref:`Eterms <def-term>`
sent to the process, while the term :ref:`boxed data <def-box>` will be located
in this process' heap. A pointer to position in the queue exists, and it is
moved forward, as BEAM opcodes for reading the mailbox are executed. When pointer
reaches end of the mailbox, it is wrapped and scanning is started over -- this is
why selective receive on very large mailboxes becomes gradually slower.

To send a message (on C level of VM source) you find process by name (using
registered names registry) or by pid (using global pid registry) and having
Process struct you lock it and manipulate its message queue.
