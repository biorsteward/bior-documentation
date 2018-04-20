Common Problems
===================

Handling VCF Files with VERY large headers
------------------------------------------

All BioR commands store the header in memory. This is done because
commands like bior_vcf_to_tjson use the header to understand the
structure of the data lines and parse the lines into JSON more
intelligently (e.g. identify numbers instead of strings, identify
arrays, ect.). In production, we have noticed that some headers are
extreamly large (multiple megabytes). When a user runs BioR, the header
is expanded into objects in memory for each BioR command. This can lead
to BioR slowing to a crawl when the ram on the machine is exceeded.
 Internally what happens is that the header is chopped off and stored in
memory, then each row streams through the system as an array of strings.
 The data rows are not that large, but the metadata in the header may
get copied many times in memory as transformations are done on the data.
 The best workaround for this problem is to use grep to cut off all
excess header lines (e.g. lines that are not descriptive) then push the
BioR output on to the file. Recombine the header if needed.

 e.g. 

zcat example.vcf.gz \| head -n 10000 \| grep -v "##" > mylongheader.vcf

zcat example.vcf.gz \| bior_vcf_to_tjson \| bior_mycommands >>
mylongheader.vcf

Large Memory Requirements
-------------------------

Sometimes users complain about large memory requrirements from BioR –
especially SNPEff. SNPEff, when run in production requires 4Gb of Ram.
BioR will align large insertions and deletions prior to sending them to
SNPEff using the same exact method used in SNPEff. When processing these
large variants, both BioR and SNPEff can crash. The current work-around
for dealing with large variants is to pre-screen them and filter them
out to another file prior to annotating with SNPEff. Hopefully the BioR
team will be able to collect better statistics and not align large
variants in the future.

BioR exits with some error I don’t understand
---------------------------------------------

Rerun the same exact command with logging enabled (-l) and submit both
the input file, and the results of the log to the BioR team. We will try
to help you ASAP.