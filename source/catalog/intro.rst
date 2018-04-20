

BioR Catalog Shortcut
---------------------

BioR commands commonly use long paths to files. One of the first things
you will want to do when using BioR is to make an alias to the location
of the BioR catalogs. For example if the BioR catalogs are located in
$bior

Then, on bash, execute the following command at the command line:

+---------------------------+
| $ export bior=/data/path/ |
+---------------------------+

You may want to put this command in your .bashrc or .bash_profile so
that the $bior environment variable shows up next time you log in.

Finding out what is in a Catalog
--------------------------------

Each data source is 'published' into a BioR catalog file for use by the
BioR scripts.  A Catalog is a collection of files (both data and
indexes) that is understood by the BioR Pipes infrastructure. BioR's
reference data consists of the raw files downloaded/updated and made
available to BioR users. These files ARE NOT catalogs. Catalogs are
transformed into the BioR standard catalog structure so that pipes can
work on the content. BioR catalogs are bgziped files [1]_ that contain 4
columns (_landmark, \_minBP, \_maxBP, and JSON). A more comprehensive
description of the BioR catalog format is in Chapter 3.

To see what is in a catalog, use the zcat command (gzcat on a mac)
followed by the catalog filename, followed by less:

+-----------------------------------------------------------------------+
| $ zcat $bior/NCBIGene/GRCh37_p10/genes.tsv.bgz \| less                |
|                                                                       |
| 1   10954   11507                                                     |
| {"_type":"gene","_landmark":"1","_strand":"+","_minBP":10954,"_maxBP" |
| :11507,"gene":"LOC100506145","note":"Derived                          |
| by automated computational analysis using gene prediction method:     |
| GNOMON. Supporting evidence includes similarity to: 1                 |
| Protein","pseudo":"","GeneID":"100506145"}                            |
|                                                                       |
| ...                                                                   |
+-----------------------------------------------------------------------+

Unix less is a good-low-memory command to look at data. Type q <enter>
to quit less. Type man less at the command line to see how to use the
less command. You can use up and down arrows to scroll through the data
a line at a time or ‘f’ and ‘b’ to scroll a page at a time.

Several of the above functions use ‘Golden Identifiers’ to match records
across catalogs. Table 2 shows the current golden identifiers used in
the codebase and what function(s) use them. The bold ones are the
primary attributes used by several commands

+-----------------------+-----------------------+-----------------------+
| **‘Golden             | **Functions**         | **Definition**        |
| Identifier’**         |                       |                       |
+-----------------------+-----------------------+-----------------------+
| **\__landmark**       | bior_overlap,         | Chromosome, or        |
|                       | bior_same_variant     | sequence ID that the  |
|                       |                       | interval is located   |
|                       |                       | on                    |
+-----------------------+-----------------------+-----------------------+
| **\__minBP**          | bior_overlap,         | Minimum 1-based       |
|                       | bior_same_variant     | position (e.g. NCBI   |
|                       |                       | coordinates) on the   |
|                       |                       | landmark sequence     |
+-----------------------+-----------------------+-----------------------+
| **\_maxBP**           | bior_overlap,         | Maximum 1-based       |
|                       | bior_same_variant     | position on the       |
|                       |                       | landmark sequence     |
+-----------------------+-----------------------+-----------------------+
| **\_refAllele**       | bior_same_variant     | REF as in VCF         |
|                       |                       | standard              |
+-----------------------+-----------------------+-----------------------+
| **\_altAlleles**      | bior_same_variant     | ALT as in VCF         |
|                       |                       | standard              |
+-----------------------+-----------------------+-----------------------+
| \_type                | (none yet)            | The type of object    |
|                       |                       | each line refers to   |
|                       |                       | (Examples: “variant”, |
|                       |                       | “gene”, “drug”,       |
|                       |                       | “pathway”, etc)       |
+-----------------------+-----------------------+-----------------------+
| \_strand              | (none yet)            | The strand direction  |
|                       |                       | (“+” or “-”)          |
+-----------------------+-----------------------+-----------------------+
| \_id                  | (none yet)            | The id for the        |
|                       |                       | variant or object.    |
|                       |                       | For example, this may |
|                       |                       | refer to the rsId for |
|                       |                       | variants              |
+-----------------------+-----------------------+-----------------------+







BioR Commands
=============

Showin the Commands in BioR Toolkit
-----------------------------------

All BioR commands start with bior\_ so once BioRTools is installed and
on your path you can type bior\_ followed by the tab key (twice) and it
will show you all of the current commands in the toolkit:

+-----------------------------------------------------------------------+
| $ bior\_                                                              |
|                                                                       |
| bior_annotate bior_concat bior_lookup bior_snpeff                     |
|                                                                       |
| bior_annotate_blaster bior_count_catalog_misses bior_merge            |
| bior_tab_to_tjson                                                     |
|                                                                       |
| bior_annotate.sh bior_create_catalog bior_modify_tjson                |
| bior_tjson_to_vcf                                                     |
|                                                                       |
| bior_bed_to_tjson bior_create_catalog_props bior_overlap              |
| bior_trim_spaces                                                      |
|                                                                       |
| bior_build_catalog bior_create_config_for_tab_to_tjson                |
| bior_pretty_print                                                     |
|                                                                       |
| bior_variant_to_tjson bior_catalog_remove_duplicates bior_drill       |
|                                                                       |
| bior_ref_allele bior_vcf_to_tjson bior_catalog_stats                  |
| bior_gbk_to_tjson                                                     |
|                                                                       |
| bior_replace_lines bior_vep bior_chunk bior_gff3_to_tjson             |
|                                                                       |
| bior_rsid_to_tjson bior_verify_catalog bior_compress                  |
| bior_index_catalog                                                    |
|                                                                       |
| bior_same_variant bior_create_catalog_props bior_ref_allele           |
+-----------------------------------------------------------------------+
|                                                                       |
+-----------------------------------------------------------------------+

Or listing them out one command per line in sorted order:

+---------------------------------------------+
| $ ls -1 $BIOR_LITE_HOME/bin \| grep ^bior\_ |
|                                             |
| bior_annotate                               |
|                                             |
| bior_annotate_blaster                       |
|                                             |
| bior_bed_to_tjson                           |
|                                             |
| bior_build_catalog                          |
|                                             |
| bior_catalog_remove_duplicates              |
|                                             |
| bior_catalog_stats                          |
|                                             |
| bior_chunk                                  |
|                                             |
| bior_compress                               |
|                                             |
| bior_concat                                 |
|                                             |
| bior_count_catalog_misses                   |
|                                             |
| bior_create_catalog                         |
|                                             |
| bior_create_catalog_props                   |
|                                             |
| bior_create_config_for_tab_to_tjson         |
|                                             |
| bior_drill                                  |
|                                             |
| bior_gbk_to_tjson                           |
|                                             |
| bior_gff3_to_tjson                          |
|                                             |
| bior_index_catalog                          |
|                                             |
| bior_lookup                                 |
|                                             |
| bior_merge                                  |
|                                             |
| bior_modify_tjson                           |
|                                             |
| bior_overlap                                |
|                                             |
| bior_pretty_print                           |
|                                             |
| bior_ref_allele                             |
|                                             |
| bior_replace_lines                          |
|                                             |
| bior_rsid_to_tjson                          |
|                                             |
| bior_same_variant                           |
|                                             |
| bior_snpeff                                 |
|                                             |
| bior_tab_to_tjson                           |
|                                             |
| bior_tjson_to_vcf                           |
|                                             |
| bior_trim_spaces                            |
|                                             |
| bior_variant_to_tjson                       |
|                                             |
| bior_vcf_to_tjson                           |
|                                             |
| bior_vep                                    |
|                                             |
| bior_verify_catalog                         |
+---------------------------------------------+

To find out which version each command was added to BioR:

+-----------------------------------------------------------------------+
| **# Path to cmds is similar to:**                                     |
|                                                                       |
| **#                                                                   |
| /usr/local/biotools/bior_scripts/4.3.0/bior_pipeline-4.3.0/bin/bior_d |
| rill**                                                                |
|                                                                       |
| **# Sort by cmd**                                                     |
|                                                                       |
| **$ for cmd in \`ls -1 $BIOR_LITE_HOME/bin \| grep ^bior`; do         |
| earliestVersion=`find $BIOR_LITE_HOME/../../ -name $cmd \| sed        |
| 's#^.*bior_pipeline-##' \| sed 's#^\.##' \| sed 's#/.*##' \| sort \|  |
| head -1`; echo -e "$cmd\t$earliestVersion"; done**                    |
|                                                                       |
| bior_annotate 0.0.3-SNAPSHOT                                          |
|                                                                       |
| bior_annotate_blaster 2.3.0                                           |
|                                                                       |
| bior_bed_to_tjson 2.1.0                                               |
|                                                                       |
| bior_build_catalog 4.1.2                                              |
|                                                                       |
| bior_catalog_remove_duplicates 3.0.0                                  |
|                                                                       |
| bior_catalog_stats 4.3.0                                              |
|                                                                       |
| bior_chunk 2.3.0                                                      |
|                                                                       |
| bior_compress 0.0.3-SNAPSHOT                                          |
|                                                                       |
| bior_concat 2.3.0                                                     |
|                                                                       |
| bior_count_catalog_misses 4.1.2                                       |
|                                                                       |
| bior_create_catalog 2.1.0                                             |
|                                                                       |
| bior_create_catalog_props 2.1.0                                       |
|                                                                       |
| bior_create_config_for_tab_to_tjson 2.1.0                             |
|                                                                       |
| bior_drill 0.0.3-SNAPSHOT                                             |
|                                                                       |
| bior_gbk_to_tjson 2.4.0                                               |
|                                                                       |
| bior_gff3_to_tjson 2.4.0                                              |
|                                                                       |
| bior_index_catalog 2.1.0                                              |
|                                                                       |
| bior_lookup 0.0.3-SNAPSHOT                                            |
|                                                                       |
| bior_merge 2.3.0                                                      |
|                                                                       |
| bior_modify_tjson 4.3.0                                               |
|                                                                       |
| bior_overlap 0.0.3-SNAPSHOT                                           |
|                                                                       |
| bior_pretty_print 0.0.3-SNAPSHOT                                      |
|                                                                       |
| bior_ref_allele 2.3.0                                                 |
|                                                                       |
| bior_replace_lines 4.3.0                                              |
|                                                                       |
| bior_rsid_to_tjson 2.4.1                                              |
|                                                                       |
| bior_same_variant 0.0.3-SNAPSHOT                                      |
|                                                                       |
| bior_snpeff 0.0.3-SNAPSHOT                                            |
|                                                                       |
| bior_tab_to_tjson 2.1.0                                               |
|                                                                       |
| bior_tjson_to_vcf 2.1.0                                               |
|                                                                       |
| bior_trim_spaces 2.2.1                                                |
|                                                                       |
| bior_variant_to_tjson 3.0.0                                           |
|                                                                       |
| bior_vcf_to_tjson 2.1.0                                               |
|                                                                       |
| bior_vep 0.0.3-SNAPSHOT                                               |
|                                                                       |
| bior_verify_catalog 4.1.2                                             |
|                                                                       |
| **# Sort by release where each command was introduced**               |
|                                                                       |
| **$ for cmd in \`ls -1 $BIOR_LITE_HOME/bin \| grep ^bior`; do         |
| earliestVersion=`find $BIOR_LITE_HOME/../../ -name $cmd \| sed        |
| 's#^.*bior_pipeline-##' \| sed 's#^\.##' \| sed 's#/.*##' \| sort \|  |
| head -1`; echo -e "$earliestVersion\t$cmd"; done \| sort -k1,1**      |
|                                                                       |
| 0.0.3-SNAPSHOT bior_annotate                                          |
|                                                                       |
| 0.0.3-SNAPSHOT bior_compress                                          |
|                                                                       |
| 0.0.3-SNAPSHOT bior_drill                                             |
|                                                                       |
| 0.0.3-SNAPSHOT bior_lookup                                            |
|                                                                       |
| 0.0.3-SNAPSHOT bior_overlap                                           |
|                                                                       |
| 0.0.3-SNAPSHOT bior_pretty_print                                      |
|                                                                       |
| 0.0.3-SNAPSHOT bior_same_variant                                      |
|                                                                       |
| 0.0.3-SNAPSHOT bior_snpeff                                            |
|                                                                       |
| 0.0.3-SNAPSHOT bior_vep                                               |
|                                                                       |
| 2.1.0 bior_bed_to_tjson                                               |
|                                                                       |
| 2.1.0 bior_create_catalog                                             |
|                                                                       |
| 2.1.0 bior_create_catalog_props                                       |
|                                                                       |
| 2.1.0 bior_create_config_for_tab_to_tjson                             |
|                                                                       |
| 2.1.0 bior_index_catalog                                              |
|                                                                       |
| 2.1.0 bior_tab_to_tjson                                               |
|                                                                       |
| 2.1.0 bior_tjson_to_vcf                                               |
|                                                                       |
| 2.1.0 bior_vcf_to_tjson                                               |
|                                                                       |
| 2.2.1 bior_trim_spaces                                                |
|                                                                       |
| 2.3.0 bior_annotate_blaster                                           |
|                                                                       |
| 2.3.0 bior_chunk                                                      |
|                                                                       |
| 2.3.0 bior_concat                                                     |
|                                                                       |
| 2.3.0 bior_merge                                                      |
|                                                                       |
| 2.3.0 bior_ref_allele                                                 |
|                                                                       |
| 2.4.0 bior_gbk_to_tjson                                               |
|                                                                       |
| 2.4.0 bior_gff3_to_tjson                                              |
|                                                                       |
| 2.4.1 bior_rsid_to_tjson                                              |
|                                                                       |
| 3.0.0 bior_catalog_remove_duplicates                                  |
|                                                                       |
| 3.0.0 bior_variant_to_tjson                                           |
|                                                                       |
| 4.1.2 bior_build_catalog                                              |
|                                                                       |
| 4.1.2 bior_count_catalog_misses                                       |
|                                                                       |
| 4.1.2 bior_verify_catalog                                             |
|                                                                       |
| 4.3.0 bior_catalog_stats                                              |
|                                                                       |
| 4.3.0 bior_modify_tjson                                               |
|                                                                       |
| 4.3.0 bior_replace_lines                                              |
+-----------------------------------------------------------------------+

Table 1 has a more complete description of these commands.

Commands in the toolkit operate on tab delimited data with a VCF style
header (starting with “#”). Commands in the toolkit insert additional
annotation to the right. Raw annotation is obtained by comparing JSON
objects in columns to JSON objects in catalogs. Table 1.0 shows the
format of columns <in,out> of each BioR function. For example
bior_vcf_to_tjson takes as an input VCF columns (and the header) and
outputs VCF + JSON in the last column.

+-----------------------+-----------------------+-----------------------+
| **Command**           | **Input, Output**     | **Description**       |
+-----------------------+-----------------------+-----------------------+
| bior_annotate         | VCF, TJSON            | Append to the VCF     |
|                       |                       | ‘info’ field a set of |
|                       |                       | commonly used         |
|                       |                       | annotations.          |
+-----------------------+-----------------------+-----------------------+
| bior_annotate_blaster | VCF, TJSON            | Similar to            |
|                       |                       | bior_annotate, but it |
|                       |                       | uses the grid engine  |
|                       |                       | to split the input    |
|                       |                       | VCF into multiple     |
|                       |                       | smaller chunks and    |
|                       |                       | annotate those chunks |
|                       |                       | concurrently before   |
|                       |                       | re-assembling them    |
|                       |                       | back into a single    |
|                       |                       | file                  |
+-----------------------+-----------------------+-----------------------+
| bior_bed_to_tjson     | BED, TJSON            | Load a BED file and   |
|                       |                       | convert to TJSON      |
|                       |                       | format.               |
+-----------------------+-----------------------+-----------------------+
| bior_build_catalog    | (various), Catalog    | Creates a catalog bgz |
|                       |                       | file from some data   |
|                       |                       | source, along with    |
|                       |                       | the accompanying      |
|                       |                       | columns.tsv and       |
|                       |                       | datasource.properties |
|                       |                       | files. Also verifies  |
|                       |                       | the catalog for       |
|                       |                       | conformity with the   |
|                       |                       | catalog spec and      |
|                       |                       | consistency with      |
|                       |                       | reference assemblies  |
+-----------------------+-----------------------+-----------------------+
| bior_catalog_remove_d | TJSON, TJSON          | Keeps the first of    |
| uplicates             |                       | several duplicate     |
|                       |                       | lines depending on    |
|                       |                       | keys specified by the |
|                       |                       | user                  |
+-----------------------+-----------------------+-----------------------+
| bior_catalog_stats    | TJSON, (stats files)  | Show statistics about |
|                       |                       | a catalog - from      |
|                       |                       | frequency of          |
|                       |                       | characters occurring  |
|                       |                       | on what percentage of |
|                       |                       | lines, to a list of   |
|                       |                       | 1000 possible values  |
|                       |                       | for each field        |
+-----------------------+-----------------------+-----------------------+
| bior_chunk            | VCF, VCF              | Breaks up a VCF into  |
|                       |                       | chunks based on start |
|                       |                       | and end lines         |
+-----------------------+-----------------------+-----------------------+
| bior_compress         | TJSON, TJSON          | Compress entries from |
|                       |                       | provided set of       |
|                       |                       | identifiers into a    |
|                       |                       | single entry with     |
|                       |                       | each value separated  |
|                       |                       | by a delimiter.       |
+-----------------------+-----------------------+-----------------------+
| bior_concat           | VCF, VCF              | Concatenate multiple  |
|                       |                       | VCF files together to |
|                       |                       | form one large one    |
+-----------------------+-----------------------+-----------------------+
| bior_count_catalog_mi | catalog, report       | Report the number of  |
| sses                  |                       | misses that would     |
|                       |                       | occur in a catalog    |
|                       |                       | due to the Tabix      |
|                       |                       | Reader bug that was   |
|                       |                       | found in Broad code   |
+-----------------------+-----------------------+-----------------------+
| bior_create_catalog   | TJSON, catalog        | Convert a text        |
|                       |                       | tabulated file into a |
|                       |                       | catalog. Chromosome   |
|                       |                       | ID, Start and End     |
|                       |                       | genomics position     |
|                       |                       | fields have to be     |
|                       |                       | explicitly named.     |
+-----------------------+-----------------------+-----------------------+
| bior\_                | catalog, property     | Create property files |
| create_catalog_props  |                       | from the metadata     |
|                       |                       | extracted from a      |
|                       |                       | catalog. Property     |
|                       |                       | files are needs for   |
|                       |                       | proper metadata       |
|                       |                       | handling.             |
+-----------------------+-----------------------+-----------------------+
| bior_create_config_fo | TSV,config            | Create a              |
| r_tab_to_tjson        |                       | configuration file    |
|                       |                       | that describes column |
|                       |                       | description. This     |
|                       |                       | file is needed when   |
|                       |                       | uploading a tab       |
|                       |                       | delimited file.       |
+-----------------------+-----------------------+-----------------------+
| bior_drill            | TJSON, TJSON          | Extract an element    |
|                       |                       | from nested JSON      |
|                       |                       | string.               |
+-----------------------+-----------------------+-----------------------+
| bior_gbk_to_tjson     | Genbank, TJSON        | Takes one or more     |
|                       |                       | genbank (gbk) files   |
|                       |                       | from input and        |
|                       |                       | outputs them as TJSON |
|                       |                       | in STDOUT             |
+-----------------------+-----------------------+-----------------------+
| bior_gff3_to_tjson    | gff3, TJSON           | Takes variant data in |
|                       |                       | GFF3 format from      |
|                       |                       | STDIN and converts it |
|                       |                       | into JSON as an       |
|                       |                       | additional column     |
|                       |                       | that is output to     |
|                       |                       | STDOUT                |
+-----------------------+-----------------------+-----------------------+
| bior_index_catalog    | identifier, index     | Index the specified   |
|                       |                       | identifier in a       |
|                       |                       | catalog. Indices a    |
|                       |                       | stored in a separate  |
|                       |                       | index file.           |
+-----------------------+-----------------------+-----------------------+
| bior_lookup           | TJSON, TJSON          | Extract annotations   |
|                       |                       | from a catalog based  |
|                       |                       | on matching values of |
|                       |                       | an identifier.        |
+-----------------------+-----------------------+-----------------------+
| bior_merge            | VCF, VCF              | Merges multiple VCFs  |
|                       |                       | together into one     |
|                       |                       | large one. This is    |
|                       |                       | done by looking at    |
|                       |                       | the next line in each |
|                       |                       | file to determine     |
|                       |                       | which should be       |
|                       |                       | inserted into the     |
|                       |                       | large VCF (vs         |
|                       |                       | bior-concat which     |
|                       |                       | simply outputs one    |
|                       |                       | file after another    |
|                       |                       | without looking at    |
|                       |                       | content of the lines) |
+-----------------------+-----------------------+-----------------------+
| bior_modify_tjson     | TJSON, TJSON          | Given a config file   |
|                       |                       | that specifies how to |
|                       |                       | transform data types  |
|                       |                       | and values, modify    |
|                       |                       | the TJSON on the fly  |
|                       |                       | (during streaming)    |
+-----------------------+-----------------------+-----------------------+
| bior_overlap          | TJSON, TJSON          | Extract annotations   |
|                       |                       | from a catalog based  |
|                       |                       | on genomic location   |
|                       |                       | overlap. The overlap  |
|                       |                       | is computed from the  |
|                       |                       | Start and End         |
|                       |                       | genomics position of  |
|                       |                       | a variant.            |
+-----------------------+-----------------------+-----------------------+
| bior_pretty_print     | TJSON, STDOUT         | Convert TJSON in a    |
|                       |                       | readable format for   |
|                       |                       | screen or file        |
|                       |                       | output.               |
+-----------------------+-----------------------+-----------------------+
| bior_ref_allele       | TJSON, TJSON          | Retrieves the         |
|                       |                       | reference allele from |
|                       |                       | the NCBI Genome       |
|                       |                       | database that matches |
|                       |                       | a chromosome, start,  |
|                       |                       | end position          |
+-----------------------+-----------------------+-----------------------+
| bior_replace_lines    | TJSON, TJSON          | Given two input files |
|                       |                       | (1 with lines to      |
|                       |                       | find, and 1 with      |
|                       |                       | lines replace),       |
|                       |                       | replace whole lines   |
|                       |                       | in the TJSON          |
+-----------------------+-----------------------+-----------------------+
| bior_rsid_to_tjson    | text, TJSON           | converts rsIDs into   |
|                       |                       | JSON as an additional |
|                       |                       | column                |
+-----------------------+-----------------------+-----------------------+
| bior_same_variant     | TJSON, TJSON          | Extract annotations   |
|                       |                       | from a catalog based  |
|                       |                       | on variant position,  |
|                       |                       | reference and         |
|                       |                       | alternate allele      |
|                       |                       | definition.           |
+-----------------------+-----------------------+-----------------------+
| bior_snpeff           | TJSON, TJSON          | Use                   |
|                       |                       | SNPEffect\ :sup:`1`   |
        |                       |                       | to annotate variants. |
        |                       |                       | Chromosome ID, Start  |
        |                       |                       | and Stop genomics     |
        |                       |                       | position, reference   |
        |                       |                       | and alternate allele  |
        |                       |                       | of the variant is     |
        |                       |                       | required .            |
        +-----------------------+-----------------------+-----------------------+
        | bior_tab_to_tjson     | TSV, TJSON            | Load a tab-delimited  |
        |                       |                       | file and convert to   |
        |                       |                       | TJSON format.         |
        +-----------------------+-----------------------+-----------------------+
        | bior_tjson_to_vcf     | TJSON, VCF            | Convert TJSON to VCF  |
        |                       |                       | format for file       |
        |                       |                       | output.               |
        +-----------------------+-----------------------+-----------------------+
        | bior_trim_spaces      | TJSON,TJSON           | Trims spaces from     |
        |                       |                       | around tab-separated  |
        |                       |                       | columns. Use this if  |
        |                       |                       | you find spaces       |
        |                       |                       | before or after your  |
        |                       |                       | vcf columns that      |
        |                       |                       | crash the tools or    |
        |                       |                       | cause VEP to take a   |
        |                       |                       | lot of memory.        |
        +-----------------------+-----------------------+-----------------------+
        | bior_variant_to_tjson | tsv, TJSON            | Converts rsID or      |
        |                       |                       | position data into    |
        |                       |                       | JSON as an additional |
        |                       |                       | column                |
        +-----------------------+-----------------------+-----------------------+
        | bior_vcf_to_tjson     | VCF, TJSON            | Load a VCF file and   |
        |                       |                       | convert to TJSON      |
        |                       |                       | format.               |
        +-----------------------+-----------------------+-----------------------+
        | bior_vep              | TJSON, TJSON          | Use VEP\ :sup:`2` to  |
        |                       |                       | annotate variants.    |
        |                       |                       | Chromosome ID, Start  |
        |                       |                       | and Stop genomics     |
        |                       |                       | position, reference   |
        |                       |                       | and alternate allele  |
        |                       |                       | of the variant is     |
        |                       |                       | required.             |
        +-----------------------+-----------------------+-----------------------+
        | bior_verify_catalog   | TJSON, Report         | Verifies the catalog  |
        |                       |                       | structure, reference  |
        |                       |                       | base pairs, as well   |
        |                       |                       | as columns.tsv and    |
        |                       |                       | datasource.properties |
        |                       |                       | files against the     |
        |                       |                       | values expected from  |
        |                       |                       | crawling the catalog  |
        +-----------------------+-----------------------+-----------------------+

        Table 1: List of commands available in the BioR Toolkit. Detailed
        description and example is displayed when executing the command with the
        –h flag.

:sup:`1`\ Cingolani, P. et al. (2012) A program for annotating and
predicting the effects of single nucleotide polymorphisms, SnpEff: SNPs
in the genome of Drosophila melanogaster strain w1118; iso-2; iso-3. Fly
(Austin). 6(2) :p. 80-92.

:sup:`2`\ McLaren W et al. (2010) Deriving the consequences of genomic
variants with the Ensembl API and SNP Effect Predictor. BMC
Bioinformatics 26(16):2069-70

Most every one of these commands supports the –h (help) flag to get
information about how to use the command. To get help on
bior_vcf_to_tjson type:

+--------------------------------------------------------------------------+
| $ bior_vcf_to_tjson -h                                                   |
|                                                                          |
| NAME                                                                     |
|                                                                          |
| bior_vcf_to_tjson -- converts VCF data into JSON as an additional column |
|                                                                          |
| SYNOPSIS                                                                 |
|                                                                          |
| bior_vcf_to_tjson [--log] [--help]                                       |
|                                                                          |
| ...                                                                      |
+--------------------------------------------------------------------------+
