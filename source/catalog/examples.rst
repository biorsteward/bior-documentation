Creating Catalogs
=====================

Indexing your Samples
---------------------

Lets say you want to get variants in your sample that overlap a gene.
 One way to do this is to stream the variants e.g:

+-----------------------------------------------------------------------+
| > cat example.vcf \| head                                             |
|                                                                       |
| ##fileformat=VCFv4.0                                                  |
|                                                                       |
| #CHROM POS ID REF ALT QUAL FILTER INFO                                |
|                                                                       |
| 21 26960070 rs116645811 G A . . .                                     |
|                                                                       |
| 21 26965148 rs1135638 G A . . .                                       |
|                                                                       |
| 21 26965172 rs010576 T C . . .                                        |
|                                                                       |
| 21 26965205 rs1057885 T C . . .                                       |
|                                                                       |
| 21 26976144 rs116331755 A G . . .                                     |
|                                                                       |
| 21 26976222 rs7278168 C T . . .                                       |
|                                                                       |
| 21 26976237 rs7278284 C T . . .                                       |
|                                                                       |
| 21 26978790 rs75377686 T C . . .                                      |
|                                                                       |
| >cat example.vcf \| bior_vcf_to_tjson \| bior_overlap -d              |
| $bior/NCBIGene/GRCh37_p10/genes.tsv.bgz \| grep "\"gene\":\"PANX2\""  |
|                                                                       |
| 22 50616005 rs35195493 C G . . .                                      |
| {"CHROM":"22","POS":"50616005","ID":"rs35195493","REF":"C","ALT":"G", |
| "QUAL":".","FILTER":".","INFO":{".":true},"_id":"rs35195493","_type": |
| "variant","_landmark":"22","_refAllele":"C","_altAlleles":["G"],"_min |
| BP":50616005,"_maxBP":50616005}                                       |
| {"_type":"gene","_landmark":"22","_strand":"+","_minBP":50609160,"_ma |
| xBP":50618724,"gene":"PANX2","gene_synonym":"hPANX2;                  |
| PX2","note":"pannexin 2; Derived by automated computational analysis  |
| using gene prediction method:                                         |
| BestRefseq.","GeneID":"56666","HGNC":"8600","HPRD":"09760","MIM":"608 |
| 421"}                                                                 |
|                                                                       |
| 22 50616806 rs5771206 A G . . .                                       |
| {"CHROM":"22","POS":"50616806","ID":"rs5771206","REF":"A","ALT":"G"," |
| QUAL":".","FILTER":".","INFO":{".":true},"_id":"rs5771206","_type":"v |
| ariant","_landmark":"22","_refAllele":"A","_altAlleles":["G"],"_minBP |
| ":50616806,"_maxBP":50616806}                                         |
| {"_type":"gene","_landmark":"22","_strand":"+","_minBP":50609160,"_ma |
| xBP":50618724,"gene":"PANX2","gene_synonym":"hPANX2;                  |
| PX2","note":"pannexin 2; Derived by automated computational analysis  |
| using gene prediction method:                                         |
| BestRefseq.","GeneID":"56666","HGNC":"8600","HPRD":"09760","MIM":"608 |
| 421"}                                                                 |
|                                                                       |
| $                                                                     |
+-----------------------------------------------------------------------+

 

 

If you just want variants that overlap any gene, you can always do
something like:

+-----------------------------------------------------------------+
| >zcat $bior/NCBIGene/                                           |
|                                                                 |
| GRCh37_p10/genes.tsv.bgz \| bior_overlap -d ./example.tsv.gz \| |
|                                                                 |
| grep -v "{}" \| less                                            |
+-----------------------------------------------------------------+

 

 

That works fine for a single gene, but what if you are starting with a
list of genes?  e.g.

+------------------+
| >cat mygenes.txt |
|                  |
| MRPL39           |
|                  |
| PANX2            |
|                  |
| BRCA1            |
|                  |
| ...              |
+------------------+

 

In this case you may want to use an index on your data.  To create the
index, do something like:

+-----------------------------------------------------------------------+
| >cat example.vcf \| bior_vcf_to_tjson \| grep "^#" \| cut -f 1,2,9 \| |
|                                                                       |
| bior_drill -k -p \_maxBP > example.tsv                                |
|                                                                       |
| >sort -k1,1 -k2,2n example.tsv                                        |
|                                                                       |
| >bgzip example.tsv                                                    |
|                                                                       |
| >tabix example.tsv.gz                                                 |
|                                                                       |
| >tabix -s 1 -b 2 -e 3 example.tsv.gz                                  |
+-----------------------------------------------------------------------+

 

Now use lookup to get the gene locations, and overlap to overlap those
locations with your data:

+-------------------------------------------------------+
| >cat mygenes.txt \| bior_lookup -p gene               |
|                                                       |
| -d $bior/NCBIGene/GRCh37_p10/genes.tsv.bgz \|         |
|                                                       |
| bior_overlap -d ./example.tsv.gz \| bior_pretty_print |
+-------------------------------------------------------+

You can now use bior_same_variant to annotate variants that overlap your
genes.

Creating Custom Catalogs
------------------------

One of the most powerful things about BioR is that users can publish
their own catalogs and integrate new data into the system. They can also
share these catalogs with others making the system extensible and much
more powerful than a system where the catalogs must all be maintained by
a single annotation team.

The Publication Process
~~~~~~~~~~~~~~~~~~~~~~~

Publishing a catalog requires (1) a parser that understands arbitrarily
formatted file formats, and (2) indexing tools.  Parsers convert
arbitrary data representations into JSON with a set of 'golden
identifiers' the BioR system understands.  Example 'golden identifiers’
include \_landmark, \_minBP, and \_maxBP.  'Golden identifiers' are
always prefixed with an underscore ('_') and must be absolutely
consistent at both in terms of syntax and semantics.  For example,
\_minBP uses the standard 1-based coordnate system (e.g. NCBI/Blast) not
interbase coordinates
(http://gmod.org/wiki/Introduction_to_Chado#Interbase_Coordinates), and
\_strand is represented as '+', '-', or '.' and NOT 'complement' as in
the gbs files from NCBI.  One of the functions of a parser, is to
convert from arbitrary file formats into JSON, the other is to extract
the 'golden identifiers' and place them in the JSON.  'Golden
identifiers'  are created so that BioR programs (e.g. bior_overlap.sh)
can work on the information regardless of the source file format (e.g.
VCF, GFF, GBS, XML, RelationalDB, Tab-Delimited, ...).

As they become availible, parsers, will be exposed to users as command
line tools.  For example, bior_vcf_to_variants.sh is a parser that
converts vcf to BioR JSON.

In summary, to make a custom catalog, you need:

 

1.       Columns 1-3 bed-like (chr            start       stop)
[1-based]

2.       The 4\ :sup:`th` column is a series of key-value pairs enclosed
by quotes and brackets

3.       The 4 column contains “Golden identifiers” [ \_landmark,
\_minBP, and \_maxBP ]

 

Once this is created, use bgzip & tabix to compress and index it for
genomic search. For those samples that do NOT have a genomic position,
use the following values (bior_create_catalog will do this for you).

+-----------------------+------------------------------------+
| **Golden Identifier** | **Default Value**                  |
+-----------------------+------------------------------------+
| \_landmark            | UNKNOWN ( a period ‘.’ is also ok) |
+-----------------------+------------------------------------+
| \_minBP               | 0                                  |
+-----------------------+------------------------------------+
| \_maxBP               | 0                                  |
+-----------------------+------------------------------------+

Zero is important because it has to be an integer and must be greater
than zero.  The JSON does not have to have the golden attribute if you
won't search on it.

Parsing and Converting the Data
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If a parser for the file format is available (e.g. bior_vcf_to_tjson,
bior_bed_to_tjson, ect.) publishing a custom catalog is extremely easy. 
Using the standard BioR tools, a publication pipeline can be constructed
rapidly.  For example:

+-----------------------------------------------------------------------+
| zcat 00-All.vcf.gz \| bior_vcf_to_tjson.sh \| cut -f 9 \|             |
| bior_drill.sh -k -p \_landmark -p \_minBP -p \_maxBP > dbSNP.tsv      |
+-----------------------------------------------------------------------+

This pipeline streams the original VCF file past the parser
(bior_vcf_to_tjson), removes the content of the original VCF (cut -f 9)
- this is ok, as all of this information is duplicated in the JSON
format, drill out the key attributes (bior_drill.sh) so that they can be
indexed, and then output to a raw data file (dbSNP.tsv).  The raw output
file should look like this:

+-----------------------------------------------------------------------+
| $ head dbSNP.tsv                                                      |
|                                                                       |
| 1    10144    10145                                                   |
|  {"CHROM":"1","POS":"10144","ID":"rs144773400","REF":"TA","ALT":"T"," |
| QUAL":".","FILTER":".","INFO":{"RSPOS":10145,"dbSNPBuildID":134,"SSR" |
| :0,"SAO":0,"VP":"050000000005000002000200","WGT":1,"VC":"DIV","ASP":t |
| rue,"OTHERKG":true},"_id":"rs144773400","_type":"variant","_landmark" |
| :"1","_refAllele":"TA","_altAlleles":["T"],"_minBP":10144,"_maxBP":10 |
| 145}                                                                  |
|                                                                       |
| 1    10177    10177                                                   |
|  {"CHROM":"1","POS":"10177","ID":"rs201752861","REF":"A","ALT":"C","Q |
| UAL":".","FILTER":".","INFO":{"RSPOS":10177,"dbSNPBuildID":137,"SSR": |
| 0,"SAO":0,"VP":"050000000005000002000100","WGT":1,"VC":"SNV","ASP":tr |
| ue,"OTHERKG":true},"_id":"rs201752861","_type":"variant","_landmark": |
| "1","_refAllele":"A","_altAlleles":["C"],"_minBP":10177,"_maxBP":1017 |
| 7}                                                                    |
|                                                                       |
| ...                                                                   |
+-----------------------------------------------------------------------+

Examples: Building Catalogs from tab-delimited data
---------------------------------------------------

Let’s say we start with a VCF file (my.vcf):

+---------------------------------------------------+
| #CHROM POS ID REF ALT QUAL FILTER INFO rsID maxBP |
|                                                   |
| 1 100 111 A G . . . rs111 100                     |
|                                                   |
| 2 200 222 C T . . . rs222 200                     |
+---------------------------------------------------+

Now, we want to create a config file for constructing a catalog (this
will take the one column header line from the VCF, remove the #
character in front of CHROM, then use this line to create the config
file)

+-----------------------------------------------------------------------+
| head -1 my.vcf \| tr -d "#' \| bior_create_config_for_tab_to_tjson >  |
| my.tab                                                                |
+-----------------------------------------------------------------------+

We modify the my.tab output file and include golden attributes in the
last column. We also change the POS and maxBP to be of type NUMBER:

+------------------------------------+
| 1 CHROM STRING COLUMN . \_landmark |
|                                    |
| 2 POS NUMBER COLUMN . \_minBP      |
|                                    |
| 3 ID STRING COLUMN . .             |
|                                    |
| 4 REF STRING COLUMN . \_refAllele  |
|                                    |
| 5 ALT STRING COLUMN . \_altAllele  |
|                                    |
| 6 QUAL STRING COLUMN . .           |
|                                    |
| 7 FILTER STRING COLUMN . .         |
|                                    |
| 8 INFO STRING COLUMN . .           |
|                                    |
| 9 rsID STRING COLUMN . \_id        |
|                                    |
| 10 maxBP NUMBER COLUMN . \_maxBP   |
+------------------------------------+

Then we build the catalog:

+---------------------------------------+
| bior_create_catalog -i my.tjson -o my |
+---------------------------------------+

That produced the files (the first being the catalog, and the second
being the tabix index):

+----------------+
| my.tsv.bgz     |
|                |
| my.tsv.bgz.tbi |
+----------------+

To build indexes for bior_lookup:

+------------------------------------------+
| bior_index_catalog -d my.tsv.bgz -p rsID |
+------------------------------------------+

Which created the index file:

+-------------------------+
| index/my.rsID.idx.h2.db |
+-------------------------+

You could now perform bior_lookup against the rsID. If you wanted to use
bior_lookup against a different column you would run bior_index_catalog
against that key in the json within the catalog. If you wanted to run it
against CHROM for instance you could run:

+-------------------------------------------+
| bior_index_catalog -d my.tsv.bgz –p CHROM |
+-------------------------------------------+

Or you could do the same thing against “_landmark” since this was
“CHROM” which was mapped to the golden attribute “_landmark”

As an example of bior_lookup on one of these indexed columns:

+-----------------------------------------------------------------------+
| echo "2" \| bior_lookup -p \_landmark -d my.tsv.bgz                   |
|                                                                       |
| ##BIOR=<ID="bior.my",Operation="bior_lookup",DataType="JSON",ShortUni |
| queName="my",Path="/home/mrm3879/tmp/my.tsv.bgz">                     |
|                                                                       |
| #UNKNOWN_1 bior.my                                                    |
|                                                                       |
| 2                                                                     |
| {"CHROM":"2","POS":200,"ID":"222","REF":"C","ALT":"T","rsID":"rs222", |
| "maxBP":200,"_landmark":"2","_minBP":200,"_refAllele":"C","_altAllele |
| ":"T","_id":"rs222","_maxBP":200}                                     |
+-----------------------------------------------------------------------+

Similarly you could build an index on dbSNP.build and do a bior_lookup
against that one as well.

Lastly the column types that can be specified in that config file (ex
my.tab) can be found by running the help on

+----------------------------------------+
| bior_create_config_for_tab_to_tjson –h |
+----------------------------------------+

See this section

+---------------------------------------------------------+
| 2) JsonType is BOOLEAN, NUMBER, or STRING               |
|                                                         |
| This describes the types of values the column can take. |
+---------------------------------------------------------+

\* As a note \_landmark is generally the chromosome but you could create
the indexes on any key within the json object in the catalog. The key
that you use for the index must match the key within the catalog.

Dots in Column Names (and JSON Keys):
-------------------------------------

You can have dots in the indexes, but the keys would have to be nested
within the JSON object in the catalog. The dot has a special meaning in
JSON to denote nested values.

For example, say you have a catalog:

+-----------------------------------------------------------------------+
| $ zcat catalog2.tsv.bgz                                               |
|                                                                       |
| 1 100 100                                                             |
| {"_landmark":"1","_minBP":100,"_maxBP":100,"rsID":"rs1111","VEP":{"Si |
| ftScore":0.01,"Sift":{"Maximum":0.991}}}                              |
|                                                                       |
| 2 200 200                                                             |
| {"_landmark":"2","_minBP":200,"_maxBP":200,"rsID":"rs2222","VEP":{"Si |
| ftScore":0.02,"Sift":{"Maximum":0.992}}}                              |
|                                                                       |
| 3 300 300                                                             |
| {"_landmark":"3","_minBP":300,"_maxBP":300,"rsID":"rs3333","VEP":{"Si |
| ftScore":0.03,"Sift":{"Maximum":0.993}}}                              |
+-----------------------------------------------------------------------+

To reference the 0.992 value within the JSON object, you would have to
refer to it as:

VEP.Sift.Maximum

We then create an index based on "VEP.Sift.Maximum":

+--------------------------------------------------------------+
| $ bior_index_catalog -d catalog2.tsv.bgz -p VEP.Sift.Maximum |
+--------------------------------------------------------------+

Then, if we want to find the value 0.992 within the catalog (with key
"VEP.Sift.Maximum"):

+-----------------------------------------------------------------------+
| $ echo "0.992" \| bior_lookup -d catalog2.tsv.bgz -p VEP.Sift.Maximum |
|                                                                       |
| ##BIOR=<ID="bior.catalog2",Operation="bior_lookup",DataType="JSON",Sh |
| ortUniqueName="catalog2",Path=".../catalog2.tsv.bgz">                 |
|                                                                       |
| #UNKNOWN_1 bior.catalog2                                              |
|                                                                       |
| 0.992                                                                 |
| {"_landmark":"2","_minBP":200,"_maxBP":200,"rsID":"rs2222","VEP":{"Si |
| ftScore":0.02,"Sift":{"Maximum":0.992}}}                              |
+-----------------------------------------------------------------------+

**Note: you can only use a column once!** If you want to make two keys
in the catalog that point to the same column data (such as "Gene" and
"Gene_name"), then duplicate the gene name column in the data and add
another row to your config file - one row for "Gene" pointing to column
6 let's say,

and another row for "Gene_name" pointing to column 7.

To avoid using the period in the key names (assuming those keys are NOT
nested), try the following.

Say we start with a tab-delimited file similar to:

+----------------------------------------------------+
| $ cat catalogCols.tab                              |
|                                                    |
| #CHROM MIN MAX RsId VEP.SiftScore VEP.Sift.Maximum |
|                                                    |
| 1 100 100 rs1111 0.01 0.991                        |
|                                                    |
| 2 200 200 rs2222 0.02 0.992                        |
|                                                    |
| 3 300 300 rs3333 0.03 0.993                        |
+----------------------------------------------------+

We then produce the config file that will help us build the catalog
(utilize the header row / first row, and strip the '#' character off the
front):

+-----------------------------------------------------------------------+
| $ head -n 1 catalogCols.tab \| tr -d "#" \|                           |
| bior_create_config_for_tab_to_tjson > catalogCols.config              |
|                                                                       |
| $ cat catalogCols.config                                              |
|                                                                       |
| 1 CHROM STRING COLUMN . .                                             |
|                                                                       |
| 2 MIN STRING COLUMN . .                                               |
|                                                                       |
| 3 MAX STRING COLUMN . .                                               |
|                                                                       |
| 4 RsId STRING COLUMN . .                                              |
|                                                                       |
| 5 VEP.SiftScore STRING COLUMN . .                                     |
|                                                                       |
| 6 VEP.Sift.Maximum STRING COLUMN . .                                  |
+-----------------------------------------------------------------------+

However, this contains columns with dots in the name.

We want to assign some columns to golden attributes so other BioR
commands will be able to use the genomic position information
(_landmark, \_minBP, \_maxBP).

Plus, we want the VEP.SiftScore and VEP.Sift.Maximum to be numeric
fields.

AND, to every line, we want to add "_type":"variant"

Say we also want to add the same description to every line (though
careful with this as a lot of text on each line can make a catalog very
big) - we add "Description".

So, edit the file and change it to:

+-----------------------------------------------------------------------+
| 1 CHROM STRING COLUMN . \_landmark                                    |
|                                                                       |
| 2 MIN STRING COLUMN . \_minBP                                         |
|                                                                       |
| 3 MAX STRING COLUMN . \_maxBP                                         |
|                                                                       |
| 4 RsId STRING COLUMN . \_id                                           |
|                                                                       |
| 5 VEP_SiftScore NUMBER COLUMN . .                                     |
|                                                                       |
| 6 VEP_Sift_Maximum NUMBER COLUMN . .                                  |
|                                                                       |
| 7 Type STRING LITERAL variant \_type                                  |
|                                                                       |
| 8 Description STRING LITERAL A single-nucleotide polymorphism (SNP) . |
+-----------------------------------------------------------------------+

Let's see what the catalog would like if we were to build it now:

+-----------------------------------------------------------------------+
| $ cat catalogCols.tab \| bior_tab_to_tjson -c catalogCols.config      |
|                                                                       |
| ##BIOR=<ID="bior.ToTJson",Operation="bior_tab_to_tjson",DataType="JSO |
| N",ShortUniqueName="ToTJson">                                         |
|                                                                       |
| #CHROM MIN MAX RsId VEP.SiftScore VEP.Sift.Maximum bior.ToTJson       |
|                                                                       |
| 1 100 100 rs1111 0.01 0.991                                           |
| {"CHROM":"1","MIN":"100","MAX":"100","RsId":"rs1111","VEP_SiftScore": |
| 0.01,"VEP_Sift_Maximum":0.991,"Type":"variant","Description":"A       |
| single-nucleotide polymorphism                                        |
| (SNP)","_landmark":"1","_minBP":"100","_maxBP":"100","_id":"rs1111"," |
| _type":"variant"}                                                     |
|                                                                       |
| 2 200 200 rs2222 0.02 0.992                                           |
| {"CHROM":"2","MIN":"200","MAX":"200","RsId":"rs2222","VEP_SiftScore": |
| 0.02,"VEP_Sift_Maximum":0.992,"Type":"variant","Description":"A       |
| single-nucleotide polymorphism                                        |
| (SNP)","_landmark":"2","_minBP":"200","_maxBP":"200","_id":"rs2222"," |
| _type":"variant"}                                                     |
|                                                                       |
| 3 300 300 rs3333 0.03 0.993                                           |
| {"CHROM":"3","MIN":"300","MAX":"300","RsId":"rs3333","VEP_SiftScore": |
| 0.03,"VEP_Sift_Maximum":0.993,"Type":"variant","Description":"A       |
| single-nucleotide polymorphism                                        |
| (SNP)","_landmark":"3","_minBP":"300","_maxBP":"300","_id":"rs3333"," |
| _type":"variant"}                                                     |
+-----------------------------------------------------------------------+

This looks pretty good, but we want the final catalog to have only the
chrom, min, max, and JSON fields. So let's cut out the extras (columns
4,5,6) and save it to a bgzip'd catalog:

+-----------------------------------------------------------------------+
| $ cat catalogCols.tab \| bior_tab_to_tjson -c catalogCols.config \|   |
| cut -f 1,2,3,7 \| bgzip -c > catalogCols.tsv.bgz                      |
+-----------------------------------------------------------------------+

Now let's look at the catalog:

+-----------------------------------------------------------------------+
| $ zcat catalogCols.tsv.bgz                                            |
|                                                                       |
| ##BIOR=<ID="bior.ToTJson",Operation="bior_tab_to_tjson",DataType="JSO |
| N",ShortUniqueName="ToTJson">                                         |
|                                                                       |
| #CHROM MIN MAX bior.ToTJson                                           |
|                                                                       |
| 1 100 100                                                             |
| {"CHROM":"1","MIN":"100","MAX":"100","RsId":"rs1111","VEP_SiftScore": |
| 0.01,"VEP_Sift_Maximum":0.991,"Type":"variant","Description":"A       |
| single-nucleotide polymorphism                                        |
| (SNP)","_landmark":"1","_minBP":"100","_maxBP":"100","_id":"rs1111"," |
| _type":"variant"}                                                     |
|                                                                       |
| 2 200 200                                                             |
| {"CHROM":"2","MIN":"200","MAX":"200","RsId":"rs2222","VEP_SiftScore": |
| 0.02,"VEP_Sift_Maximum":0.992,"Type":"variant","Description":"A       |
| single-nucleotide polymorphism                                        |
| (SNP)","_landmark":"2","_minBP":"200","_maxBP":"200","_id":"rs2222"," |
| _type":"variant"}                                                     |
|                                                                       |
| 3 300 300                                                             |
| {"CHROM":"3","MIN":"300","MAX":"300","RsId":"rs3333","VEP_SiftScore": |
| 0.03,"VEP_Sift_Maximum":0.993,"Type":"variant","Description":"A       |
| single-nucleotide polymorphism                                        |
| (SNP)","_landmark":"3","_minBP":"300","_maxBP":"300","_id":"rs3333"," |
| _type":"variant"}                                                     |
+-----------------------------------------------------------------------+

Bingo!

There is some duplication on each line (such as "CHROM" and "_landmark",
or "RsId" and "_id", but we tend to leave these alone as most users want
to retain the original names of items from data sources, and BioR wants
the golden attributes (those beginning with "_") for interoperability.

Again, we can build indexes on each JSON key. Say we want an index for
"RsId" / "_id", then we do a lookup by a known rsID:

+-----------------------------------------------------------------------+
| $ bior_index_catalog -d catalogCols.tsv.bgz -p RsId                   |
|                                                                       |
| $ bior_index_catalog -d catalogCols.tsv.bgz -p \_id                   |
|                                                                       |
| $ echo "rs2222" \| bior_lookup -d catalogCols.tsv.bgz -p RsId         |
|                                                                       |
| ##BIOR=<ID="bior.catalogCols",Operation="bior_lookup",DataType="JSON" |
| ,ShortUniqueName="catalogCols",Path=".../catalogCols.tsv.bgz">        |
|                                                                       |
| #UNKNOWN_1 bior.catalogCols                                           |
|                                                                       |
| rs2222                                                                |
| {"CHROM":"2","MIN":"200","MAX":"200","RsId":"rs2222","VEP_SiftScore": |
| 0.02,"VEP_Sift_Maximum":0.992,"Type":"variant","Description":"A       |
| single-nucleotide polymorphism                                        |
| (SNP)","_landmark":"2","_minBP":"200","_maxBP":"200","_id":"rs2222"," |
| _type":"variant"}                                                     |
+-----------------------------------------------------------------------+

Indexing the Data for Coordinate Based Search
---------------------------------------------

For positional search, BioR supports indexing using Tabix.  Tabix/bgzip
should be installed in the RCF environment.  First, compress the raw
input.  Assuming it is sorted:

+-------------------+
| $ bgzip dbSNP.tsv |
+-------------------+

Then run the tabix command:

+---------------------------------------+
| $ tabix -s 1 -b 2 -e 3 dbSNP.tsv.gz & |
+---------------------------------------+

That's it! you can now use your custom catalog as a database in BioR
commands (e.g. bior_overlap.sh -d /path/to/your/database.tsv.gz).

Hints on Creating Indexes on Custom Catalogs
--------------------------------------------

In addition to coordinate based search, users may also want to search a
custom catalog based on IDs. The process is exactly the same as in
indexing a catalog described earlier in this document, but there are
some gotcha’s that users need to be aware of.

1. The catalog structure will not automatically join data. This can be
   frustrating as the data provider may not give the data to you in a
   desirable form (e.g. you may want to know everything the data
   provider knows about a gene, but they may have their data organized
   by variant or drug) so you will have to ‘flip’ the data around so
   that all information about a gene can be provided to users of your
   catalog. The BioR team has done this many times, and for Java
   programmers, there is a robust library (BioR-Catalog) and examples to
   help in the publication of new-complex catalogs.

2. The BioR indexer command currently does not tolerate duplicate keys,
   so while duplicate keys can be in the data itself, you can’t index on
   those keys. Running bior_index_catalog with logging enabled will help
   to ensure the keys you would like to index on are valid. To index
   multiple ways simultaneously, multiple catalogs need to be created

3. Regardless of what tools are used to construct the JSON column, it
   must validate as proper JSON. Use jslint to validate:
   `http://jsonlint.com/ <http://jsonlint.com/>`__

4. JSON should not contain fields that are empty. While adding period
   “.” As the value for a given key will work, it wastes space and
   consumes additional CPU resources so is not recommended.

  

 