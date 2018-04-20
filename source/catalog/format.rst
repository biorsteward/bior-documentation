
Catalog Format Changes
----------------------

The catalogs have changed formats slightly, though the toolkit should be
compatible with most so far (except possibly for backwards compatibility
with single-column catalogs). Some changes relate to the additional
files that are included in the catalog directory

.. _section-1:

1.0.0 (2012-12)
~~~~~~~~~~~~~~~

-  Initial format bgzip file with 4 columns:

.. _section-2:

1.1.0 (2014?)
~~~~~~~~~~~~~

-  Included datasource.properties and columns.tsv

-  Directory structure should be:

   -  {SOURCE}/{VERSION}_{ASSEMBLY}[.p{PATCH}]/{TYPE}[_NODUPES].v{VERSION}

   -  Ex: /data5/bsi/catalogs/bior/v1/ESP/V2_GRCh38/variants_nodups.v1/

-  Includes blacklist files for columns to ignore for wrapper apps like
   biorweb

-  Chromosomes should conform to those in the human chromosome list and
   in sorted order

-  Chromosome should be "UNKNOWN" when not known or not applicable, and
   min,max should be 0,1. I think we needed to have at least one
   non-zero position. Previous was dots or (. 0 0) in the cols like:

   -  /data5/bsi/catalogs/bior/v1/omim/2013_02_27/genemap_GRCh37.tsv.bgz

-  Dataset property added in datasource.properties

-  Can have a single column for JSON in the catalog

-  Added HumanReadableName to columns.tsv. By default, this is set to
   the same as the key

.. _section-3:

1.1.1 (2017)
~~~~~~~~~~~~

-  Reflects the new automation format

-  Will include markdown files

-  Will include build files from automation, as well as stats (in build/
   subdirectory)
