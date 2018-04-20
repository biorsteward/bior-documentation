
bior_drill
~~~~~~~~~~

Drill allows you to extract one or more values from one JSON column. As
a simple example, say we have one JSON column with one value. By
default, it will remove the JSON column and insert a new column that has
just the value you want to drill. Note the ##BIOR metadata header that
is created for the column: it added the prefix "bior." to the column
name to distinguish it as a BioR column

+-----------------------------------------------------------------------+
| $ cat my.tsv                                                          |
|                                                                       |
| #myJson                                                               |
|                                                                       |
| {"aString":"hi"}                                                      |
|                                                                       |
| $ cat my.tsv \| bior_drill -p aString                                 |
|                                                                       |
| ##BIOR=<ID="bior.myJson.aString",Operation="bior_drill",DataType="Str |
| ing",ShortUniqueName="",Path="">                                      |
|                                                                       |
| #bior.myJson.aString                                                  |
|                                                                       |
| hi                                                                    |
+-----------------------------------------------------------------------+

You can also keep the JSON column (and it will be re-appended to the end
of the line so you can continue to perform operations on the JSON
column):

+-----------------------------------------------------------------------+
| **$ cat my.tsv \| bior_drill -p aString -k**                          |
|                                                                       |
| ##BIOR=<ID="bior.myJson.aString",Operation="bior_drill",DataType="Str |
| ing",ShortUniqueName="",Path="">                                      |
|                                                                       |
| #bior.myJson.aString myJson                                           |
|                                                                       |
| hi {"aString":"hi"}                                                   |
+-----------------------------------------------------------------------+

If bior_overlap or bior_same_variant or bior_lookup were run first
before drilling any of the fields, there is a lot more metadata that is
added about the catalog and about each field that is drilled

+-----------------------------------------------------------------------+
| **ctg=/data5/bsi/catalogs/bior/v1/1000_genomes/20110521/ALL.wgs.phase |
| 1_release_v3.20101123.snps_indels_sv.sites_GRCh37.tsv.bgz**           |
|                                                                       |
| **$ echo "{_landmark:1,_minBP:10583,_maxBP:10583}" \| bior_overlap -d |
| $ctg \| bior_drill -p ID**                                            |
|                                                                       |
| ##BIOR=<ID="bior.1kG_3",Operation="bior_overlap",DataType="JSON",Shor |
| tUniqueName="1kG_3",Source="1000                                      |
| Genomes",Description="1000 Genomes Project goal is to find most       |
| genetic variants that have frequencies of at least 1% in the          |
| populations                                                           |
| studied.",Version="3",Build="GRCh37",Path="/data5_mike/bsi/catalogs/b |
| ior/v1/1000_genomes/20110521/ALL.wgs.phase1_release_v3.20101123.snps_ |
| indels_sv.sites_GRCh37.tsv.bgz">                                      |
|                                                                       |
| ##BIOR=<ID="bior.1kG_3.ID",Operation="bior_drill",Field="ID",DataType |
| ="String",Number=".",FieldDescription="Semi-colon                     |
| separated list of unique identifiers. If this is a dbSNP variant, the |
| rs number(s) should be used. (VCF                                     |
| field)",ShortUniqueName="1kG_3",Source="1000                          |
| Genomes",Description="1000 Genomes Project goal is to find most       |
| genetic variants that have frequencies of at least 1% in the          |
| populations                                                           |
| studied.",Version="3",Build="GRCh37",Path="/data5_mike/bsi/catalogs/b |
| ior/v1/1000_genomes/20110521/ALL.wgs.phase1_release_v3.20101123.snps_ |
| indels_sv.sites_GRCh37.tsv.bgz",Delimiter="|">                        |
|                                                                       |
| #UNKNOWN_1 bior.1kG_3.ID                                              |
|                                                                       |
| {_landmark:1,_minBP:10583,_maxBP:10583} rs58108140                    |
+-----------------------------------------------------------------------+

You can use the "echo" command instead of "cat" to inject JSON into the
STDIN stream. Here we will also drill out multiple fields at once
(##BIOR headers removed from many of the following cmds for brevity, and
shortened to "##BIOR….")

+-----------------------------------------------------------------------+
| **$ echo                                                              |
| "{'aString':'hi','aFloat':0.034,'aBool':false,'anInt':34,'aDot':'.'}" |
| \| bior_drill -p aString -p aFloat -p aBool -p anInt -p aDot**        |
|                                                                       |
| ##BIOR....                                                            |
|                                                                       |
| #bior.#UNKNOWN_1.aString bior.#UNKNOWN_1.aFloat bior.#UNKNOWN_1.aBool |
| bior.#UNKNOWN_1.anInt bior.#UNKNOWN_1.aDot                            |
|                                                                       |
| hi 0.034 false 34 .                                                   |
+-----------------------------------------------------------------------+

