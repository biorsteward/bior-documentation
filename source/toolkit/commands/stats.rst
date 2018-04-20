
bior_catalog_stats
~~~~~~~~~~~~~~~~~~

Generates the following catalog statistics:

-  **Character stats** - Tracks frequency of characters in a column.
   These statistics are stored in a file for each catalog column with a
   file named <COLUMN_NAME>_char_stats.txt

-  **Value stats** - Tracks column values and frequency (up to the given
   sampling-size). These statistics are stored in a file for each
   catalog column with a file named <COLUMN_NAME>_value_stats.txt

From this we can more easily determine if fields should be Strings (they
have non-numeric characters in them (or dots)), Floats (only numbers and
dots), Integers (only numbers, no dots), Booleans (only the letters that
make up "true" and "false"), etc.

+-----------------------------------------------------------------------+
| **ctg=/data5/bsi/catalogs/bior/v1/ClinVar/20160515_GRCh37/variants_no |
| dups.v1/macarthur-lab_xml_txt.tsv.bgz**                               |
|                                                                       |
| **# Generate a report for the ClinVar catalog with 100 sample         |
| values**                                                              |
|                                                                       |
| **$ bior_catalog_stats -d $ctg -o . -n 100**                          |
|                                                                       |
| **# The reports generated: (cutting out some unnecessary columns to   |
| the left)**                                                           |
|                                                                       |
| **$ ls -l \| cut -f 7- -d" "**                                        |
|                                                                       |
| 6698 Apr 24 17:04 all_pmids_stats.txt                                 |
|                                                                       |
| 15209 Apr 24 17:04 all_submitters_stats.txt                           |
|                                                                       |
| 18825 Apr 24 17:04 all_traits_stats.txt                               |
|                                                                       |
| 5328 Apr 24 17:04 alt_stats.txt                                       |
|                                                                       |
| 2641 Apr 24 17:04 chrom_stats.txt                                     |
|                                                                       |
| 8878 Apr 24 17:04 clinical_significance_stats.txt                     |
|                                                                       |
| 844 Apr 24 17:04 conflicted_stats.txt                                 |
|                                                                       |
| 5670 Apr 24 17:04 endPos_stats.txt                                    |
|                                                                       |
| 8460 Apr 24 17:04 hgvs_c_stats.txt                                    |
|                                                                       |
| 10325 Apr 24 17:04 hgvs_p_stats.txt                                   |
|                                                                       |
| 5668 Apr 24 17:04 measureset_id_stats.txt                             |
|                                                                       |
| 1194 Apr 24 17:04 mut_stats.txt                                       |
|                                                                       |
| 836 Apr 24 17:04 pathogenic_stats.txt                                 |
|                                                                       |
| 5683 Apr 24 17:04 pos_stats.txt                                       |
|                                                                       |
| 5264 Apr 24 17:04 ref_stats.txt                                       |
|                                                                       |
| 2986 Apr 24 17:04 review_status_stats.txt                             |
|                                                                       |
| 8642 Apr 24 17:04 symbol_stats.txt                                    |
|                                                                       |
| **# Looking at the first row in the catalog, we see the fields that   |
| are reported on above (minus the golden attributes like "_landmark",  |
| "_id", etc)**                                                         |
|                                                                       |
| **$ zcat $ctg \| head -1 \| bior_pretty_print**                       |
|                                                                       |
| # COLUMN NAME COLUMN VALUE                                            |
|                                                                       |
| - ----------- ------------                                            |
|                                                                       |
| 1 #UNKNOWN_1 1                                                        |
|                                                                       |
| 2 #UNKNOWN_2 949523                                                   |
|                                                                       |
| 3 #UNKNOWN_3 949523                                                   |
|                                                                       |
| 4 #UNKNOWN_4 {                                                        |
|                                                                       |
| "chrom": "1",                                                         |
|                                                                       |
| "pos": 949523,                                                        |
|                                                                       |
| "ref": "C",                                                           |
|                                                                       |
| "alt": "T",                                                           |
|                                                                       |
| "mut": "ALT",                                                         |
|                                                                       |
| "measureset_id": 183381,                                              |
|                                                                       |
| "symbol": "ISG15",                                                    |
|                                                                       |
| "clinical_significance": "Pathogenic",                                |
|                                                                       |
| "review_status": "no assertion criteria provided",                    |
|                                                                       |
| "hgvs_c": "NM_005101.3:c.163C\u003eT",                                |
|                                                                       |
| "hgvs_p": "NP_005092.1:p.Gln55Ter",                                   |
|                                                                       |
| "all_submitters": "OMIM",                                             |
|                                                                       |
| "all_traits": "Immunodeficiency 38;IMMUNODEFICIENCY 38 WITH BASAL     |
| GANGLIA CALCIFICATION",                                               |
|                                                                       |
| "all_pmids": "25307056",                                              |
|                                                                       |
| "pathogenic": 1,                                                      |
|                                                                       |
| "conflicted": 0,                                                      |
|                                                                       |
| "endPos": 949523,                                                     |
|                                                                       |
| "_altAlleles": [                                                      |
|                                                                       |
| "T"                                                                   |
|                                                                       |
| ],                                                                    |
|                                                                       |
| "_id": ".",                                                           |
|                                                                       |
| "_landmark": "1",                                                     |
|                                                                       |
| "_minBP": 949523,                                                     |
|                                                                       |
| "_refAllele": "C",                                                    |
|                                                                       |
| "_maxBP": 949523                                                      |
|                                                                       |
| }                                                                     |
|                                                                       |
| **# Here, the "COLUMN METADATA" is info pulled from the catalog's     |
| columns.tsv file**                                                    |
|                                                                       |
| **# Characters stats are obtained by crawling each line in the        |
| catalog and determining what characters appear with what frequency**  |
|                                                                       |
| $ cat chrom_stats.txt                                                 |
|                                                                       |
| # COLUMN METADATA                                                     |
|                                                                       |
| COLUMN chrom                                                          |
|                                                                       |
| TYPE String                                                           |
|                                                                       |
| COUNT 1                                                               |
|                                                                       |
| DESCRIPTION Chromosome                                                |
|                                                                       |
| # CHARACTER STATS                                                     |
|                                                                       |
| Total Lines in file: 117329                                           |
|                                                                       |
| Lines that had column: 117329 (100.0%)                                |
|                                                                       |
| Total column value chars: 173129                                      |
|                                                                       |
| NON-Alphanumeric                                                      |
|                                                                       |
| CHARACTER LINE_FREQ LINE_% CHAR_FREQ CHAR_%                           |
|                                                                       |
| Alphanumeric                                                          |
|                                                                       |
| CHARACTER LINE_FREQ LINE_% CHAR_FREQ CHAR_%                           |
|                                                                       |
| 0 5802 4.9% 5802 3.4%                                                 |
|                                                                       |
| 1 60443 51.5% 68477 39.6%                                             |
|                                                                       |
| 2 23790 20.3% 25435 14.7%                                             |
|                                                                       |
| 3 11383 9.7% 11383 6.6%                                               |
|                                                                       |
| 4 5572 4.7% 5572 3.2%                                                 |
|                                                                       |
| 5 9576 8.2% 9576 5.5%                                                 |
|                                                                       |
| 6 9870 8.4% 9870 5.7%                                                 |
|                                                                       |
| 7 15207 13.0% 15207 8.8%                                              |
|                                                                       |
| 8 4372 3.7% 4372 2.5%                                                 |
|                                                                       |
| 9 8959 7.6% 8959 5.2%                                                 |
|                                                                       |
| M 81 0.1% 81 0.0%                                                     |
|                                                                       |
| X 8365 7.1% 8365 4.8%                                                 |
|                                                                       |
| Y 30 0.0% 30 0.0%                                                     |
|                                                                       |
| # VALUE STATS                                                         |
|                                                                       |
| VALUE_FREQ VALUE_% VALUE                                              |
|                                                                       |
| 13750 11.7% 2                                                         |
|                                                                       |
| 9227 7.9% 17                                                          |
|                                                                       |
| 8365 7.1% X                                                           |
|                                                                       |
| 8034 6.8% 11                                                          |
|                                                                       |
| 7987 6.8% 1                                                           |
|                                                                       |
| 6351 5.4% 16                                                          |
|                                                                       |
| 6229 5.3% 5                                                           |
|                                                                       |
| 5980 5.1% 7                                                           |
|                                                                       |
| 5829 5.0% 13                                                          |
|                                                                       |
| 5554 4.7% 3                                                           |
|                                                                       |
| 5469 4.7% 12                                                          |
|                                                                       |
| 4608 3.9% 9                                                           |
|                                                                       |
| 4351 3.7% 19                                                          |
|                                                                       |
| 4103 3.5% 10                                                          |
|                                                                       |
| 3519 3.0% 6                                                           |
|                                                                       |
| 3347 2.9% 15                                                          |
|                                                                       |
| 3007 2.6% 14                                                          |
|                                                                       |
| 2861 2.4% 8                                                           |
|                                                                       |
| 2565 2.2% 4                                                           |
|                                                                       |
| 1699 1.4% 20                                                          |
|                                                                       |
| 1645 1.4% 22                                                          |
|                                                                       |
| 1511 1.3% 18                                                          |
|                                                                       |
| 1227 1.0% 21                                                          |
|                                                                       |
| 81 0.1% M                                                             |
|                                                                       |
| 30 0.0% Y                                                             |
|                                                                       |
| **# If we look at the "VALUE STATS" from                              |
| "clinical_significance_stats.txt for example, we can see what the top |
| 10 most frequently occurring values are:**                            |
|                                                                       |
| **$ grep -A 12 "VALUE STATS" clinical_significance_stats.txt**        |
|                                                                       |
| # VALUE STATS                                                         |
|                                                                       |
| VALUE_FREQ VALUE_% VALUE                                              |
|                                                                       |
| 31677 27.0% Pathogenic                                                |
|                                                                       |
| 30630 26.1% Uncertain significance                                    |
|                                                                       |
| 12700 10.8% Benign                                                    |
|                                                                       |
| 12597 10.7% not provided                                              |
|                                                                       |
| 10660 9.1% Likely benign                                              |
|                                                                       |
| 6197 5.3% Likely pathogenic                                           |
|                                                                       |
| 2293 2.0% Benign;Likely benign                                        |
|                                                                       |
| 1772 1.5% other                                                       |
|                                                                       |
| 1188 1.0% Likely benign;Uncertain significance                        |
|                                                                       |
| 1077 0.9% Likely pathogenic;Pathogenic                                |
+-----------------------------------------------------------------------+
