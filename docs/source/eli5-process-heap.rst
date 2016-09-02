Process Heaps ELI5
==================

Everything in Erlang is a :ref:`Term <def-term>`.

A heap of the process is an array of :ref:`Terms <def-term>`. A stack is
another array of :ref:`Terms <def-term>`. Stack is located inside the heap.
:ref:`Registers <def-registers>` --- are array of :ref:`Terms <def-term>` too!
Things on the heap are mostly arrays of :ref:`Terms <def-term>`, but tagged
with :ref:`header tag <def-header>` (see :doc:`memory-layout`).

Allocation
----------

Allocation in Erlang happens inside so called “carriers”. They look like
“zone memory”, used by games — big chunks of system heap pre-allocated
beforehand. Inside carriers the real allocation is happening. For simplicity
you can imagine simple malloc/realloc, that’s how its done.

This complication is done to fight memory fragmentation and is not important
for the understanding.
You can see ``erts/emulator/beam/erl_*alloc.c`` (many those files, one per
strategy of allocation). Emulator has command line flags, which control
allocation strategies (see http://erlang.org/doc/man/erts_alloc.html flags
section).

Garbage Collection ELI5
-----------------------

When a heap comes to its capacity threshold (something like 75%), the process
triggers garbage collection. A new bigger heap may be allocated. Scanning
generational garbage collection algorithm runs on the heap. The algorithm
takes “roots” — all known to be live values on stack and in registers, and
moves them to a new heap. Then it scans the remaining parts of source heap
and extracts more values, referred by the roots. After the scanning, source
heap contains only dead values and algorithm drops it.

In reality the picture is a bit more complicated. There can be one or two
heaps with different placement logic applied to them. Also there is mature
mark for values which survived previous GC.

Having garbage collector run per process heap allows Erlang to keep
collection latency low. Also it doesn’t pause or affect other processes on
other schedulers. This is not a simple topic, but theory is all here:
http://gchandbook.org/
