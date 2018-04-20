==================
Installation Guide
==================

Installing on a Stand-Alone Server
----------------------------------

Also: Workstation (or on a server on the cloud)

This includes Java JDK 1.7, Tabix and BGZIP. Most BioR functions will
still work if you don’t install SNPEFF/VEP and all of their
dependencies, but bior_annotate will have limited functionality and
bior_vep and bior_snpeff will not work. The environment variable,
$BIOR_LITE_HOME represents the location where BioR is installed.

Installing the BioR software
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

System Requirements
^^^^^^^^^^^^^^^^^^^

Linux/Unix Distribution: preferably Ubuntu 12.04.
Memory: 4gb (8gb prefered)
Architecture: 64 bit system

Disk Space: (Toolkit requires 1GB, catalogs can very, please see
the BioR download website for more details)

Prerequisites:
^^^^^^^^^^^^^^

BioR is written in Java, so in principle it will work on any machine,
but it depends on some command line tools (e.g. SNPEFF, VEP) that are
not so friendly. The development team has BioR working on both Macintosh
and Linux. To install, first make sure first that Java 1.6+ is installed
and on your path (Java 1.7 is preferred). Then download the BioR
executable and place it in your path.

Java Installation:
''''''''''''''''''

We prefer to use the official version of Java directly from Oracle
(Sun). Use uname -a on your Linux machine to determine the version of
Java that you should be running on your server. Sign up for an Oracle
account (if needed) and download the correct version. Copy it to your
server (i.e. ~ - your home directory for this guide). You can run java
-version to see if it is already installed on your server to skip this
step.

**Download Link:**

`**http://www.oracle.com/technetwork/java/javase/downloads/index.html** <http://www.oracle.com/technetwork/java/javase/downloads/index.html>`__

**Installation Instructions:**

