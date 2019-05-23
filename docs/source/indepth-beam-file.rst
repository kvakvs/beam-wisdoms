BEAM File Format
================

BEAM file format is binary chunked file format, which contains multiple named
sections and has a header.

*   Read 4 bytes of a .beam file: ``'FOR1'``
    (marks `IFF container <https://en.wikipedia.org/wiki/Interchange_File_Format>`_).
*   Read U32/big length (so many more bytes must be available in the file)
*   Read 4 bytes ``'BEAM'`` marking a BEAM file section of an IFF file.

Sections
--------

Repeat until end of file:

*   Read 4 bytes chunk name (for example can be: Atom, Code, CatT, FunT, ExpT,
    LitT, StrT, ImpT, Line, Abst and possibly other chunks).
    Do a sanity check: Must contain ASCII letters.
*   Read U32/big chunk length. This is the data size.
*   Perform chunk-dependent reading (see subsections below)
*   To find next chunk, pad the length to the multiple of ALIGN=4

    .. code-block:: c

        file_pos += ALIGN * ((chunk_length + ALIGN - 1) / ALIGN);

"Atom" and "AtU8" - Atoms Table
```````````````````````````````

Both tables have same format and same limitations (256 bytes max length)
except that bytes in strings are treated either as latin1 or utf8.

*   Read U32/big atoms count.
*   For each atom: read byte length, followed by characters

Atoms[0] is a module name from ``-module(M).`` attribute.

"Code" - Compiled Bytecode
``````````````````````````

*   Read U32/big code version (must match emulator's own version)
*   Read U32/big max opcode, U32/big label count, U32/big fun count
*   Read the code as a block. Format is discussed at :ref:`beam-code-format`.

"Abst" - Abstract Syntax Tree
`````````````````````````````

Optional section which contains ``term_to_binary`` encoded AST tree.

A quick way to get ``Abst`` section (if it exists):

.. code-block:: erlang

    get_abst(Filename) ->
        Chunks = beam_lib:all_chunks(Filename),
        Abst = proplists:get_value("Abst", element(3, Chunks)),
        binary_to_term(Abst).

"CatT" - Catch Table
````````````````````

Contains catch labels nicely lined up and marking try/catch blocks.
This section description is INCOMPLETE and UNTESTED.

*   Read U32/big count
*   Read array of ``count`` U32/big offsets or labels (not sure).

"FunT" - Function/Lambda Table
``````````````````````````````

Contains pointers to functions in the module.

*   Read U32/big count

Until the ``count`` do:

*   Read U32/big fun_atom_index (name by index from atom table),
    U32/big arity,
    U32/big offset (code position),
    U32/big index,
    U32/big nfree (frozen values for closures),
    U32/big ouniq. Sanity check: fun_Atom_index must be in atom table range.

"ExpT" - Exports Table
``````````````````````

Encodes exported functions and arity in the ``-export([]).`` attribute.

*   Read U32/big count

Until the ``count`` do:

*   Read U32/big export name atom index. Sanity check: atom table range.
*   Read U32/big arity, U32/big label (offset in BEAM code section, should
    be translated into the loaded code offset).

"LitT" - Literals Table
```````````````````````

Contains all the constants in file which are larger than 1 machine Word.
It is compressed using zip Deflate.

*   Read U32/big uncompressed size (prepare output buffer of this size). Run
    zip inflate (uncompress) on the data.

Inside the uncompressed data:

*   Read U32/big value count

Until the ``value count`` do:

*   Skip U32/big
*   Read byte ext term format marker (must be 131)
*   Read tag byte, ... (follow the documentation)

Values are encoded using the external term format.
A better reference is in the
`standard documentation <http://erlang.org/doc/apps/erts/erl_ext_dist.html>`_

"ImpT" - Imports Table
``````````````````````

Encodes functions from other modules invoked by the current module.

* Read U32/big count

Until the ``count`` do:

