
Get all Variants in a Gene
--------------------------

Lets do something useful -- say we wanted all genetic variants in VCF
format that overlap the BRCA1 gene from dbSNP. This section will
illustrate how to use BioR to rapidly build a program that does just
that. BioR is executed at the Linux/UNIX command line, so any command
that is available at the command line can be used with BioR (grep, cut,
sed, awk, perl, …). Lets start with the echo command to find BRCA1 in
the gene catalog.

+-----------------------------------------------------------------------+
| $ echo "BRCA1" \| bior_lookup -p gene -d                              |
| $bior/NCBIGene/GRCh37_p10/genes.tsv.bgz \| bior_pretty_print          |
|                                                                       |
| # COLUMN NAME COLUMN VALUE                                            |
|                                                                       |
| - ----------- ------------                                            |
|                                                                       |
| 1 UNKNOWN_1 BRCA1                                                     |
|                                                                       |
| 2 LookupPipe {                                                        |
|                                                                       |
| "_type": "gene",                                                      |
|                                                                       |
| "_landmark": "17",                                                    |
|                                                                       |
| "_strand": "-",                                                       |
|                                                                       |
| "_minBP": 41196312,                                                   |
|                                                                       |
| "_maxBP": 41277500,                                                   |
|                                                                       |
| "gene": "BRCA1",                                                      |
|                                                                       |
| "gene_synonym": "BRCAI; BRCC1; BROVCA1; IRIS; PNCA4; PPP1R53; PSCP;   |
| RNF53",                                                               |
|                                                                       |
| "note": "breast cancer 1, early onset; Derived by automated           |
| computational analysis using gene prediction method: BestRefseq.",    |
|                                                                       |
| "GeneID": "672",                                                      |
|                                                                       |
| "HGNC": "1100",                                                       |
|                                                                       |
| "HPRD": "00218",                                                      |
|                                                                       |
| "MIM": "113705"                                                       |
|                                                                       |
| }                                                                     |
|                                                                       |
| $                                                                     |
+-----------------------------------------------------------------------+

The UNIX pipe (‘|’) allows you to stream the output of one command to
the next. In this example, echo prints BRCA1 to the screen. bior_lookup
uses this ID to find the entry in the gene catalog with the key gene and
value ‘BRCA1’. Now we have the genomic coordinates for BRCA1. Lets use
these positions to find all catalog entries in dbSNP that are between
41196312 and 41277500 on chromosome 17.

+-----------------------------------------------------------------------+
| $ echo "BRCA1" \| bior_lookup -p gene -d                              |
| $bior/NCBIGene/GRCh37_p10/genes.tsv.bgz \| bior_overlap -d            |
| $bior/dbSNP/137/00-All_GRCh37.tsv.bgz \| bior_pretty_print            |
|                                                                       |
| # COLUMN NAME COLUMN VALUE                                            |
|                                                                       |
| - ----------- ------------                                            |
|                                                                       |
| 1 UNKNOWN_1 BRCA1                                                     |
|                                                                       |
| 2 LookupPipe {                                                        |
|                                                                       |
| "_type": "gene",                                                      |
|                                                                       |
| "_landmark": "17",                                                    |
|                                                                       |
| "_strand": "-",                                                       |
|                                                                       |
| "_minBP": 41196312,                                                   |
|                                                                       |
| "_maxBP": 41277500,                                                   |
|                                                                       |
| "gene": "BRCA1",                                                      |
|                                                                       |
| "gene_synonym": "BRCAI; BRCC1; BROVCA1; IRIS; PNCA4; PPP1R53; PSCP;   |
| RNF53",                                                               |
|                                                                       |
| "note": "breast cancer 1, early onset; Derived by automated           |
| computational analysis using gene prediction method: BestRefseq.",    |
|                                                                       |
| "GeneID": "672",                                                      |
|                                                                       |
| "HGNC": "1100",                                                       |
|                                                                       |
| "HPRD": "00218",                                                      |
|                                                                       |
| "MIM": "113705"                                                       |
|                                                                       |
| }                                                                     |
|                                                                       |
| 3 OverlapPipe {                                                       |
|                                                                       |
| "CHROM": "17",                                                        |
|                                                                       |
| "POS": "41196363",                                                    |
|                                                                       |
| "ID": "rs8176320",                                                    |
|                                                                       |
| "REF": "C",                                                           |
|                                                                       |
| "ALT": "T",                                                           |
|                                                                       |
| "QUAL": ".",                                                          |
|                                                                       |
| "FILTER": ".",                                                        |
|                                                                       |
| "INFO": {                                                             |
|                                                                       |
| "RSPOS": 41196363,                                                    |
|                                                                       |
| "RV": true,                                                           |
|                                                                       |
| "GMAF": 0.0050,                                                       |
|                                                                       |
| "dbSNPBuildID": 117,                                                  |
|                                                                       |
| "SSR": 0,                                                             |
|                                                                       |
| "SAO": 0,                                                             |
|                                                                       |
| "VP": "050000800201040517000100",                                     |
|                                                                       |
| "GENEINFO": "BRCA1:672",                                              |
|                                                                       |
| "WGT": 1,                                                             |
|                                                                       |
| "VC": "SNV",                                                          |
|                                                                       |
| "REF": true,                                                          |
|                                                                       |
| "U3": true,                                                           |
|                                                                       |
| "VLD": true,                                                          |
|                                                                       |
| "HD": true,                                                           |
|                                                                       |
| "GNO": true,                                                          |
|                                                                       |
| "KGPhase1": true,                                                     |
|                                                                       |
| "KGPROD": true,                                                       |
|                                                                       |
| "OTHERKG": true,                                                      |
|                                                                       |
| "PH3": true                                                           |
|                                                                       |
| },                                                                    |
|                                                                       |
| "_id": "rs8176320",                                                   |
|                                                                       |
| "_type": "variant",                                                   |
|                                                                       |
| "_landmark": "17",                                                    |
|                                                                       |
| "_refAllele": "C",                                                    |
|                                                                       |
| "_altAlleles": [                                                      |
|                                                                       |
| "T"                                                                   |
|                                                                       |
| ],                                                                    |
|                                                                       |
| "_minBP": 41196363,                                                   |
|                                                                       |
| "_maxBP": 41196363                                                    |
|                                                                       |
| }                                                                     |
|                                                                       |
| $                                                                     |
+-----------------------------------------------------------------------+

