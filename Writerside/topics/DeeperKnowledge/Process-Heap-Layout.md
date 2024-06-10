# In Depth: Process Heap Layout

An Erlang process has a young heap (where all new data is placed), old heap (possibly this one does not exist, contains
data which survived previous garbage collection) and a stack, which is located inside young heap.

Also, a process has access to read-only literal area, which belongs to one or more of Erlang modules, it contains
constant literals which are found in module code. At some rare occasions (code upgrade or purge) literal area can be
copied to old heap by means of garbage-collection (see `erts_garbage_collect_literals` in `erl_gc.c`).

A process has chain of Off-heap objects (so-called MSO -- mark and sweep object list). The head of the chain is a field
in `Process` struct, and the chain continues through the heap visiting [](BEAM-Definitions.md#header-tag) objects such
as refc binaries and fun closures. During garbage collection this chain is followed and actions can be taken (similar to
C++ destructors) when an MSO object is going to die -- for example refc binaries have their refcount reduced and
possibly are freed. MSO objects are strictly sorted from new to old, by the order of their creation on heap.

## How Heap Grows

Additionally, heap segments can exist (for situations, when heap is not enough, but data must be allocated). Heap
segments are consolidated before garbage collection and merged onto heap.

Heap top (`heap_top`) marks current allocation position in heap. When more data is needed, `heap_top` is advanced
forward and data goes there.

Heap always grows forward from `heap_start` (see `erl_process.h`, `Process` struct, fields `heap`, `old_heap`). Stack
always grows backwards from `heap_end` to `heap_top`, as soon as stack meets heap top, the young heap is considered full
and garbage collection is triggered.

Old heap does not contain a stack. Old heap can only grow during garbage collection and its future size is
precalculated.

To minimize memory fragmentation, heap sizes like a puzzle pieces are allocated
following Fibonacci sequence (starting from 12 and 38 [Words](BEAM-Definitions.md#word))
for 23 total steps, after which (at about 1.3 million [Words](BEAM-Definitions.md#word))
sequence continues by adding 20% to the previous value.
