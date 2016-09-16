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
    ``file_pos += ALIGN * ((chunk_length + ALIGN - 1) / ALIGN);``

"Atom" - The Atoms Table
````````````````````````

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
*   Read U32/big arity, U32/big label

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

Encodes imported functions in the ``-import([]).`` attribute.

* Read U32/big count

Until the ``count`` do:

*   Read U32/big module atom index, U32/big function atom index, U32/big arity

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

.. _beam-term-format:

BEAM Compact Term Encoding
--------------------------

BEAM file uses a special encoding to store simple terms in BEAM file in
a space-efficient way.
It is different from memory term layout, used by BEAM VM.
It uses first 3 bits of a first byte as a tag to specify the type of the
following value.
If the bits were all 1 (decimal 7), then few more bits are used.

Parse the value ``tag``:

*   Read a byte and see its first 3 bits, what they are. This is base tag.
    Literal=0, Integer=1, Atom=2, XRegister=3, YRegister=4, Label=5,
    Character=6, Extended=7.
*   If the base tag was Extended=7, the byte>>4 + 7 will become extended tag:
    Float=8, List=9, FloatReg=10, AllocList=11, Literal=12.

`Github read signed word <https://github.com/kvakvs/gluonvm/blob/master/emulator/src/beam_loader.cpp#L513-L533>`_
routine used to read signed words later:

.. _beam-parse-smallint:

`Github parse small integer <https://github.com/kvakvs/gluonvm/blob/master/emulator/src/beam_loader.cpp#L535-L555>`_:
(used to read SmallInt values later).

*   Look into the first byte read, bit #3:

    *  Bit #3 = 1: Look into bit #4:

        *     Bit #4 = 1: Use remaining 3 bits of the byte as byte length
                (if under 7 - read N+2 bytes into signed words,
                if it is 7 - then length is larger than that and we have to
                read length first - INFORMATION INCOMPLETE)
        *     Bit #4 = 0: use remaining 3 bits + 8 more bits of the following byte

    *  Bit #3 = 0: Use remaining 4 bits

Now how to parse an encoded term:

*   Read a SmallInt, case ``tag`` of:

    *   Tag=Integer or Literal: use smallint value.
    *   Tag=Atom: use smallint value MINUS 1 as index in the atom table.
        0 smallint means NIL (empty list).
    *   Tag=Label: use as label index, or 0 means invalid value.
    *   Tag=XRegister, Tag=YRegister: use as register index.
    *   Tag=Character (an Unicode symbol): use val as unsigned.
    *   Tag=Extended List: create tuple of size smallint. For smallint/2 do: parse
        a term (``case of`` value), parse a small int (label index), place them
        into the tuple.

.. _beam-code-format:

BEAM Code Section Format
------------------------

Code section in BEAM file contains list of instructions and arguments.
To read an encoded term see :ref:`BEAM Term format <beam-term-format>`.

*   Read a byte, this is opcode (R19 has 158 base opcodes).
    Opcode is converted into a label address (for threaded interpreter) or
    a pointer to handler function.
*   Query opcode table and get arity for this opcode.
*   Until ``arity``: parse term and put it into the output
*   If any of the parsed terms was a label value, remember its output position
    to later revisit it and overwrite with actual label address in memory
    (it is not known until code parsing is done).
