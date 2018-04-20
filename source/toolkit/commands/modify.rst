
bior_modify_tjson
~~~~~~~~~~~~~~~~~

This command takes a config file that describes how fields within a JSON
column should be modified, then modifies each line streamed from STDIN

The config files consists of several columns that are required: KEY,
OPERATION, NEW_TYPE, NEW_COUNT; along with several additional columns
that are optional: DELIM, COMMENT, FIND, SET_TO (FIND and SET_TO can be
repeated as needed to make additional replacements. You will need to
specify as many pairs of these two headers as the row with the maximum
number of replacements requires. For example, if row 2 requires three
replacements, you will need three sets of FIND and SET_TO headers)

First, let's start with a simple example where we want to convert a
String to an Integer, while first replacing any values of "-inf" with
-99999.

+-----------------------------------------------------------------------+
| **# NOTE: The config file looks like this with tabs**                 |
|                                                                       |
| **$ cat ez.config**                                                   |
|                                                                       |
| #KEY OPERATION NEW_TYPE NEW_COUNT DELIM COMMENT FIND SET_TO           |
|                                                                       |
| Count MODIFY Integer 1 ; Str to int -inf -99999                       |
|                                                                       |
| **# Using the "column" cmd to line up columns makes it a little       |
| easier to read:**                                                     |
|                                                                       |
| **$ cat ez.config \| column -t -s $'\t'**                             |
|                                                                       |
| #KEY OPERATION NEW_TYPE NEW_COUNT DELIM COMMENT FIND SET_TO           |
|                                                                       |
| Count MODIFY Integer 1 ; Str to int -inf -99999                       |
|                                                                       |
| **$ cat ez.tsv**                                                      |
|                                                                       |
| {"Count":"-100"}                                                      |
|                                                                       |
| {"Count":"0"}                                                         |
|                                                                       |
| {"Count":"100"}                                                       |
|                                                                       |
| {"Count":"-inf"}                                                      |
|                                                                       |
| **# Note the conversion of "-inf" to -99999**                         |
|                                                                       |
| **$ cat ez.tsv \| bior_modify_tjson -f ez.config**                    |
|                                                                       |
| {"Count":-100}                                                        |
|                                                                       |
| {"Count":0}                                                           |
|                                                                       |
| {"Count":100}                                                         |
|                                                                       |
| {"Count":-99999}                                                      |
+-----------------------------------------------------------------------+

Now, let's take a more complicated example affecting multiple fields.
First, define a config file that will:

-  Convert a string field (MAF) to a float (replacing value "-inf" with
   "-999999", and value of "NA" with null)

-  Convert an integer field (isInDbSnp) to a boolean (where it contains
   only 0 or 1 values).

-  Add "_type" field which will always be set to value "variant"

-  Remove "X" field

-  Convert the \_altAlleles array to a string separated by pipes

-  Convert the ALT values (separated by commas) into a JSON array

-  AF - create an array from a single value, but first remove the "NA"
   and "-inf" values

-  INFO.GENE - modify a nested object

-  Replace a value within an array

