.. BEAM VM Wisdoms documentation master file, created by
   sphinx-quickstart on Sat Aug 27 14:47:51 2016.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome, adventurer!
====================

This is the collection of easy to read (ELI5) articles as well as in-depth
knowledge such as VM internals, memory layout, opcodes etc.
The project is work in progress, so come back soon!
Github repository for pages https://github.com/kvakvs/beam-wisdoms

ELI5 Section (Explain Me Like I'm Five)
---------------------------------------

.. toctree::
    :maxdepth: 2

    eli5-vm
    eli5-atoms
    eli5-processes
    eli5-process-heap
    eli5-io
    eli5-bif-nif
    eli5-types
    eli5-etf

.. todo::
    ELI5 scheduling (extend);
    ELI5 IO (extend);
    ELI5 Ext term format;
    ELI5 dirty schedulers

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

.. todo::
    More details from ELI5 articles;
    IO and ports;
    Links and monitors;
    ETS?;
    BIFs, traps;
    Process: Exceptions;
    Ext term format
