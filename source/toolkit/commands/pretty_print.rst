

Pretty Print
------------

Data in the 4\ :sup:`th` column of a catalog is stored as JSON. JSON can
be deeply nested and hard to read if it is all smashed into one line.
BioR has a command bior_pretty_print that can make reading JSON text
easier. Take the earlier example and replace less with
bior_pretty_print:

+-----------------------------------------------------------------------+
| $ zcat $bior/NCBIGene/GRCh37_p10/genes.tsv.bgz \| bior_pretty_print   |
|                                                                       |
| # COLUMN NAME COLUMN VALUE                                            |
|                                                                       |
| - ----------- ------------                                            |
|                                                                       |
| 1 UNKNOWN_1 1                                                         |
|                                                                       |
| 2 #UNKNOWN_2 10954                                                    |
|                                                                       |
| 3 #UNKNOWN_3 11507                                                    |
|                                                                       |
| 4 #UNKNOWN_4 {                                                        |
|                                                                       |
| "_type": "gene",                                                      |
|                                                                       |
| "_landmark": "1",                                                     |
|                                                                       |
| "_strand": "+",                                                       |
|                                                                       |
| "_minBP": 10954,                                                      |
|                                                                       |
| "_maxBP": 11507,                                                      |
|                                                                       |
| "gene": "LOC100506145",                                               |
|                                                                       |
| "note": "Derived by automated computational analysis using gene       |
| prediction method: GNOMON. Supporting evidence includes similarity    |
| to: 1 Protein",                                                       |
|                                                                       |
| "pseudo": "",                                                         |
|                                                                       |
| "GeneID": "100506145"                                                 |
|                                                                       |
| }                                                                     |
|                                                                       |
| $                                                                     |
+-----------------------------------------------------------------------+

Use –r to specify the row to pretty print. This is very useful when
handling sparse data, where the values for columns you are interested in
do not appear on every line. In JSON if there is no value for a given
key, the key is not shown (instead of reporting NULL), so you may need
to hunt around in the dataset a bit to find keys of interest.
