Sun Grid Engine
===============

This section gives tips on how to configure a Sun Grid Engine (SGE) job
to request the right amount of resources to successfully execute one or
more BioR toolkit commands.

Enable SGE at Mayo
------------------

At Mayo, for example, you can log onto an RCF system, such as crick7, then run “mayobiotools” and choose “69. ogs”, then select the available option, choose “0” to save and exit, then log out and back in again. You should now be able to run SGE commands such as “qsub”.
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Multiple Cores
--------------

By default, an SGE job will run on a single core. It’s possible to run a
job on multiple cores is specified via the qsub command’s parallel
environment option “-pe”.

    -pe parallel_environment n[-[m]]|[-]m,...

To get a list of available parallel environments setup by your SGE
admin:

+------------------+
| > qconf -spl     |
|                  |
| fluent_pe        |
|                  |
| make             |
|                  |
| mpich2_141_hydra |
|                  |
| mpich2_mpd       |
|                  |
| namd2            |
|                  |
| openmpi          |
|                  |
| pvm              |
|                  |
| pvm-tight        |
|                  |
| threaded         |
+------------------+

Here is an example of requesting 4 cores for a job:

+-----------------------+
| > qsub -pe threaded 4 |
+-----------------------+

The following table gives recommend core values for toolkit commands.

+-----------------------+-----------------------+-----------------------+
| **Command**           | **Cores**             | **Notes**             |
+-----------------------+-----------------------+-----------------------+
| Arbitrary UNIX        | 0                     | examples: /bin/cat,   |
| commands              |                       | /bin/grep, /bin/cut   |
+-----------------------+-----------------------+-----------------------+
| bior_vcf_to_tjson     | 1                     |                       |
+-----------------------+-----------------------+-----------------------+
| bior_overlap          | 1                     |                       |
+-----------------------+-----------------------+-----------------------+
| bior_same_variant     | 1                     |                       |
+-----------------------+-----------------------+-----------------------+
| bior_lookup           | 1                     |                       |
+-----------------------+-----------------------+-----------------------+
| bior_drill            | 1                     |                       |
+-----------------------+-----------------------+-----------------------+
| bior_compress         | 1                     |                       |
+-----------------------+-----------------------+-----------------------+
| bior_vep              | 2                     | Warning: Variant      |
|                       |                       | Effect Predictor is   |
|                       |                       | implemented using     |
|                       |                       | PERL. The virtual     |
|                       |                       | memory for the PERL   |
|                       |                       | process grows         |
|                       |                       | linearly with more    |
|                       |                       | variants.             |
+-----------------------+-----------------------+-----------------------+
| bior_snpeff           | 2                     | SnpEff loads data     |
|                       |                       | into memory for       |
|                       |                       | performance           |
+-----------------------+-----------------------+-----------------------+
| bior_annotate         | 29                    | Annotate performs     |
|                       |                       | many commands in      |
|                       |                       | parallel              |
+-----------------------+-----------------------+-----------------------+
| bior_pretty_print     | 1                     |                       |
+-----------------------+-----------------------+-----------------------+

Virtual Memory
--------------

Virtual memory is specified via the qsub command’s resource request list
option “-l”.

    -l resource=value,...

NOTE: Resources specified with this option are **per-core**. If your job
uses 2 cores, you will need to divide the resource value by 2.

For virtual memory, the resource name to use is h_vmem. Here is an
example of requesting 10MB of virtual memory for a job running on 1
core:

+----------------------+
| > qsub -l h_vmem=10M |
+----------------------+

The following table gives recommend virtual memory values for toolkit
commands.

+-----------------------+-----------------------+-----------------------+
| **Command**           | **Virtual Memory**    | **Notes**             |
+-----------------------+-----------------------+-----------------------+
| Arbitrary UNIX        | 100M                  | examples: /bin/cat,   |
| commands              |                       | /bin/grep, /bin/cut   |
+-----------------------+-----------------------+-----------------------+
| bior_vcf_to_tjson     | 600M                  |                       |
+-----------------------+-----------------------+-----------------------+
| bior_overlap          | 600M                  |                       |
+-----------------------+-----------------------+-----------------------+
| bior_same_variant     | 600M                  |                       |
+-----------------------+-----------------------+-----------------------+
| bior_lookup           | 600M                  |                       |
+-----------------------+-----------------------+-----------------------+
| bior_drill            | 600M                  |                       |
+-----------------------+-----------------------+-----------------------+
| bior_compress         | 600M                  |                       |
+-----------------------+-----------------------+-----------------------+
| bior_vep              | 1200M\*               | Warning: Variant      |
|                       |                       | Effect Predictor is   |
|                       |                       | implemented using     |
|                       |                       | PERL. The virtual     |
|                       |                       | memory for the PERL   |
|                       |                       | process grows         |
|                       |                       | linearly with more    |
|                       |                       | variants.             |
+-----------------------+-----------------------+-----------------------+
| bior_snpeff           | 5100M                 | SnpEff loads data     |
|                       |                       | into memory for       |
|                       |                       | performance           |
+-----------------------+-----------------------+-----------------------+
| bior_annotate         | 24000M                |                       |
+-----------------------+-----------------------+-----------------------+
| bior_pretty_print     | 225M                  |                       |
+-----------------------+-----------------------+-----------------------+