Drill from a middle column (not the last one)

**NOTE**: This shows how to use tabs and newlines with "echo" command to
add columns and header rows (or multiple data rows)

**NOTE**: The drilled column is removed by default and all following
columns shifted left one position

+-----------------------------------------------------------------------+
| **$ echo -e "#Id\tMyJson\tRef\tAlt\nRs1\t{'A':1}\tA\tC"**             |
|                                                                       |
| #Id MyJson Ref Alt                                                    |
|                                                                       |
| Rs1 {'A':1} A C                                                       |
|                                                                       |
| **$ echo -e "#Id\tMyJson\tRef\tAlt\nRs1\t{'A':1}\tA\tC" \| bior_drill |
| -c 2 -p A**                                                           |
|                                                                       |
| ##BIOR....                                                            |
|                                                                       |
| #Id Ref Alt bior.MyJson.A                                             |
|                                                                       |
| Rs1 A C 1                                                             |
+-----------------------------------------------------------------------+

Arrays and nulls are handled specially.

-  Nulls and empty arrays when drilled are denoted as dots to maintain
   placeholders in an otherwise empty column

-  If a field does not appear in the line, its drilled value will be a
   dot (ex: the "nonexistent" field specified below)

-  Single-value arrays have the brackets ("[", "]") removed and only the
   value inserted

-  Multi-value arrays have the brackets removed, and values are
   separated by pipes by default (ex: "anArray":["A","B","C"] drilled
   becomes "A|B|C").

   -  If the separator occurs in one of the values, an error occurs

   -  Change the default separator by using the -d flag

   -  Skip any nulls or dots in an array using the -s flag. **Warning**:
      some arrays keep nulls or dots to maintain order to correlate
      other values from another array within the same data source, so
      order and position may be important.

-  **NOTE**: BioR v4.3.0 changed the way arrays are drilled. Prior to
   that version, "{'key':[1,2,3]}" would drill to "[1,2,3]", instead of
   the "1|2|3" as is now done. This change allows easier parsing of
   drilled values

-  **NOTE**: BioR v4.2.0 fixed a bug in the way null values were handled
   when drilling. Previously, instead of inserting a dot as a
   placeholder for that column, the column was missing entirely, which
   shifted all columns left one position.

+-----------------------------------------------------------------------+
| **$ echo                                                              |
| "{'aNull':null,'anArray':['A','B','C'],'anArraySingleVal':['D'],'anAr |
| rayEmpty':[],'anArrayNull':null}"                                     |
| \| bior_drill -p aNull -p anArray -p anArraySingleVal -p anArrayEmpty |
| -p anArrayNull -p nonexistent**                                       |
|                                                                       |
| ##BIOR....                                                            |
|                                                                       |
| #bior.#UNKNOWN_1.aNull bior.#UNKNOWN_1.anArray                        |
| bior.#UNKNOWN_1.anArraySingleVal bior.#UNKNOWN_1.anArrayEmpty         |
| bior.#UNKNOWN_1.anArrayNull bior.#UNKNOWN_1.nonexistent               |
|                                                                       |
| . A|B|C D . . .                                                       |
|                                                                       |
| **# Error if separator occurs in value**                              |
|                                                                       |
| **$ echo "{'anArray':['A|B','B','C']}" \| bior_drill -p anArray**     |
|                                                                       |
| Application error bior_drill:                                         |
|                                                                       |
| : Error: the delimiter '|' was found within one of the array values   |
| that was drilled: 'A|B'                                               |
|                                                                       |
| Execute bior_drill with logging enabled using the -l or --log option  |
|                                                                       |
| Command executed bior_drill -p anArray                                |
|                                                                       |
| **# Override the default separator**                                  |
|                                                                       |
| **$ echo "{'anArray':['A|B','B','C']}" \| bior_drill -p anArray -d    |
| "##"**                                                                |
|                                                                       |
| ##BIOR=....                                                           |
|                                                                       |
| #bior.#UNKNOWN_1.anArray                                              |
|                                                                       |
| A|B##B##C                                                             |
|                                                                       |
| **# Skip any nulls and dots within an array. NOTE: Empty strings are  |
| still added**                                                         |
|                                                                       |
| **$ echo "{'anArray':['A',null,'.','B',null,'C',null,'D','']}" \|     |
| bior_drill -p anArray -s**                                            |
|                                                                       |
| ##BIOR=....                                                           |
|                                                                       |
| #bior.#UNKNOWN_1.anArray                                              |
|                                                                       |
| A|B|C|D\|                                                             |
+-----------------------------------------------------------------------+
