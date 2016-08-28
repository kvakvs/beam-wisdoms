Process Heaps
==============

The heap is allocated inside so called carriers, these look like "zone memory"
used by games -- big chunks of system heap pre-allocated beforehand, and inside
them the real allocation is happening. For simplicity you can imagine simple
malloc/realloc, that's how its done. This is not essential for understanding how
the thing works, but is mostly done to fight memory fragmentation: you can see
``erts/emulator/beam/erl_*alloc.c`` (many those files, one per strategy of
allocation).

Garbage Collection
------------------

To resize process heap or when the heap comes to its capacity threshold
(something like 75%), a new bigger heap is allocated, and scanning generational
garbage collection algorithm is ran on it -- it takes "roots" -- all known live
values on stack and in registers, moves them to a new heap, then repeatedly
scans the parts of from-heap and extracts all the values, referred to by
roots and data which is already moved, and also appends them to the same
to-heap. When this is finished, from-heap is assumed to contain only dead values
and is freed.

This is simplified picture because in reality heap is split into old values that
survived last GC and young (added since last GC) and different placement logic
is applied to them, also there is mature mark for values which survived previous
GC.

Running GC per process heap allows erlang to keep GC latency very low and
doesn't affect other processes on other schedulers. This is not an easy topic,
but theory is all here: http://gchandbook.org/