Resources for a Toolkit Pipeline
--------------------------------

This section describes how to request the right resources for a
multi-command Toolkit pipeline.

Here is an example script that will be submitted to SGE:

+-----------------------------------------------------------------------+
| > cat example.sh                                                      |
|                                                                       |
| #!/bin/sh                                                             |
|                                                                       |
| # dbSNP 137 catalog                                                   |
|                                                                       |
| DBSNP_CATALOG=/path/to/catalogs/dbSNP/137/00-All_GRCh37.tsv.bgz       |
|                                                                       |
| # run toolkit pipeline to annotate my variants with dbSNP rsIDs       |
|                                                                       |
| cat data.vcf \| bior_vcf_to_tjson \| bior_same_variant -d             |
| $DBSNP_CATALOG \| bior_drill -p INFO.ID                               |
+-----------------------------------------------------------------------+

The number of cores needed to run this script’s processes in parallel
can be calculated by referencing the table in the Multiple Cores
section. The example script will require 3 cores to run optimally.

+-------------------+-----------+
| **Command**       | **Cores** |
+-------------------+-----------+
| example.sh        | 0         |
+-------------------+-----------+
| /bin/cat          | 0         |
+-------------------+-----------+
| bior_vcf_to_tjson | 1         |
+-------------------+-----------+
| bior_same_variant | 1         |
+-------------------+-----------+
| bior_drill        | 1         |
+-------------------+-----------+

The virtual memory needed to run this script can be calculated by
referencing the table in the Virtual Memory section. The example script
will require 2000M of virtual memory (100 + 100 + 600 + 600 + 600).

+-------------------+--------------------+
| **Command**       | **Virtual Memory** |
+-------------------+--------------------+
| example.sh        | 100M               |
+-------------------+--------------------+
| /bin/cat          | 100M               |
+-------------------+--------------------+
| bior_vcf_to_tjson | 600M               |
+-------------------+--------------------+
| bior_same_variant | 600M               |
+-------------------+--------------------+
| bior_drill        | 600M               |
+-------------------+--------------------+

The virtual memory setting h_vmem is specified on a **per-core** basis.
Since example.sh will be using 3 cores and 2000MB of virtual memory
total, h_vem is 2000/3 or roughly 670.

**Find an Open Queue**

+--------------+
| > qconf -sql |
+--------------+

Then choose a queue from the list to run your script under. For example,
we’ll assume there is a queue in the list called “MY_QUEUE” which we’ll
use in the final command.

Here is the final qsub command with the correct resource requirements:

+-----------------------------------------------------------------------+
| > qsub -m bae -M myemail@company.com -q 1-day -l h_vmem=5000M -pe     |
| threaded 3 -V -cwd example.sh                                         |
+-----------------------------------------------------------------------+

-cwd The -cwd param will specify output to go into the current directory
(execute job from current dir).

-wd You can specify a target directory for output using -wd <pathToDir>.

-V Using -V param will export ALL or your environment variables.

-M Send email

-m Notify me by mail (with -M flag) when certain conditions occur

Status of your Command
----------------------

Find the status of your command - here it is waiting in the queue, but
has not yet started processing:

+-----------------------------------------------------------------------+
| > qstat                                                               |
|                                                                       |
| job-ID prior name user state submit/start at queue slots ja-task-ID   |
|                                                                       |
| --------------------------------------------------------------------- |
| ----------------------------                                          |
|                                                                       |
| 119477 0.00000 example.sh m054457 qw 11/22/2013 09:28:11 3            |
+-----------------------------------------------------------------------+

After kicking off the process, it looks like:

+-----------------------------------------------------------------------+
| $ qstat                                                               |
|                                                                       |
| job-ID prior name user state submit/start at queue slots ja-task-ID   |
|                                                                       |
| --------------------------------------------------------------------- |
| ---------------------------                                           |
|                                                                       |
| 119477 0.51062 example.sh m054457 r 11/22/2013 10:47:47               |
| 1-day@dnode91.mayo.edu 3                                              |
+-----------------------------------------------------------------------+

**Get Command Results**

The grid will output several files that begin with the script name you
executed, and end with the queue jobId.

+------------------------------------------------------------------+
| > ls -la example.sh.\*                                           |
|                                                                  |
| -rw-r--r-- 1 m054457 biostat 0 Nov 22 09:38 example.sh.pe119477  |
|                                                                  |
| -rw-r--r-- 1 m054457 biostat 0 Nov 22 09:38 example.sh.pe119477  |
|                                                                  |
| -rw-r--r-- 1 m054457 biostat 0 Nov 22 09:38 example.sh.o119477   |
|                                                                  |
| -rw-r--r-- 1 m054457 biostat 283 Nov 22 09:38 example.sh.e119477 |
+------------------------------------------------------------------+

.. [1]
   `http://samtools.sourceforge.net/tabix.shtml <http://samtools.sourceforge.net/tabix.shtml>`__

   `http://bioinformatics.oxfordjournals.org/content/27/5/718.abstract <http://bioinformatics.oxfordjournals.org/content/27/5/718.abstract>`__
   HYPERLINK
   "http://bioinformatics.oxfordjournals.org/content/27/5/718.abstract"