+-----------------------------------------------------------------------+
| **# NOTE: The config file looks like this with tabs**                 |
|                                                                       |
| **$ cat my.config**                                                   |
|                                                                       |
| #KEY OPERATION NEW_TYPE NEW_COUNT DELIM COMMENT FIND SET_TO FIND      |
| SET_TO                                                                |
|                                                                       |
| MAF MODIFY Float 1 ; Str to float NA (NULL) -inf -99999               |
|                                                                       |
| isInDbSnp MODIFY Boolean 1 ; Int to bool (NULL) 0                     |
|                                                                       |
| \_type ADD String 1 . Add type of variant to each line (ANY) variant  |
|                                                                       |
| X REMOVE                                                              |
|                                                                       |
| \_altAlleles MODIFY STRING 1 \| JSON array to A|B v T C AAA           |
|                                                                       |
| ALT MODIFY STRING . , ,-sep str to array T C                          |
|                                                                       |
| AF MODIFY STRING . \| To array, rm NA NA (REMOVE)                     |
|                                                                       |
| INFO.GENE MODIFY STRING 1 \| Nested values BRCA1b BRCA1               |
|                                                                       |
| Arr MODIFY STRING . , Replace array val x X                           |
|                                                                       |
| **# But using the "column" cmd to line up columns makes it a little   |
| easier to read:**                                                     |
|                                                                       |
| **$ cat my.config \| column -t -s $'\t'**                             |
|                                                                       |
| #KEY OPERATION NEW_TYPE NEW_COUNT DELIM COMMENT FIND SET_TO FIND      |
| SET_TO                                                                |
|                                                                       |
| MAF MODIFY Float 1 ; Str to float NA (NULL) -inf -99999               |
|                                                                       |
| isInDbSnp MODIFY Boolean 1 ; Int to bool (NULL) 0                     |
|                                                                       |
| \_type ADD String 1 . Add type of variant to each line (ANY) variant  |
|                                                                       |
| X REMOVE                                                              |
|                                                                       |
| \_altAlleles MODIFY STRING 1 \| JSON array to A|B v T C AAA           |
|                                                                       |
| ALT MODIFY STRING . , ,-sep str to array T C                          |
|                                                                       |
| AF MODIFY STRING . \| To array, rm NA NA (REMOVE)                     |
|                                                                       |
| INFO.GENE MODIFY STRING 1 \| Nested values BRCA1b BRCA1               |
|                                                                       |
| Arr MODIFY STRING . , Replace array val x X                           |
|                                                                       |
| **$ cat my.tsv**                                                      |
|                                                                       |
| {"MAF":"0.32","isInDbSnp":0,"X":"abc","_altAlleles":["A","v","C","T", |
| "CC"],"ALT":"A,T,C","AF":"0.01|NA|0.12","INFO":{"GENE":"BRCA1b"},"Arr |
| ":["a","x"]}                                                          |
|                                                                       |
| {"MAF":"NA","isInDbSnp":null}                                         |
|                                                                       |
| {"MAF":"-inf","isInDbSnp":1}                                          |
|                                                                       |
| **# Notes:**                                                          |
|                                                                       |
| **# - MAF has been converted from a string to a float, with "NA"      |
| replaced with null, and "-inf" replaced with -99999**                 |
|                                                                       |
| **# - isInDbSnp has been converted from integer to boolean, with null |
| value replaced with false**                                           |
|                                                                       |
| **# - \_type has been inserted in each row with a value of            |
| "variant"**                                                           |
|                                                                       |
| **# - X key-value pair was removed entirely**                         |
|                                                                       |
| **# - \_altAlleles was converted from a JSON array to a               |
| pipe-separated string with replacements: "v" => "T", "C" => "AAA"**   |
|                                                                       |
| **# - ALT was converted from a string to a JSON array with            |
| replacement: "T" => "C"**                                             |
|                                                                       |
| **# - AF converted from pipe-delimited string to**                    |
|                                                                       |
| **$ cat my.tsv \| bior_modify_tjson -f my.config**                    |
|                                                                       |
| {"MAF":0.32,"isInDbSnp":false,"_altAlleles":"A|T|AAA|T|CC","ALT":["A" |
| ,"C","C"],"AF":["0.01","0.12"],"INFO":{"GENE":"BRCA1"},"Arr":["a","X" |
| ],"_type":"variant"}                                                  |
|                                                                       |
| {"MAF":null,"isInDbSnp":false,"_type":"variant"}                      |
|                                                                       |
| {"MAF":-99999,"isInDbSnp":true,"_type":"variant"}                     |
|                                                                       |
| **# If you want to change the name of a key, you can do that using    |
| the Linux "sed" command**                                             |
|                                                                       |
| **# Say we want to change "isInDbSnp" key to "isSNP"**                |
|                                                                       |
| **$ cat my.tsv \| bior_modify_tjson -f my.config \| sed               |
| 's/"isInDbSnp"/"isSNP"/g'**                                           |
|                                                                       |
| {"MAF":0.32,"isSNP":false,"_altAlleles":"A|T|AAA|T|CC","ALT":["A","C" |
| ,"C"],"AF":["0.01","0.12"],"INFO":{"GENE":"BRCA1"},"Arr":["a","X"],"_ |
| type":"variant"}                                                      |
|                                                                       |
| {"MAF":null,"isSNP":false,"_type":"variant"}                          |
|                                                                       |
| {"MAF":-99999,"isSNP":true,"_type":"variant"}                         |
+-----------------------------------------------------------------------+
