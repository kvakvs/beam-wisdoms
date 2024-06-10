# In Depth: Data Types Memory Layout

This describes data format on heap, as seen in C code or from debugger.
You will never be able to see this format from Erlang code.

## Immediate values

![](memoryLayout_immed.svg)

Immediate types always occupy 1 [](BEAM-Definitions.md#word). To know if you found an immediate, its least-significant 2
bits will have value `TAG_PRIMARY_IMMED1=3`.

To know which exactly immediate you've got, see the two following bits (bit 2 and 3):

* `_TAG_IMMED1_PID=0`,
* `_TAG_IMMED1_PORT=1`,
* `_TAG_IMMED1_SMALL=3`.

This leaves remaining [](BEAM-Definitions.md#word) minus 4 bits for the actual value.

If the bits 2 and 3 contained `_TAG_IMMED1_IMMED2=2` then two more bits (bit 4 and 5) are taken and interpreted. They
can be:

* `_TAG_IMMED2_ATOM=0`,
* `_TAG_IMMED2_CATCH=1`
* `_TAG_IMMED2_NIL=3`,

which leaves remaining [Word-size](BEAM-Definitions.md#word) minus 6 bits for the actual value.

This also explains why max small integer range is `32-4=28`: `2^27-1` + one bit sign, because small is an immediate-1
value. And why max number of atoms can be `32-6=26` (2^26=32 million), because atom is an immediate-2 value. For
compatibility reasons this limit also applies to 64-bit systems.

## Lists (Cons)

![](memoryLayout_listCell.svg)

A list term is boxed value (i.e. contains a pointer to heap). 2 least-significant bits of list value
have `TAG_PRIMARY_LIST=1`, remaining bits are the pointer.

A value on heap contains 2 [Words](BEAM-Definitions.md#word) -- namely CAR (or list head) and CDR (list tail) (see `CAR`
and `CDR` macros in `emulator/beam/erl_term.h`). This pair of words is called "Cons Cell" (terminology from Lisp and
functional programming). Cons cell has no header word stored in memory.

Each cons cell contains a pointer to next cell in CDR (tail). As this structure is also visible from Erlang, last cons
cell of a list contains `NIL` (represented by a special value for empty list `[]`) or a
non-list [](BEAM-Definitions.md#term) value (this makes an *improper* list).

This structure may look inefficient, but on the other hand it allows connecting any tail of any list to multiple cons
cells to reuse existing data. Also list cells do not require a header word, they are stored in memory as a pair of
words.

## Boxed

![](memoryLayout_box.svg)

Boxed value is a pointer with 2 least-significant bits tagged with `TAG_PRIMARY_BOXED=2`. Remaining bits are the
pointer.

A boxed pointer must always point to a [](BEAM-Definitions.md#header-tag) (see explanation of headers below). Boxed
values can be found everywhere: in registers, on stack, on heaps.

Box always points at a Header (below). During the garbage collection a Box can point to another Box or
to `THE_NON_VALUE` to mark a moved object, but this can never happen after the collection.

## Headers (Prefix for Any Other Data)

Header tag is placed on any boxed value on heap, also on temporary blocks used by internal emulator logic, they will be
automatically garbage collected later.

Header values can never be found in register or on stack. This is heap-only data structure.

### Tuple (ARITYVAL=0)

![](memoryLayout_tuple.svg)

A tuple has header word tagged with `TAG_PRIMARY_HEADER` with `ARITYVAL_SUBTAG`. Remaining bits in header word represent
tuple arity (see `arityval` and `make_arityval` macros).

Following are tuple elements. This explains, why tuple is very easy to access at arbitrary index, and very hard to grow.
Modification of tuple elements in place is used as optimization by Erlang compiler if it can prove, that intermediate
tuple value will be dropped.

### Bignum, A Big Integer (NEG=2/POS_BIG=3)

![](memoryLayout_bignum.svg)

Bignums have header word tagged with `TAG_PRIMARY_HEADER` followed by either `POS_BIG_SUBTAG` or `NEG_BIG_SUBTAG`.
Remaining bits in header word are arity, i.e. how many extra [Words](BEAM-Definitions.md#word) are used by bignum bits.

Following are bits of the bignum, a [Word](BEAM-Definitions.md#word) at a time. Most significant word goes first.

### Reference (REF=4)

![](memoryLayout_ref.svg)

See struct `RefThing` in `emulator/beam/erl_term.h`. Contains header word tagged with `TAG_PRIMARY_HEADER`
with `REF_SUBTAG` which also matches the first field of `RefThing`.

Following are other `RefThing` fields (3 32-bit words or 2 64-bit words) which have the ref value stored in them.
Internal (local) ref layout is explained in `emulator/beam/erl_term.h` search for text "Ref layout (internal
references)" and "Ref layout on a 64-bit" (2 comments).

### Fun/Closure (FUN=5)

See struct `ErlFunThing` in `erl_fun.h`. Contains header word tagged with `TAG_PRIMARY_HEADER` with `FUN_SUBTAG` which
also matches the first field of `ErlFunThing`.

This is a closure (a function pointer with frozen variable values). It contains pointer to function entry, arity, amount
of frozen variables, pid of creator process and array of frozen variables.

### Float (FLOAT=6)

Contains header word tagged with `TAG_PRIMARY_HEADER` with `FLOAT_SUBTAG`. Followed by 64 bit of C `double` IEEE-754
format.

### Export (EXPORT=7)

![](memoryLayout_export.svg)

Refers to a `{Mod, Fun, Arity}` triple. Contains a pointer to the *export table*. Always has "arity" bits set 1 (because
contains only one pointer), this is not the same as the `Arity` part of the `m:f/a` triple.

A record in export table contains:

* Pointers to all (old and current) versions of the code for the function
* 2 words with `func_info` opcode for the function. Note, that this is executable BEAM code.
* 3 words: Module (atom), Function (atom), Arity (as untagged integer)
* 1 word which is 0 or may contain a apply, call or breakpoint opcode. Note, that this is executable BEAM code.
* 1 word argument for the previous opcode. This may be a pointer to BEAM code, a pointer to a C BIF function or 0.

> See Also: `Export` struct in `emulator/beam/export.h`

### Reference-counted Binary (REFC_BINARY=8)

A pointer to binary on the binary heap. When this is destroyed, the reference count is reduced (can only happen during
GC). Reference to refc binary is called procbin. A refc binary whose refc reaches 0 is deleted.

### Heap Binary (HEAP_BINARY=9)

A smaller binary (under 64 bytes) which is directly placed on the process heap.

### Sub-binary (SUB_BINARY=10)

A temporary pointer to a part of another binary (either heap or refcounted, but never into another sub-binary). Can be
produced by `split_binary` function. Refers to a larger binary and holds a reference to it.

### Match context

Similar to sub-binary but is created in the process of binary matching. Refers to a larger binary and holds a reference
to it.

### External Pid 12

Pid containing node name. Refers to a process located on another node.

### External Port 13

Port located on another node and the data contains the node name.

### External Ref (EXTERNAL_REF=14)

Refers to a reference created by another node. Good as any other reference. External ref format is explained
in `erl_term.h` search for "External thing layout".