*   Read U32/big module atom index, U32/big function atom index, U32/big arity

"LocT" - Local Functions
````````````````````````

Essentially same as the export table format ``ExpT`` for local functions.

* Read U32/big count

Until the ``count`` do:

*   Read U32/big func atom index, U32/big arity, U32/big location (label)

"Line" - Line Numbers Table
```````````````````````````

Encodes line numbers mapping to give better error reporting and code navigation
for the program user.

*   Read U32/big version (must match emulator's own version 0).
*   Skip U32/big flags
*   Read U32/big line_instr_count, U32/big num_line_refs, U32/big num_filenames
*   Store invalid location const as Word[] linerefs first element which points
    at file #0, line 0.
*   Set fname_index = 0, this is index in file name table, empty now

Until the ``num_line_refs`` do:

*   Parse term at read position (see :ref:`BEAM Term format <beam-term-format>`)
*   If the term is a small integer, push a pair of (fname_index, value) to
    the linerefs array.
*   If the term is an atom, use its numeric value as new fname_index. Sanity
    check: value must be under ``num_filenames``.

Until the ``num_filenames`` do (fill the file names table):

*   Read U16/big name size
*   Read string of bytes
*   Convert string to an atom and push into file names table

"StrT" - Strings Table
``````````````````````

What I've been able to see from the compiler source, is that this is a huge
binary with all concatenated strings from the Erlang parsed AST (syntax tree).
Everything ``{string, X}`` goes here. There are no size markers or separators
between strings, so this part is confusing and requires more code reading.

Consider ``compiler`` application in standard library, files:
``beam_asm``, ``beam_dict`` (record ``#asm{}`` field ``strings``), and
``beam_disasm``.

"Attr" - Attributes
```````````````````

Contains two parts: a proplist of module attributes, encoded as External Term
Format, and a compiler info (options and version) encoded similarly.


.. _beam-term-format:

BEAM Compact Term Encoding
--------------------------

BEAM file uses a special encoding to store simple terms in BEAM file in
a space-efficient way.
It is different from memory term layout, used by BEAM VM.

The idea is to stick as many type and value data into the 1st byte as possible::

    7 6 5 4 3 | 2 1 0
    ----------+------
              | 0 0 0 — Literal
              | 0 0 1 — Integer
              | 0 1 0 — Atom
              | 0 1 1 — X Register
              | 1 0 0 — Y Register
              | 1 0 1 — Label
              | 1 1 0 — Character
    0 0 0 1 0 | 1 1 1 — Extended — Float
    0 0 1 0 0 | 1 1 1 — Extended — List
    0 0 1 1 0 | 1 1 1 — Extended — Floating point register
    0 1 0 0 0 | 1 1 1 — Extended — Allocation list
    0 1 0 1 0 | 1 1 1 — Extended — Literal

.. note::

    In OTP 20 the Floats are encoded as literals, and every other extended code
    is shifted, i.e. List becomes 1 (0b10111), Float register becomes 2 (0b100111),
    alloc list becomes 3 (0b110111) and literal becomes 4 (0b1000111).

It uses first 3 bits of a first byte as a tag to specify the type of the
following value.
If the bits were all 1 (special value 7), then few more bits are used.

For values under 16 the value is placed entirely into bits 4-5-6-7 having bit
3 set to 0::

    7 6 5 4 | 3 | 2 1 0
    --------+---+------
    Value>> | 0 | Tag>>

For values under 16#800 (2048) bit 3 is set to 1, marks that 1 continuation
byte will be used and 3 most significant bits of the value will extend into
this byte's bits 5-6-7::

    7 6 5 | 4 3 | 2 1 0
    ------+-----+------
    Value | 0 1 | Tag>>