This command shows the first match in dbSNP that overlaps the BRCA1 gene
according to the NCBI annotation. The version of dbSNP used to publish
the catalog was a VCF file, therefore many fields from the VCF standard
are represented in the JSON. A combination of the UNIX cut command and
bior_drill can quickly extract a VCF file. When trying this example,
decompose the commands and use them one at a time to understand what
each command is doing.

+-----------------------------------------------------------------------+
| $ echo "BRCA1" \| bior_lookup -p gene -d                              |
| $bior/NCBIGene/GRCh37_p10/genes.tsv.bgz \| bior_overlap -d            |
| $bior/dbSNP/137/00-All_GRCh37.tsv.bgz \| bior_drill -p CHROM -p POS   |
| \| cut -f 1,3,4 \| head -10                                           |
|                                                                       |
| ##BIOR=<ID="bior.gene37p10",Operation="bior_lookup",DataType="JSON",S |
| hortUniqueName="gene37p10",Source="NCBIGene",Description="NCBI's      |
| Gene Annotation directly from the gbs                                 |
| file",Version="37p10",Build="GRCh37.p10",Path="/data5/bsi/catalogs/bi |
| or/v1/NCBIGene/GRCh37_p10/genes.tsv.bgz">                             |
|                                                                       |
| ##BIOR=<ID="bior.dbSNP137",Operation="bior_overlap",DataType="JSON",S |
| hortUniqueName="dbSNP137",Source="dbSNP",Description="NCBI's          |
| dbSNP Variant                                                         |
| Database",Version="137",Build="GRCh37.p5",Path="/data5/bsi/catalogs/b |
| ior/v1/dbSNP/137/00-All_GRCh37.tsv.bgz">                              |
|                                                                       |
| ##BIOR=<ID="bior.dbSNP137.CHROM",Operation="bior_drill",Field="CHROM" |
| ,DataType="String",Number="1",FieldDescription="Chromosome.           |
| (VCF                                                                  |
| field)",ShortUniqueName="dbSNP137",Source="dbSNP",Description="NCBI's |
| dbSNP Variant                                                         |
| Database",Version="137",Build="GRCh37.p5",Path="/data5/bsi/catalogs/b |
| ior/v1/dbSNP/137/00-All_GRCh37.tsv.bgz">                              |
|                                                                       |
| ##BIOR=<ID="bior.dbSNP137.POS",Operation="bior_drill",Field="POS",Dat |
| aType="Integer",Number="1",FieldDescription="The                      |
| reference position, with the 1st base having position 1. (VCF         |
| field)",ShortUniqueName="dbSNP137",Source="dbSNP",Description="NCBI's |
| dbSNP Variant                                                         |
| Database",Version="137",Build="GRCh37.p5",Path="/data5/bsi/catalogs/b |
| ior/v1/dbSNP/137/00-All_GRCh37.tsv.bgz">                              |
|                                                                       |
| #UNKNOWN_1 bior.dbSNP137.CHROM bior.dbSNP137.POS                      |
|                                                                       |
| BRCA1 17 41196363                                                     |
|                                                                       |
| BRCA1 17 41196368                                                     |
|                                                                       |
| BRCA1 17 41196372                                                     |
|                                                                       |
| BRCA1 17 41196403                                                     |
|                                                                       |
| BRCA1 17 41196408                                                     |
+-----------------------------------------------------------------------+

The result: a simple VCF-like file constructed for all variants in the
BRCA1 gene! There are a few small fixes that will need to be made to
make it truly VCF-compliant, and this quickstart glosses over many
features such as the metadata and headers. These and many other issues
will be covered in more detail in the following sections.
