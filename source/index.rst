.. The Biological Repository (BioR) for Genomic Annotation documentation master file, created by
   sphinx-quickstart on Thu Apr 19 19:25:20 2018.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

===================================================
Biological Repository (BioR) for Genomic Annotation
===================================================

Overview:
---------
.. _toolkit-docs:

BioR is light-weight annotation distribution infrastructure. It is a general purpose genomic data integration
tool that enables genomic coordinate based or accession lookup searches. This user guide will help get you up to speed in how
to use BioR. BioR utilizes catalogs, which are the data files containing the data of interest. Please see :doc:`toolkit/install`
to get this tool for yourself.


Catalogs:
---------

 A BioR catalog is basically a BED-JSON hybrid file that is
indexed using Tabix for coordinate based search and BioR’s own indexing
system for string matching based searches.

.. toctree::
   :maxdepth: 2
   :glob:

   catalog/*



Toolkit:
   BioR uses a
`Pipe-And-Filter <http://www.dossier-andreas.net/software_architecture/pipe_and_filter.html>`__
architecture. Data to be annotated by BioR is streamed through a
pipeline, a sequence of one or more pipes. Pipes is based on Flow Based
Programming by J.P. Morrison.
`DataFlow-Article <http://www.drdobbs.com/database/dataflow-programming-handling-huge-data/231400148?pgno=2>`__,
`Flow-Based-Programing <http://www.amazon.com/Flow-Based-Programming-2nd-Application-Development/dp/1451542321/>`__.

.. image:: `images/biortools_fig01.png`

**Figure 1: BioRTools works by adding annotation to the right on the
original file.**

BioR leverages UNIX pipes to flow data from program to program. As BioR
programs work on the data, they place annotation to the right (the red,
blue and green columns in Figure 1).

.. toctree::
   :maxdepth: 2
   :glob:

   toolkit/commands/*