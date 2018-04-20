
bior_build_catalog
~~~~~~~~~~~~~~~~~~

This command should be your go-to command for automating the build and
verification of new catalogs.

Internally, the command consists of these steps, which are run in this
order to produce a catalog

1. make_json - calls a user-provided script that produces the JSON for
   the catalog

2. make_catalog - creates the catalog format with columns for
   chromosome, minBasePairPosition, maxBasePairPosition, JsonData. It
   bgzip's the data, then applies a tabix index to it for quick range
   query searches.

3. make_prop_files - creates the default properties files (columns.tsv
   and datasource.properties) by crawling the catalog to determine keys,
   types, number.

4. merge_prop_files - merges properties file data from previous catalog
   data, the default files from catalog crawling, and user's manual
   changes.

5. make_indexes - Creates indexes for bior_lookup for all keys specified
   by the user (which it puts in an H2 database, mapping each value for
   each key to a specific line location within the bgzip catalog)

6. verify - Runs several verification checks against the catalog,
   properties files, and indexes; such as:

   a. Verifying all lines are in sorted order

   b. Reference alleles match those in the reference assembly

   c. The first and last lines of each catalog are compared with the
      appropriate lines obtained from a tabix search on the same
      positions.

   d. Properties file match the expected column names and types

   e. Indexes correctly point to the expected lines

The command takes a build_info.txt file which contains several keys that
describe the data source as well as paths to temp and target
directories. For example:

+-----------------------------------------------------------------------+
| DATA_SOURCE=dbSnp                                                     |
|                                                                       |
| DATA_SOURCE_VERSION=142                                               |
|                                                                       |
| DATA_SOURCE_BUILD=GRCh37                                              |
|                                                                       |
| CATALOG_PREFIX=my_dbsnp_catalog_v2                                    |
|                                                                       |
| MAKE_JSON_SCRIPT_PATH=/data5/bsi/somePretendInputDir/build_dbsnp_tjso |
| n_input_b37.sh                                                        |
|                                                                       |
| MAKE_JSON_ARGS=myInputDbsnpDataSourceFile.txt                         |
| anotherImportantParameterToBuildScript                                |
|                                                                       |
| MAKE_JSON_OUTPUT_FILE_PATH=my_dbsnp_tjson_input_b37.tsv               |
|                                                                       |
| TARGET_DIR=/data5/bsi/somePretendOutputDir/v2/                        |
|                                                                       |
| TEMP_DIR=/local2/tmp/someTempDir/                                     |
|                                                                       |
| INDEXES=RSID,POS                                                      |
|                                                                       |
| PREVIOUS_CATALOG_PATH=/data5/bsi/somePretendOutputDir/v1/catalog_v1.t |
| sv.bgz                                                                |
|                                                                       |
| SORT_ROWS=false                                                       |
+-----------------------------------------------------------------------+

Here we'll build a simple catalog from a basic VCF file and
build_info.txt file in a temp directory on one of the RCF servers

