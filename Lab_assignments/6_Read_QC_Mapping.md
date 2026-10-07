
## Read mapping

We are continuing our build up to exploring genomic data via bioinformatics. You've learned the basics:  programming, interacting with Linux environments, and job management and submission via `SLURM`. You've also learned how to programmatically interact with NCBI and download data, as well as how to assemble and annotate genomes. Today you will learn how to  map whole genome resequence data to a genome assembly. The ideas are comparable to other data (Sanger sequences, primers, RNAseq), but those have notable differences.

As we've discussed in class, we want to inspect and prepare our data prior to working with our data. We also want to make sure we do some initial trimming of our data to get rid of adaptor sequences, low quality bases, etc before we actually align our data to a genome assembly. 

As always, we will be writing a script today, so make sure that you write a single, functional script from this tutorial that you run on the compute node :frog:. I will also be asking what your opinion of the data QC in the assignment. Finally, I am giving you software packages to run, but unlike previous labs I am not giving you the exact commands and instead I am expecting you to read the documentation more thoroughly before using them.
### Data QC

The first steps are data QC. The very first of which is verifying the "md5 hash", which is basically a unique key to the data to make sure we have the files properly downloaded. Some software will automatically check this. When you download data from a core facility or from a collaborator you will want to make sure you have the correct, uncorrupted/truncated files. Often, this is done using `md5` values. md5 hash values are generated prior to uploading/transferring, and then tested after download. You can use `md5sum` to generate and/or check md5 values for files.

I have placed fastq files `/project/stuckert/bioinformatics/mapping`to check. The file of md5 values is `/project/stuckert/bioinformatics/mapping/md5values.txt. Use the help from the md5sum command to figure out how to use a file of md5 values to check.

The next thing you will want to do is explore the read quality with `fastp`. Download precompiled binaries:

```
wget https://github.com/s-andrews/FastQC/releases/download/v0.13.0/fastqc_v0.13.0.zip
```

Make sure `java` is in your path, and you can call `fastqc` on your data files. You may need to add this via the `module add` command on the compute node. Run `fastqc` on all the data files in `/project/stuckert/bioinformatics/mapping`.

To show how we are building on our work throughout the semester, below is a script I used to download all the data. You can see elements from nearly every lab to date are in this.

```
#!/bin/bash
#SBATCH --cpus-per-task=1
#SBATCH --mem=20Gb
#SBATCH -t 1-0:00:00 

# download *D. sech* metadata
esearch -db sra -query '"Drosophila sechellia"[Organism] AND "strategy wgs"[Properties] AND "platform illumina"[Properties]' \
  | efetch -format xml \
  | xtract -pattern EXPERIMENT_PACKAGE \
    -element RUN@accession SAMPLE@alias INSTRUMENT_MODEL Center_name RUN@published \
    -block SAMPLE_ATTRIBUTE -if TAG -equals "geo_loc_name" -element VALUE > D.sech.samples.txt

# filter out just populations I want:
grep -E 'Denis|Anro|PNF' D.sech.samples.txt | grep "HiSeq 2500" > D.sech.metadata.txt

# extract just the accession numbers to download
cut -f1 D.sech.metadata.txt > D.sech.accessions.txt

# load sratoolkit
ml sra

# download all samples
while read line
do
prefetch $line
fasterq-dump --split-files $line
done < D.sech.accessions.txt

```


## Trimming/filtering

There are many pieces of software designed to trim data. Commonly seen ones in the genomics literature are TrimGalore, Trimmomatic, Cutadapt, Fastp, HTSstream. Today we will be using `fastp` for our trimming. As we discussed in class, you want to make sure your reads are cleaned up by removing adaptors, trimming poor quality bases (typically via the ends), removing optical and PCR duplicates (because they are artifacts, not biologically relevant), and removing reads that are very bad. 

Fastp is a simple command, but by default it does a lot under the hood. You should look at the help output. Make sure that you use the paired end option(s) AND write new output files (remember DO NOT overwrite your raw data; in fact make it READ ONLY). Fastp has a list of common adaptors internally, which will work for most Illumina reads, but you want to know your data sources prior to running.

To download `fastp`:

```
wget http://opengene.org/fastp/fastp
chmod a+x ./fastp
```

## Alignment

After trimming our raw reads we will align them with bwa-mem2. 

```
curl -L https://github.com/bwa-mem2/bwa-mem2/releases/download/v2.2.1/bwa-mem2-2.2.1_x64-linux.tar.bz2 \
  | tar jxf -

```

Before we align reads, we actually first need to index the genome. But, before we index the genome we actually need the genome. In previous labs, we assembled a genome for *Drosophila sechellia*, but today we will be working with the polished assembly on NCBI. You can download this via the NCBI `datasets` software, and I have a version of this for you (in `/project/stuckert/bioinformatics/software`). I want you to download only the genomic fasta, so I have specified 

```
datasets download genome accession "GCA_004382195.2" --include genome
```

Once you have unzipped your genome, you can index it in preparation for alignment:

```
bwa-mem2 index [GENOME]
```

Now that you have indexed your genome, you can align your samples to it. Here is the format for this from the `bwa-mem2` help output.

```
Usage: bwa-mem2 mem -o [Output SAM file name] [other options] <idxbase> <in1.fq> [in2.fq] 
```

Align every sample that you QC'd earlier. This is a great time for a loop.