Larger and negative values are first converted to bytes.
Then if the value takes 2..8 bytes, bits 3-4 will be set to 1, and bits
5-6-7 will contain the ``(Bytes-2)`` size for the value, which follows::

    7  6  5 | 4 3 | 2 1 0
    --------+-----+------
    Bytes-2 | 1 1 | Tag>>

If the following value is greater than 8 bytes, then all bits 3-4-5-6-7
will be set to 1, followed by a nested encoded unsigned ``?tag_u`` value
of ``(Bytes-9):8``, and then the data::

    7 6 5 4 3 | 2 1 0
    ----------+------ Followed by nested encoded int (Size-9)
    1 1 1 1 1 | Tag>>

.. seealso ::
    Refer to ``beam_asm:encode/2`` in the ``compiler`` application for
    details about how this is encoded. Tag values are presented in this
    section, but also can be found in ``compiler/src/beam_opcodes.hrl``.

Base and Extended Tag
`````````````````````

Let's parse the value of ``tag``:

*   Read a byte and extract its least 3 bits. This is the base tag.
    It can be Literal=0, Integer=1, Atom=2, XRegister=3, YRegister=4, Label=5,
    Character=6, Extended=7.
*   If the base tag was Extended=7, then bits 4-5-6-7 PLUS 7 will become
    the extended tag. It can have values
    Float=8, List=9, FloatReg=10, AllocList=11, Literal=12.

A badly written and incomplete
`Github example of reading signed word <https://github.com/kvakvs/gluonvm1/blob/master/emulator/src/beam_loader.cpp#L513-L533>`_
routine used to read signed words later:

.. _beam-parse-smallint:

A badly written and incomplete
`Github example of parsing a small integer <https://github.com/kvakvs/gluonvm1/blob/master/emulator/src/beam_loader.cpp#L535-L555>`_:
(used to read SmallInt values later).

Reading the Value
`````````````````

This is the logic, as was decoded from source code of BEAM VM and Ling VM.
It looks at the bits in slightly different order.

*   Look into the first byte read, bit 3:

    *  Bit 3 is 1, so look into bit 4:

        *     Bit is 1: Use remaining 3 bits of the byte as byte length
                (if under 7 - read ``N+2`` bytes into signed words,
                if the value is 7 - then length is larger than that and we
                have to read length first -- it follows as ``?tag_u=0``
                (Literal) nested unsigned value)
        *     Bit 4 is 0: use remaining 3 bits + 8 more bits of the following byte

    *  Bit #3 = 0: Use remaining 4 bits

Now how to parse an encoded term:

*   Read a SmallInt, case ``tag`` of:

    *   Tag=Integer: use the value (signed?)
    *   Tag=Literal: use smallint value as index in ``LitT`` table.
    *   Tag=Atom: use smallint value MINUS 1 as index in the atom table.
        0 smallint means ``NIL []``.
    *   Tag=Label: use as label index, or 0 means invalid value.
    *   Tag=XRegister, Tag=YRegister: use as register index.
    *   Tag=Character (an Unicode symbol): use val as unsigned.
    *   Tag=Extended List: contains pairs of terms.
        Read smallint ``Size``. Create tuple of ``Size``, which will contain
        ``Size/2`` values.
        For ``Size/2`` do:
        read and parse a term (``case of`` value),
        read a small int (label index), place them into the tuple.

.. _beam-code-format:

BEAM Code Section Format
------------------------

Code section in BEAM file contains list of instructions and arguments.
To read an encoded term see :ref:`BEAM Term format <beam-term-format>`.

*   Read a byte, this is opcode (R19 has 158 base opcodes).
    Opcode is converted into a label address (for threaded interpreter) or
    a pointer to handler function.
*   Query opcode table and get arity for this opcode.
*   Until ``arity``: parse term and put it into the output one term or word at
    a time. VM loop will read the opcode later and expect that ``arity``
    args will follow it.
*   If any of the parsed terms was a label value, remember its output position
    to later revisit it and overwrite with actual label address in memory
    (it is not known until code parsing is done).
