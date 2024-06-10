# Process Heaps ELI5

Everything in Erlang is a [](BEAM-Definitions.md#term).

A heap of the process is an array of [Terms](BEAM-Definitions.md#term). A stack is
another array of [Terms](BEAM-Definitions.md#term). Stack is located inside the heap.
[Registers](BEAM-Definitions.md#registers) — are array of [Terms](BEAM-Definitions.md#term) too!
Things on the heap are mostly arrays of [Terms](BEAM-Definitions.md#term), but tagged
with a [](BEAM-Definitions.md#header-tag) (see [](Data-Types-Memory-Layout.md)).

## Memory Carriers

Allocation in Erlang happens inside so-called “carriers”. They look like “zone memory”, used by games — big chunks of
system heap pre-allocated beforehand. Inside carriers the real allocation is happening. For simplicity, you can imagine
simple malloc/re-alloc, that’s how it's done.

This complication is done to fight memory fragmentation and is not important for the understanding. You can
see `erts/emulator/beam/erl_*alloc.c` (many those files, one per strategy of allocation). Emulator has command line
flags, which control allocation strategies (see [Erlang Documentation](http://erlang.org/doc/man/erts_alloc.html) flags
section).

## Allocating Inside Heap

When a process needs some memory, its `heap_top` is incremented and memory under it is ready to use. Some activities may
want to allocate on other process' heaps, for example sending a message will perform a copy to the receiving process.

No bookkeeping is happening inside a process heap. There is no tracking which word belongs where, but it is possible to
know what is stored in each memory cell by looking at the tag bits.

## Garbage Collection

Garbage collector traces known live values from registers and stack and saves them, then drops everything else.

When a heap comes to its capacity threshold (something like 75%), the process triggers garbage collection. A new bigger
heap may be allocated. Scanning generational garbage collection algorithm runs on the heap. The algorithm takes
the “[](BEAM-Definitions.md#roots)” and moves them to the new heap. Then it scans the remaining parts of source heap and
extracts more values, referred by the roots. After the scanning, source heap contains only dead values and algorithm
drops it.

"Scanning" means, that garbage collector follows the data [](BEAM-Definitions.md#roots) from the start to the end,
parsing everything it meets. "Generational" means, that algorithm splits data into young and old generation, assuming
that data often dies young, and old data is less likely to be freed. Also, the algorithm remembers the old
position (`mature`), where the previous scanning has finished. Any data below it is guaranteed to have not updated since
the last scan. This trick reduces amount of scanning and speeds up the algorithm.

In reality the picture is a bit more complicated. There can be one or two heaps with different placement logic applied
to them.

Having garbage collector run per process heap allows Erlang to keep collection latency low. Also, it doesn’t pause or
affect other processes on other schedulers. This is not a simple topic, but theory is all
here: [Garbage Collection Book](http://gchandbook.org/)

> See Also:
>
> * [](Process-Heap-Layout.md)
> * [](Data-Types-Memory-Layout.md)
> * [Garbage Collector in Erlang 19](https://www.erlang-solutions.com/blog/erlang-19-0-garbage-collector.html)