+-----------------------------------------------------------------------+
| **$ cat myMiniDbsnp.vcf**                                             |
|                                                                       |
| ##INFO=<ID=RS,Number=1,Type=Integer,Description="dbSNP ID (i.e. rs    |
| number)">                                                             |
|                                                                       |
| ##INFO=<ID=RSPOS,Number=1,Type=Integer,Description="Chr position      |
| reported in dbSNP">                                                   |
|                                                                       |
| ##INFO=<ID=R5,Number=0,Type=Flag,Description="In 5' gene region       |
| FxnCode = 15">                                                        |
|                                                                       |
| #CHROM POS ID REF ALT QUAL FILTER INFO                                |
|                                                                       |
| 1 10019 rs775809821 TA T . . RS=775809821;RSPOS=10020;R5              |
|                                                                       |
| 1 10039 rs978760828 A C . . RS=978760828;RSPOS=10039;R5               |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
| **$ cat build_info.txt**                                              |
|                                                                       |
| DATA_SOURCE=dbSnp                                                     |
|                                                                       |
| DATA_SOURCE_VERSION=142                                               |
|                                                                       |
| DATA_SOURCE_BUILD=GRCh37                                              |
|                                                                       |
| CATALOG_PREFIX=mini_dbsnp_catalog_v2                                  |
|                                                                       |
| # Path to a relative script                                           |
|                                                                       |
| MAKE_JSON_SCRIPT_PATH=make_json.sh                                    |
|                                                                       |
| # Provide the path to the mini_dbSNP source data directory (current   |
| directory)                                                            |
|                                                                       |
| MAKE_JSON_ARGS=.                                                      |
|                                                                       |
| # Leave blank to accept default temp path                             |
|                                                                       |
| MAKE_JSON_OUTPUT_FILE_PATH=                                           |
|                                                                       |
| # Use the "Target" subdirectory within the current working directory  |
| (Leave blank to use the current working dir)                          |
|                                                                       |
| TARGET_DIR=Target                                                     |
|                                                                       |
| # Leave blank to let the command find the best temp location. Usually |
| in this order: /local2/tmp, /local1/tmp/, /tmp                        |
|                                                                       |
| TEMP_DIR=                                                             |
|                                                                       |
| # Create an index on ID                                               |
|                                                                       |
| INDEXES=ID                                                            |
|                                                                       |
| # Point to the previous dbSNP catalog to pull descriptions from       |
| column.tsv                                                            |
|                                                                       |
| PREVIOUS_CATALOG_PATH=/data5/bsi/catalogs/bior/v1/dbSNP/142_GRCh37.p1 |
| 3/variants_nodups.v2/00-All.vcf.tsv.bgz                               |
|                                                                       |
| # Don't sort the rows after the make_json.sh script is run            |
|                                                                       |
| SORT_ROWS=false                                                       |
+-----------------------------------------------------------------------+

There are some good utility scripts available in the "bior-catalogs"
TFS-Git repository under the "common/" directory for checking
directories, BioR versions, etc. View the make_json.sh scripts for
several of the existing catalog builds there for more details. This
make_json.sh is a very simplistic version

+-----------------------------------------------------------------------+
| **$ touch make_json.sh**                                              |
|                                                                       |
| **# Make the script executable**                                      |
|                                                                       |
| **$ chmod u+x make_json.sh**                                          |
|                                                                       |
| **# Edit make_json.sh....**                                           |
|                                                                       |
| **$ cat make_json.sh**                                                |
|                                                                       |
| ## Cat the mini vcf                                                   |
|                                                                       |
| ## Convert it from VCF to TJSON                                       |
|                                                                       |
| ## Drill out the \_landmark, \_minBP, and \_maxBP fields, using the   |
| -k flag to keep the JSON on the end                                   |
|                                                                       |
| ## Remove the header metadata with grep -v                            |
|                                                                       |
| ## Remove the first 8 columns using cut                               |
|                                                                       |
| ## Dump to the default MAKE_JSON_OUTPUT_FILE_PATH                     |
|                                                                       |
| cat $MAKE_JSON_ARGS/myMiniDbsnp.vcf \| bior_vcf_to_tjson \|           |
| bior_drill -p \_landmark -p \_minBP -p \_maxBP -k \| grep -v "^#" \|  |
| cut -f9- > $MAKE_JSON_OUTPUT_FILE_PATH                                |
+-----------------------------------------------------------------------+

This make_json.sh script will produce an output file similar to:

