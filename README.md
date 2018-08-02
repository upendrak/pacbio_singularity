# Running Iso-Seq3 analysis on a Test data using Singularity container

*IsoSeq3* contains the newest tools to identify transcripts in PacBio single-molecule sequencing data. Starting in SMRT Link v6.0.0, those tools power the IsoSeq3 GUI-based analysis application. A composable workflow of existing tools and algorithms, combined with a new clustering technique, allows to process the ever-increasing yield of PacBio machines with similar performance to IsoSeq1 and IsoSeq2.

**Note:** This is an example of an end-to-end cmd-line-only workflow from this [tutorial](https://github.com/PacificBiosciences/IsoSeq3) to get from subreads to polished isoforms; timings are system dependent.

## Availability

The Iso-Seq3 can be run using Singualrity container hosted on Singularity hub

## Install Singularity 

There are many ways to [install Singularity](https://www.sylabs.io/guides/2.5.1/user-guide/quick_start.html#installation) but this quick start guide will only cover one.

```
git clone https://github.com/singularityware/singularity.git
cd singularity
./autogen.sh
./configure --prefix=/usr/local
make
sudo make install
cd ..
```
**Note:** Singularity must be installed as root to function properly.

After installing Singularity make sure to run the --help option gives an overview of Singularity options and subcommands

```
$ singularity --help

USAGE: singularity [global options...] <command> [command options...] ...
```
  
## Download the test data

```
For each cell, the <movie>.subreads.bam and <movie>.subreads.bam.pbi are needed for processing.

$ mkdir tutorial && cd tutorial
$ wget https://downloads.pacbcloud.com/public/dataset/RC0_1cell_2017/m54086_170204_081430.subreads.bam
$ wget https://downloads.pacbcloud.com/public/dataset/RC0_1cell_2017/m54086_170204_081430.subreads.bam.pbi
```

## Installation: Pull the Singularity container from Singularity hub

```
singularity pull --name isoseq.img shub://upendrak/pacbio_singularity
```

### Step 1: Consensus calling

```
$ singularity exec isoseq.img ccs --version
ccs 3.0.0 (commit a54f14a)

$ nohup singularity exec isoseq.img ccs m54086_170204_081430.subreads.bam m54086_170204_081430.ccs.bam --noPolish --minPasses 1 &

Note: This step takes a long time. On a 6 CPU VM, it took around 5 hrs to complete
``` 

### Step 2: Primer removal and demultiplexing

```
$ singularity exec isoseq.img lima --version
lima 1.7.0 (commit v1.7.0-2-g9479065)
```

```
$ cat primers.fasta

>primer_5p
AAGCAGTGGTATCAACGCAGAGTACATGGG
>primer_3p
GTACTCTGCGTTGATACCACTGCTT
```

```
$ singularity exec isoseq.img lima m54086_170204_081430.ccs.bam primers.fasta demux.bam --isoseq --no-pbi --dump-clips &
```

```
$ ls demux*
demux.json  demux.lima.counts  demux.lima.report  demux.lima.summary  demux.primer_5p--primer_3p.bam  demux.primer_5p--primer_3p.subreadset.xml
```

### Step 3: Clustering and transcript clean up

```
$ nohup singularity exec isoseq.img isoseq3 cluster demux.primer_5p--primer_3p.bam unpolished.bam --verbose &
```

```
$ ls unpolished*
unpolished.bam  unpolished.bam.pbi  unpolished.cluster  unpolished.fasta  unpolished.flnc.bam  unpolished.flnc.bam.pbi  unpolished.flnc.consensusreadset.xml  unpolished.transcriptset.xml
```

### Step 4: Polishing

```
$ nohup singularity exec isoseq.img isoseq3 polish unpolished.bam m54086_170204_081430.subreads.bam polished.bam --verbose
```

```
$ ls polished*
polished.bam  polished.bam.pbi  polished.hq.fasta.gz  polished.hq.fastq.gz  polished.lq.fasta.gz  polished.lq.fastq.gz  polished.transcriptset.xml
```