Check the version of your Linux OS by typing ‘uname -a’ at the command
line. if it says x86_64, you have a 64bit system (most systems). Based
on the version of your OS, download Java directly from Oracle. We
recommend JDK 1.7. (I usually download it to my home computer and then
scp it up to the server using winscp or scp from the command prompt on a
mac. This will result in a file in your home directory called
jdk-7u45-linux-i586.tar.gz. Unzip Java and set your JAVA_HOME.

Once you have java downloaded to your PC/Mac, you will need to copy it
up to your server. If you have a mac, you can use scp from the command
line, on Windows, you can use WinSCP or similar program. For example to
upload from my Mac I use the following scp command (after downloading
from Oracle):

$ scp -i biorloginkey.pem ~/Downloads/jdk-7u45-linux-x64.tar.gz
ubuntu@10.148.2.10:~/jdk-7u45-linux-x64.tar.gz

Yours will differ because your cloud server will be on a different IP
and you will have a different credential file (the .pem).

This will place a copy of Java in my home directory (~ = /home/ubuntu)
Extract it.

ubuntu@biorinstall2:~$ pwd

/home/ubuntu

ubuntu@biorinstall2:~$ tar -zxvf jdk-7u45-linux-x64.tar.gz

Then you will need to put it in your path:

ubuntu@biorinstall2:~$ ls

bior_2.2.0 bior_2.2.0.tar.gz jdk1.7.0_45 jdk-7u45-linux-x64.tar.gz

ubuntu@biorinstall2:~$ JAVA_HOME=/home/ubuntu/jdk1.7.0_45

ubuntu@biorinstall2:~$ PATH=$PATH:$HOME/bin:$JAVA_HOME/bin

ubuntu@biorinstall2:~$ export $JAVA_HOME

-bash: export: \`/home/ubuntu/jdk1.7.0_45': not a valid identifier

ubuntu@biorinstall2:~$ export JAVA_HOME

ubuntu@biorinstall2:~$ export PATH

ubuntu@biorinstall2:~$ javac

(usage information should be here, if not it is not correctly installed)

I put these commands in ~/.bash_profile so that they are in place next
time I log in.

Tabix / BGZIP:
''''''''''''''

**Install:**

Make sure you have a version of bgzip2 installed:

$ sudo apt-get install bzip2

You may also be able to install tabix in the same way:

$ sudo apt-get install tabix

**Tabix Manual install:**

If the apt-get command fails, you can download and install it yourself:

Download Link:
`http://sourceforge.net/projects/samtools/files/tabix/ <http://sourceforge.net/projects/samtools/files/tabix/>`__

In this case, we chose the 0.2.6 version.

Download from the link above and unzip:

$ bunzip2 tabix-0.2.6.tar.bz2

$ tar -xvf tabix-0.2.6.tar

$ cd tabix-0.2.6/

Compile Tabix:

$ make

Add Tabix and BGZIP to your path:

$ PATH=~/tabix-0.2.6:$PATH

$ which tabix

/home/dquest/tabix-0.2.6/tabix

You may want this command also to be in your .bashrc file so that tabix,
bgzip work properly.

SNPEFF+VEP:
'''''''''''

SNPEFF+VEP have complex setups, please see below in the user guide on
how to set them up and get them working with the toolkit. Note, you will
not need these tools for most of the BioR commands or the quickstart.

BioR Toolkit Installation:
^^^^^^^^^^^^^^^^^^^^^^^^^^

**Download Link:**

You can download BIOR and Catalog datasources from
`http://bioinformaticstools.mayo.edu/research/bior/ <http://bioinformaticstools.mayo.edu/research/bior/>`__.

**Installation Instructions:**

1) Download the toolkit: (e.g.)

$ wget
`https://s3-us-west-2.amazonaws.com/mayo-bic-tools/bior/bior_2.2.0.tar.gz <https://s3-us-west-2.amazonaws.com/mayo-bic-tools/bior/bior_2.1.1.tar.gz>`__

2) Unzip the bior_version zip file you downloaded. (unzip
bior_version.zip -d target directory) e.g.:

$ tar -xzvf bior_2.2.0.tar.gz

3) Make sure all your files in bior_pipeline project are executable:

$ cd bior_2.2.0/

$ chmod -R +x bin/

4) Now you need to setup the environment variables and add to the path.

$ export BIOR_LITE_HOME=YOUR BIOR\_ FOLDER

$ export PATH=$BIOR_LITE_HOME/bin:$PATH

There is a quick script that comes with BioR that can help with the
setup: setupEnv.sh. Just source the file:

$ source setupEnv.sh

You will need to setup your paths each time you login so it might make
sense to add this command into your .bash_profile/.bashrc.

5) Now try bior\_ and press tab key twice on terminal. Now you should
see all bior commands displayed. If they are not being displayed, look
inside the setupEnv.sh and change the paths so that they work with your
envorinment. (BioR is using BASH). Change it as needed or ask a system
administrator for help. Make sure you can type bior_(tab tab) and get
all of the commands back before moving on to the next step.

6) Verify that it is installed correctly by typing bior_pretty_print -h.
You should see a help message, if you see an error like “java: not
found” then you need to install java correctly.

Now you have successfully installed the toolkit. The quickstart guide is
a good place to go to check if your toolkit is functioning properly and
to run some biologically motivated examples contributed from our
bioinformaticians (they even use versions of these in production!).

One of the hardest to set up commands is bior_annotate. Bior_annotate is
a kitchen sink command and requires the catalogs, command line tools and
all dependencies be installed properly on your system. The next three
sections will go over how to install all of the catalogs it needs, and
how to install SNPEFF and VEP. For now, it is important to highlight the
bior.properties file in $BIOR_LITE_HOME/conf. For example:

$ pwd

/home/ubuntu/bior_2.2.0/conf

ubuntu@biorinstall2:~/bior_2.2.0/conf$ ls

allCatalogs.columns.properties allCatalogs.columns.tsv bior.properties
cli log4j.properties tools

Edit the configuration file so that all tools, catalogs, and paths are
consistent with your install locations (see the next section).

Installing the Biological Repository Catalogs
---------------------------------------------

Catalogs can be found at $BIOR_CATALOG ($bior in this documentation) If
you are doing a stand alone server, download the catalog flat files and
place them locally on your server in a similar directory structure. BioR
Tools does not make any assumptions about the location of catalogs
relative to each other, but it does assume that tabix indexes are in the
same directory as the compressed catalog and that ID indices are in a
folder called index in the same directory as the catalog. However,
bior_annotate does have a configuration file that will make that command
not work if you don’t change the configuration file or place the data
repositories in the same location as we do at Mayo (or provide a
symbolic link). We put the data here:
$BIOR_CATALOG=/data5/bsi/catalogs/bior/v1. More details for installing
the catalog structure properly on a stand alone server can be found in
the next section “Installing on a Stand-Alone Server or Workstation”
(next).

1) use wget to get all of the BioR catalogs and place them in /data.
There are two scripts: $BIOR_LITE_HOME/scripts/downloadFullCatalogs.sh

and

$BIOR_LITE_HOME/scripts/downloadSmallCatalogs.sh

that can be used to easily download the production catalogs and example
catalogs respectively. For example do this to download the full
catalogs:

$ cd $BIOR_LITE_HOME

$ cd scripts

$ ./downloadFullCatalogs.sh /data/

This will download and extract the downloaded catalogs into the /data
directory.

3) Now, for bior_annotate to work, you will need to set the properties
(for all of the rest of BioR, you are good to go).

You will find a file named *bior.properties* under the folder conf in
your bior_version directory (See the section above on installing the
toolkit). This is the file where you need to set the tools path and home
path of catalogs directory. Tool commands like bior_vep and bior_snpeff
and as well as bior_annotate make use of this properties file. My file
is here:

$ pwd

/home/ubuntu/bior_2.2.0/conf

$ ls

allCatalogs.columns.properties allCatalogs.columns.tsv bior.properties
cli log4j.properties tools

The rest of the setup guide will assume that your bior.properties file
is set up as follows (note fileBase = /data/ and it MUST end in a
slash), the paths you may need to change are in bold:

$ cat bior.properties

###================================================================================================

### NOTE: These keys are defined in BiorProperties.java and must match
the keys in the enum there

###================================================================================================

### Base directory for Reference Assembly catalogs that contain the
reference base pairs for various builds

refAssemblyBaseDir=/data/ref_assembly

###SNPEFF ============================================

**SnpEffJar=/data/snpeff_2_0_5d/snpEff.jar**

**SnpEffConfig=/data/snpeff_2_0_5d/snpEff.config**

**SnpEffCmd=java -Xmx4g -jar $SnpEffJar eff -c $SnpEffConfig -v -o vcf
-noLog -noStats GRCh37.64**

###VEP ===============================================

**BiorVepPerl=/usr/bin/perl**

**BiorVep=/data/variant_effect_predictor/variant_effect_predictor.pl**

**BiorVepCache=/data/variant_effect_predictor/cache/**

###BIOR_TREAT ========================================

### AnnotateMaxLinesInFlight:

### NOTE: Min = 2. Default = 10. Max = 50 (could do more, but not
recommended)

### WARNING: Do not increase it to much more than 50 or you may
encounter a hang state, especially with a high number of fanouts, as the
process buffers will overflow!

AnnotateMaxLinesInFlight=10

**### NOTE: make sure the “fileBase” path ends with a slash!**

**fileBase=/data/**

genesFile=NCBIGene/GRCh37_p10/genes.tsv.bgz

bgiFile=BGI/hg19/LuCAMP_200exomeFinal.maf_GRCh37.tsv.bgz

espFile=ESP/build37/ESP6500SI_GRCh37.tsv.bgz

hapMapFile=hapmap/2010-08_phaseII+III/allele_freqs_GRCh37.tsv.bgz

dbsnpFile=dbSNP/137/00-All_GRCh37.tsv.bgz

dbsnpClinvarFile=dbSNP/137/clinvar_20130226_GRCh37.tsv.bgz

cosmicFile=cosmic/v63/CosmicCompleteExport_GRCh37.tsv.bgz

kGenomeFile=1000_genomes/20110521/ALL.wgs.phase1_release_v3.20101123.snps_indels_sv.sites_GRCh37.tsv.bgz

blacklistedFile=ucsc/hg19/wgEncodeDacMapabilityConsensusExcludable_GRCh37.tsv.bgz

repeatFile=ucsc/hg19/rmsk_GRCh37.tsv.bgz

regulationFile=ucsc/hg19/oreganno_GRCh37.tsv.bgz

uniqueFile=ucsc/hg19/wgEncodeDukeMapabilityRegionsExcludable_GRCh37.tsv.bgz

tssFile=ucsc/hg19/switchDbTss_GRCh37.tsv.bgz

tfbsFile=ucsc/hg19/tfbsConsSites_GRCh37.tsv.bgz

enhancerFile=ucsc/hg19/vistaEnhancers_GRCh37.tsv.bgz

###
conservationFile=ucsc/hg19/phastConsElements46wayPrimates_GRCh37.tsv.bgz

conservationFile=ucsc/hg19/phastConsElements46way_GRCh37.tsv.bgz

hgncFile=hgnc/2012_08_12/hgnc_GRCh37.tsv.bgz

hgncIndexFile=hgnc/2012_08_12/index/hgnc_GRCh37.Entrez_Gene_ID.idx.h2.db

hgncEnsemblGeneIndexFile=hgnc/2012_08_12/index/hgnc_GRCh37.Ensembl_Gene_ID.idx.h2.db

omimFile=omim/2013_02_27/genemap_GRCh37.tsv.bgz

omimIndexFile=omim/2013_02_27/index/genemap_GRCh37.MIM_Number.idx.h2.db

mirBaseFile=mirbase/release19/hsa_GRCh37.p5.tsv.bgz

### List of catalogs and their high-level properties.

### This is built by running this command that crawls the catalog
directories and assembles info about them

### \_bior_catalog_list <CATALOGS_PARENT_DIR>

catalogListFile=/data/CATALOG_LIST.txt

### Human reference files for bior_verify

### GRCh37

humanRefSeqGrch37File=/data/ref_assembly/ucsc/hg19/20161109/genome_GRCh37.100kBPsPerLine.tsv.bgz

humanRefChrSizesGrch37File=/data/ucsc_genome/human/hg19/downloaded/2015_07_07/hg19.chrom.sizes

humanRefChrOrderGrch37File=/data/ucsc_genome/human/hg19/processed/2015_07_07/chroms/chrs_in_order

### GRCh38

humanRefSeqGrch38File=/data/ref_assembly/ucsc/hg38/20161109/genome_GRCh38.100kBPsPerLine.tsv.bgz

humanRefChrSizesGrch38File=/data/ucsc_genome/human/hg38/downloaded/2015_07_22/hg38.chrom.sizes

humanRefChrOrderGrch38File=/data/ucsc_genome/human/hg38/processed/2015_07_22/chroms/chrs_in_order

Now in the file you need to set fileBase=”catalogs directory” value to
your catalogs directory.

***If you chose to download the chr17-only catalogs, make sure to update
the paths in the bior.properties file as appropriate.***

*Some users notice a problem with Sage (an error in the bior.log file
when running with the --log flag) not being able to make a connection.
If you get this error, make sure the SAGE_ENVIRONMENT is set as follows:
SAGE_ENVIRONMENT=null, (or =prod)*

*in the Global.properties file (found in:
/home/ubuntu/bior_2.2.0/conf/cli on the example system, or
$BIOR_LITE_HOME/conf/cli)*

Example : fileBase=/data/

Next step is tools installation.

**Dependant Tools Installation and Setup**

We have integrated two tools SNPEff and Variant Effect Predictor (VEP)
into our toolkit.

**Variant Effect Predictor (VEP):**

The Version of VEP we support is 2.7.

`http://useast.ensembl.org/info/docs/tools/vep/script/vep_download.html#versions <http://useast.ensembl.org/info/docs/tools/vep/script/vep_download.html#versions>`__

***NOTE: There is a breaking change in the latest VEP tools that
replaces the --hgnc flag with --symbol. Make sure to use version 2.7 to
avoid this!***

You can follow the installation instructions in the above page. Here is
how we install it step by step:

1) Download VEP from their website:

$ cd /data

$ curl -o variant_effect_predictor.v69.tar.gz
“http://cvs.sanger.ac.uk/cgi-bin/viewvc.cgi/ensembl-tools/scripts/variant_effect_predictor.tar.gz?view=tar&root=ensembl&pathrev=branch-ensembl-69”

*NOTE: The quotes in the command above can cause problems when
copy-and-pasting from this document into your terminal windows. If that
is the case, try deleting the quotes at the command line and adding them
back in by typing them manually.*

2) Extract

$ tar -xzvf variant_effect_predictor.tar.gz

3) Make sure you have Perl version 14 on your computer:

$ perl -v

4) If the perl is the correct version, and you have admin rights, you
can now install the Perl libraries needed by VEP:

$ sudo apt-get install libwww-perl

$ sudo perl -MCPAN -e'force install "LWP::Simple"'

$ sudo apt-get install libdbi-perl

$ sudo apt-get install libdbd-mysql-perl

You can also download the Perl libraries separately and point your
PERL5LIB environment variable to them. For more information, see:

`http://linuxgazette.net/139/okopnik.html <http://linuxgazette.net/139/okopnik.html>`__

5) Then run the VEP installer:

    cd variant_effect_predictor

    perl INSTALL.pl [options]

6) Make sure to download the needed cache files or correct the install
(in my case the download script failed)

It may ask you to install cache files into your ~/.vep directory. If you
choose to do this, and it succeeds, make sure to update the
bior.properties file outlined in a previous step to point to this
directory. Choose the two options for homo_sapiens by specifying them
both, separated by a space:

25 : homo_sapiens_refseq_vep_73.tar.gz

26 : homo_sapiens_vep_73.tar.gz

You may follow instructions at
http://www.ensembl.org/info/docs/api/api_installation.html which
provides alternate options for the API installation. Example download of
the cache files.

$ nohup wget
ftp://ftp.ensembl.org/pub/release-69/variation/VEP/homo_sapiens_vep_69.tar.gz
&

$ nohup wget
ftp://ftp.ensembl.org/pub/release-69/variation/VEP/homo_sapiens_vep_69_sift_polyphen.tar.gz
&

7) unzip the alignment files to a directory called “cache” inside the
same directory as VEP.

cd /data/variant_effect_predictor/

mkdir cache

cd cache

tar -zxvf ../homo_sapiens_vep_69_sift_polyphen.tar.gz

tar -zxvf ../homo_sapiens_vep_69.tar.gz

8) Change the bior.properties file to point at the version of vep that
you just installed.

###VEP ===============================================

BiorVepPerl=/usr/bin/perl

BiorVep=/data/variant_effect_predictor/variant_effect_predictor.pl

BiorVepCache=/data/variant_effect_predictor/cache/

9) Test that VEP works stand-alone on a VCF file to ensure it is
installed correctly. Here is an example command (note there is an
example.vcf is in $BIOR_LITE_HOME/examples/quickstart2/example.vcf but
any properly formatted vcf will work):

perl /data/variant_effect_predictor/variant_effect_predictor.pl -i
example.vcf -o STDOUT -dir /data/variant_effect_predictor/cache/ -vcf
-polyphen b -sift b --offline

***NOTE: If you downloaded the small catalogs (for chr17 only), you may
want to use the $BIOR_LITE_HOME/examples/quickstart2/example_chr17.vcf
file for all subsequent examples (instead of example.vcf), as this
provides variants within chromosome 17, specifically within the BRCA1
gene range.***

10) Test that VEP works inside the BioR wrapper:

cat example.vcf \| bior_vep > vepAnnotated.tjson

*NOTE: If VEP fails here, try it again with the --log flag, then cat the
bior.log file. If it shows an error similar to “Unable to locate file on
classpath: /coreutils-8.21/linux-i686/stdbuf”, then you are most likely
trying to run it on a 32-bit OS. This is currently not supported. Please
try on a 64-bit OS for now.*

**SNPEff:**

Currently we support SNPEff verison 2.0.5d.This was recommended by GATK
for worst pick logic.

Installation files and instructions can be found at

`http://snpeff.sourceforge.net/download.html <http://snpeff.sourceforge.net/download.html>`__\ If
you using linux or Mac you can just use wget command to download the
files in steps 2 and 3. After doing so, make sure to change SNPEFF
config file snpEff.config (within the newly downloaded snpeff directory)
to include the path to the database you downloaded.

1) Make sure you have unzip installed so you can extract the zip file
(on ubuntu linux, use apt-get. Use yum or whatever package manager to
install unzip on other boxes):

$ sudo apt-get install unzip

2) Extract SNPEFF

$ cd /data

$ wget
`http://sourceforge.net/projects/snpeff/files/snpEff_v2_0_5d_core.zip <http://sourceforge.net/projects/snpeff/files/snpEff_v2_0_5d_core.zip>`__

$ unzip snpEff_v2_0_5d_core.zip

3) By default, SNPEFF expects that you place the data in a directory
called ‘data’ residing in the same directory as snpEff.jar
(SNPEFF_HOME). Extract the data for SNPEFF to SNPEFF_HOME/data.

$ cd snpEff_2_0_5d

$ wget
`http://sourceforge.net/projects/snpeff/files/databases/v2_0_5/snpEff_v2_0_5_GRCh37.64.zip <http://sourceforge.net/projects/snpeff/files/databases/v2_0_5/snpEff_v2_0_5_GRCh37.64.zip>`__

$ unzip snpEff_v2_0_5_GRCh37.64.zip

###(This will create the /data/snpEff_2_0_5d/data/ directory)

4) After you have installed SNPEff, set the paths in
bior_pipeline/conf/bior.properties

Example:

###SNPEFF ============================================

**SnpEffJar=/data/snpEff_2_0_5d/snpEff.jar**

**SnpEffConfig=/data/snpEff_2_0_5d/snpEff.config**

SnpEffCmd=java -Xmx4g -jar $SnpEffJar eff -c $SnpEffConfig -v -o vcf
-noLog -noStats GRCh37.64

5) check that SNPEFF works as a stand alone tool. This will take at
least a minute to load the database before it starts processing vcf
lines (note there is an example.vcf is in
$BIOR_LITE_HOME/examples/quickstart2/example.vcf but any properly
formatted vcf will work).

cat example.vcf \| java -Xmx4g -jar /data/snpEff_2_0_5d/snpEff.jar eff
-c /data/snpEff_2_0_5d/snpEff.config -v -o vcf -noLog -noStats GRCh37.64

*NOTE: If you are running on a system with less than 4GB of memory, this
will throw an exception. Run again with the --log flag, then check the
bior.log file for an error similar to “AbnormalExitException”. Please
change the -Xmx4g option to a smaller value to fit your hardware
limitations (such as -Xmx2g). Likewise, if you have more memory than
that, or get an exception because SnpEff runs out of memory, you can up
the memory as well.* *For the bior_snpeff command, this can be set in
the bior.properties file as part of the SnpEffCmd property.*

6) Run the BioR wrapper for SNPEFF:

cat example.vcf \| bior_snpeff > annotated.tjson

NOTE: As before, the memory allocated to SnpEff might be a problem. For
the bior_snpeff command, you can specify the max heap size by altering
this field in the bior.properties file:

SnpEffCmd=java **-Xmx4g** -jar $SnpEffJar eff -c $SnpEffConfig -v -o vcf
-noLog -noStats GRCh37.64

Instructions for Mayo Users.
----------------------------

There is no need to install anything, just use the mayobiotools command
on the RCF. Please read the drupal documentation here:
`http://bsiweb.mayo.edu/bior-20-installing-command-line-client <http://bsiweb.mayo.edu/bior-20-installing-command-line-client>`__

Installing BioR Tools from Source
---------------------------------

Source installation requires that you have both Java 1.7 and Maven
installed and on your path. It also requires that you have access to the
Mayo NEXUS servers or you place several libraries in your ~/.m2
directory.

If you have troubles installing BioR or compiling it, please contact the
BioR Team (dlrstitbiorall@mayo.edu) so we can update the documentation
and make the process easier.

Java Heap Size
--------------

On some machines, the default JVM size is 2GB. This is very large for
BioR. By default the BioR toolkit is capped at 128M. To change this
setting, change the Maven bior_pipeline/pom.xml (e.g.
<jvmOpts>-Xmx128m</jvmOpts>).