+-----------------------------------------------------------------------+
| 1 10019 10020                                                         |
| {"CHROM":"1","POS":10019,"ID":"rs775809821","REF":"TA","ALT":"T","INF |
| O":{"RS":775809821,"RSPOS":10020,"R5":true},"_id":"rs775809821","_typ |
| e":"variant","_landmark":"1","_refAllele":"TA","_altAlleles":["T"],"_ |
| minBP":10019,"_maxBP":10020}                                          |
|                                                                       |
| 1 10039 10039                                                         |
| {"CHROM":"1","POS":10039,"ID":"rs978760828","REF":"A","ALT":"C","INFO |
| ":{"RS":978760828,"RSPOS":10039,"R5":true},"_id":"rs978760828","_type |
| ":"variant","_landmark":"1","_refAllele":"A","_altAlleles":["C"],"_mi |
| nBP":10039,"_maxBP":10039}                                            |
+-----------------------------------------------------------------------+

Now, we run bior_build_catalog

+-----------------------------------------------------------------------+
| $ mkdir Target                                                        |
|                                                                       |
| $ bior_build_catalog -b build_info.txt                                |
|                                                                       |
| --------------------------------------------------------------------- |
| ----------------                                                      |
|                                                                       |
| Variables from build_info.txt (as well as BIOR_LITE_HOME, JAVA_HOME,  |
| and PATH)                                                             |
|                                                                       |
| --------------------------------------------------------------------- |
| ----------------                                                      |
|                                                                       |
| MAKE_JSON_SCRIPT_PATH                                                 |
| /local2/tmp/m054457/bior_build_catalog_temp/make_json.sh              |
|                                                                       |
| JAVA_HOME /usr/lib/jvm/jre-1.6.0-openjdk.x86_64                       |
|                                                                       |
| SORT_ROWS false                                                       |
|                                                                       |
| PATH                                                                  |
| /home/m054457/Tools:/data5/bsi/bictools/scripts/bior_annotate/v2.8.7/ |
| trunk:/home/m054457/perl5/bin:/home/m054457/apache-maven-3.2.3/bin:/p |
| rojects/bsi/bictools/apps/variant_detection/vcftools/0.1.11/perl/:/ho |
| me/m054457/bin:/home/m054457/groovy-2.4.3/bin:/data5/bsi/bictools/tig |
| er_toolkit/tiger_toolkit-1.1-SNAPSHOT/bin/:/data5/bsi/bictools/bin:/p |
| rojects/bsi/bictools/scripts/util:/data5/bsi/bictools/src/samtools/1. |
| 3.1/htslib-1.3.1:/projects/bsi/bictools/apps/alignment/tabix/0.2.5/:/ |
| projects/bsi/bictools/apps/variant_detection/vcftools/0.1.8a/perl/:/p |
| rojects/bsi/bictools/apps/variant_detection/vcftools/0.1.8a/lib/perl5 |
| /site_perl/:/usr/local/biotools/refdata/1.0.beta/current/bin:/usr/loc |
| al/biotools/blast/blast-2.2.13/bin:/usr/local/biotools/hmmer/hmmer-3. |
| 0/bin:/usr/local/biotools/perl/5.16.2-centos6/bin:/usr/local/biotools |
| /misc/ver1/bin:/usr/local/biotools/java/jdk1.6.0_05/bin:/usr/local/bi |
| otools/emboss/EMBOSS-5.0.0/bin:/home/oge/ge2011.11/bin/linux-x64:/hom |
| e/oge/ge2011.11/bin/linux-x64:/usr/local/biotools/bior_scripts/4.3.0/ |
| bior_pipeline-4.3.0/bin:/usr/local/biotools/git/2.6.4/bin:/usr/local/ |
| biotools/ruby/2.0.0-p195/bin:/usr/lib64/qt-3.3/bin:/usr/local/bin:/bi |
| n:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/home/m054457/bin          |
|                                                                       |
| DATA_SOURCE_BUILD GRCh37                                              |
|                                                                       |
| TARGET_DIR /local2/tmp/m054457/bior_build_catalog_temp/Target         |
|                                                                       |
| PREVIOUS_CATALOG_PATH                                                 |
| /data5/bsi/catalogs/bior/v1/dbSNP/142_GRCh37.p13/variants_nodups.v2/0 |
| 0-All.vcf.tsv.bgz                                                     |
|                                                                       |
| MAKE_JSON_OUTPUT_FILE_PATH                                            |
| /local2/tmp/m054457/bior_build_catalog_tmp/tmp_18681_bior_build_catal |
| og/make_json_output.tsv                                               |
|                                                                       |
| HOSTNAME franklin03.mayo.edu                                          |
|                                                                       |
| TEMP_DIR                                                              |
| /local2/tmp/m054457/bior_build_catalog_tmp/tmp_18681_bior_build_catal |
| og                                                                    |
|                                                                       |
| INDEXES ID                                                            |
|                                                                       |
| CATALOG_PREFIX mini_dbsnp_catalog_v2                                  |
|                                                                       |
| BIOR_LITE_HOME                                                        |
| /usr/local/biotools/bior_scripts/4.3.0/bior_pipeline-4.3.0            |
|                                                                       |
| DATA_SOURCE dbSnp                                                     |
|                                                                       |
| MAKE_JSON_ARGS .                                                      |
|                                                                       |
| DATA_SOURCE_VERSION 142                                               |
|                                                                       |
| --------------------------------------------------------------------- |
| ----------------                                                      |
|                                                                       |
| Step make_json STARTED                                                |
|                                                                       |
| Executing script                                                      |
| '/local2/tmp/m054457/bior_build_catalog_temp/make_json.sh' with args: |
| '.'                                                                   |
|                                                                       |
| Step make_json SUCCEEDED                                              |
|                                                                       |
| -------------------------------------                                 |
|                                                                       |
| Step make_catalog STARTED                                             |
|                                                                       |
| Creating catalog from JSON in last column of:                         |
| '/local2/tmp/m054457/bior_build_catalog_tmp/tmp_18681_bior_build_cata |
| log/make_json_output.tsv'                                             |
|                                                                       |
| Catalog created:                                                      |
| /local2/tmp/m054457/bior_build_catalog_temp/Target/mini_dbsnp_catalog |
| _v2.tsv.bgz                                                           |
|                                                                       |
| Tabix index created:                                                  |
| /local2/tmp/m054457/bior_build_catalog_temp/Target/mini_dbsnp_catalog |
| _v2.tsv.bgz.tbi                                                       |
|                                                                       |
| Step make_catalog SUCCEEDED                                           |
|                                                                       |
| -------------------------------------                                 |
|                                                                       |
| Step make_prop_files STARTED                                          |
|                                                                       |
| Datasource properties file created at:                                |
| /local2/tmp/m054457/bior_build_catalog_temp/Target/mini_dbsnp_catalog |
| _v2.datasource.properties                                             |
|                                                                       |
| Read 2 lines from catalog                                             |
|                                                                       |
| Columns properties file created at:                                   |
| /local2/tmp/m054457/bior_build_catalog_temp/Target/mini_dbsnp_catalog |
| _v2.columns.tsv                                                       |
|                                                                       |
| Blacklist properties file created at:                                 |
| /local2/tmp/m054457/bior_build_catalog_temp/Target/mini_dbsnp_catalog |
| _v2.columns.tsv.blacklist                                             |
|                                                                       |
| Blacklist (BioRWeb) properties file created at:                       |
| /local2/tmp/m054457/bior_build_catalog_temp/Target/mini_dbsnp_catalog |
| _v2.columns.tsv.blacklist.biorweb                                     |
|                                                                       |
| Step make_prop_files SUCCEEDED                                        |
|                                                                       |
| -------------------------------------                                 |
|                                                                       |
| Step merge_prop_files STARTED                                         |
|                                                                       |
| Some changes made to                                                  |
| '/local2/tmp/m054457/bior_build_catalog_temp/Target/mini_dbsnp_catalo |
| g_v2.datasource.properties'                                           |
|                                                                       |
| Some changes made to                                                  |
| '/local2/tmp/m054457/bior_build_catalog_temp/Target/mini_dbsnp_catalo |
| g_v2.columns.tsv'                                                     |
|                                                                       |
| Copying previous catalog blacklist file                               |
|                                                                       |
| Copying previous catalog blacklist.biorweb file                       |
|                                                                       |
| Step merge_prop_files SUCCEEDED                                       |
|                                                                       |
| -------------------------------------                                 |
|                                                                       |
| Step make_indexes STARTED                                             |
|                                                                       |
| Creating index on key 'ID', path                                      |
| '/local2/tmp/m054457/bior_build_catalog_temp/Target/index/mini_dbsnp_ |
| catalog_v2.ID.idx.h2.db'                                              |
|                                                                       |
| Step make_indexes SUCCEEDED                                           |
|                                                                       |
| -------------------------------------                                 |
|                                                                       |
| Step verify STARTED                                                   |
|                                                                       |
| Verifying catalog                                                     |
| '/local2/tmp/m054457/bior_build_catalog_temp/Target/mini_dbsnp_catalo |
| g_v2.tsv.bgz'                                                         |
| starting at 2:07:44 AM on Apr 25, 2017                                |
|                                                                       |
| Verifying chromosomal order and indexes for                           |
| '/local2/tmp/m054457/bior_build_catalog_temp/Target/mini_dbsnp_catalo |
| g_v2.tsv.bgz'                                                         |
| starting at 2:07:44 AM on Apr 25, 2017                                |
|                                                                       |
| Verifying each row for                                                |
| '/local2/tmp/m054457/bior_build_catalog_temp/Target/mini_dbsnp_catalo |
| g_v2.tsv.bgz'                                                         |
| starting at 2:07:44 AM on Apr 25, 2017                                |
|                                                                       |
| Verified 2 rows of catalog, finished at 2:07:44 AM on Apr 25, 2017.   |
|                                                                       |
| Verify #ERROR: 0, #WARNING: 0. See details in                         |
| 'mini_dbsnp_catalog_v2_verify.txt'                                    |
|                                                                       |
| Step verify SUCCEEDED                                                 |
|                                                                       |
| -------------------------------------                                 |
+-----------------------------------------------------------------------+

Note the temp directory that is listed in the directory above. This can
sometimes give clues to any problems that arise

+-----------------------------------------------------------------------+
| **$ ls -aR                                                            |
| /local2/tmp/m054457/bior_build_catalog_tmp/tmp_18681_bior_build_catal |
| og**                                                                  |
|                                                                       |
| /local2/tmp/m054457/bior_build_catalog_tmp/tmp_18681_bior_build_catal |
| og:                                                                   |
|                                                                       |
| . .. make_json_output.tsv                                             |
+-----------------------------------------------------------------------+

Here are the main files that are generated, with an index created on the
"ID" field nder the Target/index directory

+-----------------------------------------------------+
| **$ ls -1R Target**                                 |
|                                                     |
| Target:                                             |
|                                                     |
| index                                               |
|                                                     |
| mini_dbsnp_catalog_v2.columns.tsv                   |
|                                                     |
| mini_dbsnp_catalog_v2.columns.tsv.blacklist         |
|                                                     |
| mini_dbsnp_catalog_v2.columns.tsv.blacklist.biorweb |
|                                                     |
| mini_dbsnp_catalog_v2.datasource.properties         |
|                                                     |
| mini_dbsnp_catalog_v2.tsv.bgz                       |
|                                                     |
| mini_dbsnp_catalog_v2.tsv.bgz.tbi                   |
|                                                     |
| Target/index:                                       |
|                                                     |
| mini_dbsnp_catalog_v2.ID.idx.h2.db                  |
+-----------------------------------------------------+

Other logging and default properties files used in the build of the
catalog appear under the Target/.build subdirectory, and can also be
helpful in tracking down any problems that may arise

+-------------------------------------------------------------+
| **$ ls -1R Target/.build**                                  |
|                                                             |
| Target/.build:                                              |
|                                                             |
| mini_dbsnp_catalog_v2.columns.tsv.01                        |
|                                                             |
| mini_dbsnp_catalog_v2.columns.tsv.blacklist.biorweb.default |
|                                                             |
| mini_dbsnp_catalog_v2.columns.tsv.blacklist.default         |
|                                                             |
| mini_dbsnp_catalog_v2.columns.tsv.default                   |
|                                                             |
| mini_dbsnp_catalog_v2.datasource.properties.01              |
|                                                             |
| mini_dbsnp_catalog_v2.datasource.properties.default         |
|                                                             |
| progress                                                    |
|                                                             |
| Target/.build/progress:                                     |
|                                                             |
| make_catalog.log                                            |
|                                                             |
| make_catalog.successful                                     |
|                                                             |
| make_catalog.summary                                        |
|                                                             |
| make_indexes.log                                            |
|                                                             |
| make_indexes.successful                                     |
|                                                             |
| make_indexes.summary                                        |
|                                                             |
| make_json.log                                               |
|                                                             |
| make_json.successful                                        |
|                                                             |
| make_json.summary                                           |
|                                                             |
| make_prop_files.log                                         |
|                                                             |
| make_prop_files.successful                                  |
|                                                             |
| make_prop_files.summary                                     |
|                                                             |
| merge_prop_files.log                                        |
|                                                             |
| merge_prop_files.successful                                 |
|                                                             |
| merge_prop_files.summary                                    |
|                                                             |
| verify.log                                                  |
|                                                             |
| verify.successful                                           |
|                                                             |
| verify.summary                                              |
+-------------------------------------------------------------+
