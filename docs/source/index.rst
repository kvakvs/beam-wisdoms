.. BEAM VM Wisdoms documentation master file

Welcome, adventurer!
====================

This is the collection of easy to read (ELI5) articles as well as in-depth
knowledge such as VM internals, memory layout, opcodes etc.
The project is work in progress, so come back soon!
Github repository for pages https://github.com/kvakvs/beam-wisdoms

Latest
``````

*   2018-10-21  ELI5 for memory performance on BEAM
*   2018-01-13  ELI5 Spawning a process (actor), amended ELI5 copying, Pyrlang added to interfacing
*   2017-10-01  BEAM format: Added UTF8 atoms section, LocT, Attr section
*   2017-03-17  Data copying ELI5, efficiency guide

ELI5 Section (Explain Me Like I'm Five)
---------------------------------------

.. toctree::
    :maxdepth: 1

    eli5-vm
    eli5-atoms
    eli5-processes
    eli5-process-heap
    eli5-io
    eli5-tracing
    eli5-bif-nif
    eli5-types
    eli5-etf
    eli5-property-based
    eli5-efficiency
    eli5-efficiency-memory-perf


Deeper Knowledge Section
------------------------

.. toctree::
    :maxdepth: 2

    definitions
    indepth-memory-layout
    indepth-data-sizes
    indepth-heap-layout
    interfacing
    indepth-beam-file
    indepth-beam-instructions
    indepth-io

.. todo::
    More details from ELI5 articles;
    IO and ports;
    Links and monitors;
    ETS?;
    BIFs, traps;
    Process: Exceptions;
    Ext term format

Smaller things you would've never noticed otherwise. OTP C source-related.

:doc:`otp-c-style-guide`, :doc:`otp-cpp-ramblings`, :doc:`otp-c-refactor-todo`
