
bior_replace_lines
~~~~~~~~~~~~~~~~~~

Replaces lines in an input stream with other lines, usine one text file
as text to find, and another text file for text to replace those lines
with. This can be useful for changing VCF header lines on the fly to
adjust type or count before calling bior_vcf_to_tjson. For example, if
line 1 from find.txt is encountered in the input stream, it will be
replaced with line 1 from replacewith.txt.

+-----------------------------------------------------------------------+
| **# Start with the file we want to modify**                           |
|                                                                       |
| **# - Replace the double-quotes around Minor Allele Frequency with    |
| single quotes**                                                       |
|                                                                       |
| **# - Change the Type and Number on each ##INFO line**                |
|                                                                       |
| **# - Change the order of Type and Number around**                    |
|                                                                       |
| **# - Add a description for AC**                                      |
|                                                                       |
| **$ cat my.vcf**                                                      |
|                                                                       |
| ##INFO=<ID=MAF,Type=String,Number=.,Description="Description of       |
| "Minor Allele Frequency"">                                            |
|                                                                       |
| ##INFO=<ID=AC,Type=String,Number=.,Description="">                    |
|                                                                       |
| ##JUNK_HEADER1                                                        |
|                                                                       |
| ##JUNK_HEADER2                                                        |
|                                                                       |
| #CHROM POS ID REF ALT QUAL FILTER INFO                                |
|                                                                       |
| 1 100 rs1 A C . . .                                                   |
|                                                                       |
| **$ cat find.txt**                                                    |
|                                                                       |
| ##INFO=<ID=MAF,Type=String,Number=.,Description="Description of       |
| "Minor Allele Frequency"">                                            |
|                                                                       |
| ##INFO=<ID=AC,Type=String,Number=.,Description="">                    |
|                                                                       |
| **$ cat replaceWith.txt**                                             |
|                                                                       |
| ##INFO=<ID=MAF,Number=1,Type=Float,Description="Description of 'Minor |
| Allele Frequency'">                                                   |
|                                                                       |
| ##INFO=<ID=AC,Number=1,Type=Integer,Description="Allele Count">       |
|                                                                       |
| **# Now, let's do the replacement. Additionally, let's call "grep -v" |
| to remove the "##JUNK_HEADER" lines**                                 |
|                                                                       |
| $ cat my.vcf \| bior_replace_lines -f find.txt -t replaceWith.txt \|  |
| grep -v "##JUNK_HEADER"                                               |
|                                                                       |
| ##INFO=<ID=MAF,Number=1,Type=Float,Description="Description of 'Minor |
| Allele Frequency'">                                                   |
|                                                                       |
| ##INFO=<ID=AC,Number=1,Type=Integer,Description="Allele Count">       |
|                                                                       |
| #CHROM POS ID REF ALT QUAL FILTER INFO                                |
|                                                                       |
| 1 100 rs1 A C . . .                                                   |
+-----------------------------------------------------------------------+
