BEAM Internal data sizes
========================

Each boxed term has size described below plus one word for term value
itself which you store somewhere else (be it on heap then you should count size
for this term too, or in register/on stack - then it will not consume heap
memory).

For specific bit positions and bit sizes please always refer to original beam
sources, file: ``erl_term.h``

Immediate
---------

All immediate values have size of one :ref:`Word <word-label>`.

They are: local **pid**, local **port**, **small integer**, **atom**, **NIL**
(an empty list ``[]``) and catch (internal value). :ref:`CP <cp-label>` can
also be considered an immediate although it is a special value which only
appears on stack and never in registers or heap.

List
-----

List value is a pointer to cons cell. Cons cell has size of two
:ref:`Words <word-label>` (one for head, and another for tail). Cons cell can
link to next cons cell and so on, until tail becomes a non-list term (for improper
lists) or ``NIL`` (for proper lists).

Size of a list in memory will be ``2*num_cells`` :ref:`Words <word-label>`.

Boxed
-----

Boxed values typically have an `arity`, which is stored in most-significant bits
of their first word, following the header tag.

They are: arityval (a **tuple**),
**fun** (closure with frozen variable values),
positive and negative **bigint** (tagged separately to avoid storing sign bit),
**float**, **export**, **refvalue**, **refcounted binary**, **heap binary**,
**subbinary**, external **pid**, **port**, **ref**,
**matchstate** (internal object) and **map**.

Tuple size: its arity plus 1 :ref:`Word <word-label>` header.

Float size: always 64 bit (1 or 2 words) + 1 :ref:`Word <word-label>` header.

Bigint size: ``ceil(log2^64(Number))`` Words + 1 :ref:`Word <word-label>`
header. Thus a <=64bit will take 1 word, 65-127bit will take 2 words and so on.
On 32-bit architectures, of course, the Word size is 32 bit and everything is
32-bit based.


Header
------

Header is a special tag which hides all sorts of internal opaque data. Header
can be found on heap only and remaining most-significant bits usually represent
its size (arity). For maps, arity is calculated using different formula (see
``MAP_HEADER_ARITY`` macro).