# ChIP-seq and ATAC-seq processing

## Contents

1. Environment variables and `.bash_profile`
2. Running background jobs with `screen`
3. Sequence alignment with `seqalign`
4. Peak calling with `chipseqpeaks`

## 1. Environment variables and `.bash_profile`

Environment variables allow us to store information and recall it without
having to keep it memorized. For example, the variable `$HOME` stores the
location of your "home" directory:

```
echo $HOME
```

We can create or change environment variables using the `export` command. For
example:

```
export GREETING="hello world"
echo $GREETING
```
```
hello world
```

For the purpose of ChIP-seq and ATAC-seq analysis, there are a few environment
variables we will need to keep track of. For this pupose we will use the
`.bash_profile` found in our home directory. `.bash_profile` allows us to list
a series of `export` commands that will be run each time we log in,
setting up the environment variables automatically. We can view it by running:

```
cat ~/.bash_profile
```

For ChIP-seq and ATAC-seq analysis, the following variables should be in
`.bash_profile`:

```
# Set temporary directory
export TMPDIR=/nfs/lab/tmp

# pyhg19
export PYHG19_PATH=/nfs/lab/KG/ref/male.hg19.fa
export PYHG19_MASKED=/nfs/lab/KG/ref/male.masked.hg19.fa
export PYHG19_BOWTIE2_INDEX=/nfs/lab/hg19/male.hg19

```

## 2. Running background jobs with `screen`

Processing sequencing data can take some time (often a few hours), so we'll
need to be able to leave the computer and have it run while we're gone. To
achieve that we'll use a program called `screen`.

We can start a screen window simply by entering the `screen` command, but it's
best to also give the screen a name with the `-S` flag:

```
screen -S myscreen
```

To see a list of all screens currently running, use:

```
screen -ls
```

It will show something like this:

```
There are screens on:
        35367.myscreen  (Attached)
        6873.pts-49.gatsby      (Detached)
2 Sockets in /var/run/screen/S-aaylward.
```

We can see that we are now "attached" to the screen named "myscreen".
To detach from it, use:

```
screen -d
```

We can then use `screen -ls` again to check that we are detached from all
screens.

To re-attach, use:

```
screen -r myscreen
```

Next, we'll run a job that takes a minute to finish:

```
sleep 60
```

This is an instruction to "do nothing for 60 seconds". During those 60 seconds,
the terminal is busy, so we can't use `screen -d` to detach. Instead, use
`ctrl+a` to "activate" the letter keys, and then press `d` to detach. After the
60 seconds are up, we can re-attach and check that the job is finished.

When the screen window is no longer needed, we can terminate it using:

```
screen -X -S myscreen quit
```

Running `screen -ls` then shows the screen is no longer running.

> Note, you can also terminate a screen when it is attached by entering `exit`
> or by using `ctrl+a` then `k` (for "kill").

## 3. Sequence alignment with `seqalign`

The first step of processing sequencing data is to align them to a reference
genome. We will use `seqalign` to accomplish this. First, make sure `seqalign`
is installed and up to date:

```
pip install --upgrade seqalign
```

Once it is installed, we can learn a bit about it by running:

```
seqalign
```
```
There are two commands for aligning sequencing data: `seqalign-chip-seq`
and `seqalign-atac-seq`. See their respective documentation for more
information by running:

seqalign-chip-seq --help
seqalign-atac-seq --help
```

There are several optional arguments for these commands, but typically (for
human data) they will not be needed. As an example, we can run:

```
cd /nfs/lab/ChIPseq/example/
seqalign-chip-seq reads/HepG2_FOXA2_1_S1_L004_R1_001.fastq.gz bam/HepG2_FOXA2_1
```

The first argument is a FASTQ file from a ChIP-seq experiment, the second is
a prefix for output files.

The input/control data can be aligned in the same way:

```
seqalign-chip-seq reads/HepG2_input_S12_L004_R1_001.fastq.gz bam/HepG2_input
```

We may want to use optional arguments occasionally. For example, by default
reads are aligned to a human reference genome, so to align mouse data, we will
have to specify a mouse genome using the `--reference` option.

```
seqalign-chip-seq --reference /nfs/lab/mm10/GRCm38.mm10.fa reads/MIN6_FOXA2_1_S1_L008_R1_001.fastq.gz bam/MIN6_FOXA2_1
```

Similarly, we can perform an atac-seq alignment by running:

```
cd /nfs/lab/ATACseq/example/
seqalign-atac-seq reads/SAMN10079665_untreated_R1.fastq.gz reads/SAMN10079665_untreated_R2.fastq.gz bam/SAMN10079665_untreated
```

In this case, the first two arguments are the two files representing the
paired-end read data. The last argument is again a prefix for the output files.

Sometimes ChIP-seq data may also be paired end. In those cases the command for ChIP-seq alignment is similar to ATAC-seq alignment:

```
cd /nfs/lab/ChIPseq/example
seqalign-chip-seq reads/CREB1.ENCFF199RYE.R1.fastq.gz reads/CREB1.ENCFF149OFF.R2.fastq.gz bam/CREB1
```

## 4. Peak calling with `chipseqpeaks`

Both ChIP-seq and ATAC-seq peaks are called with `chipseqpeaks`, since they
use the same algorithm. First, make sure it is installed and up to date:

```
pip install --upgrade chipseqpeaks
```

To call peaks on ChIP-seq data, we'll nead to specify both the treatment
and input/control alignment data, like this:

```
cd /nfs/lab/ChIPseq/example/
chipseqpeaks-call --output-dir peaks/ --control bam/HepG2_input.sort.filt.rmdup.bam  bam/HepG2_FOXA2_1.sort.filt.rmdup.bam
```

On ATAC-seq data, we just need to give the `--atac-seq` option and specify
the alignment data.

```
cd /nfs/lab/ATACseq/example/
chipseqpeaks-call --atac-seq --output-dir peaks/ bam/SAMN10079665_untreated.sort.filt.rmdup.bam
```
