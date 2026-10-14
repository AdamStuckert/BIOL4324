

Today we will be exploring gene expression. We will start with raw reads and end up with a small differential expression analysis. Today is the first time we will be working in R. R is commonly used for statistical analyses and visualization (ie, plotting). 

For today's lab, we are working with samples that have been sequenced to explore gene expression in fat bodies and hemocytes of *D. sechellia*. These are found in `/project/stuckert/bioinformatics/rnaseq`.


|Sample Accession|Experiment Title|
|SRR18584878|larval Fat body|
|SRR18584879|larval Hemocytes|
|SRR18584880|larval Fat body|
|SRR18584881|larval Hemocytes|
|SRR18584888|larval Fat body|
|SRR18584891|larval Fat body|
|SRR18584892|larval Hemocytes|
|SRR18584893|larval Fat body|
|SRR18584894|larval Hemocytes|
|SRR18584899|larval Hemocytes|
|SRR18584904|larval Fat body|
|SRR18584905|larval Hemocytes|
|SRR18584906|larval Fat body|
|SRR18584907|larval Hemocytes|
|SRR18584910|larval Fat body|
|SRR18584911	|larval Hemocytes|


There are many things people might use gene expression for, including but not limited to: 
* Producing a transcriptome
* Coding evidence for annotating a genome (which is basically also producing a transcriptome)
* Comparative transcriptomics
* Broadly linking expression to phenotype, including differential gene expression studies and splicing analyses

Five or ten years ago **most** gene expression studies were either for *de novo* transcriptome assembly or used a transcriptome for gene expression analyses. Now, we are finally on the cusp of broadly using only genomes to map to. The last time I taught a version of this course in 2024, I taught transcriptome assembly, but no longer thing this is particularly useful for you. So, we are going to run these analyses using the whole genome sequence. For today's lab, we will be mapping to the *D. sechellia* genome (like last week!), but here we need to use a *splice aware* mapping program. This is because we are dealing with organisms that splice out introns, so we are not mapping to contiguous chunks of DNA. The plan for today's lab is:

1. QC of reads (exactly what we did last week!)
2. Trimming reads (exactly what we did last week!)
3. Indexing the genome (almost exactly what we did last week)
4. Aligning to the genome
5. Counting gene reads
6. Quick differential expression analysis between tissue types in R

So, today you will have two scripts, a bash script and an R script.

## QC and read trimming

You did this last week, integrate it with your new data. Please check the data files before running, there is something slightly different about this week's data (cough cough hint cough cough).

## Index the genome

RNA-seq is different from WGS reads in that we need to use a *splice aware* aligner because we are skipping introns. Today we will use Spliced Transcripts Alignment to a Reference (`STAR`). First, install:

```
wget https://github.com/alexdobin/STAR/archive/2.7.11b.tar.gz
tar -xzf 2.7.11b.tar.gz
cd STAR-2.7.11b

# check in the `Linux_x86_64` directory
```

Now, you will generate a genome index. For big genomes, this can take a long time and a lot of memory. But we are using small genomes, so not that big of a deal for us. Still, use a compute node.

```
   STAR --runMode genomeGenerate \
    --genomeDir [ ... ] \
    --genomeFastaFiles [ ... ] \
    --sjdbGTFfile [ ... ] [ ... other parameters ... ]
```

You need the genome fasta file and the annotation file (a general feature file, or GFF file). I have both of those for you here: `/project/stuckert/bioinformatics/rnaseq`. Find them in that directory.
## Align reads to the genome

You will then map each read to the genome using `STAR`.

```
 STAR --runMode alignReads \
        --genomeDir [ ... ] \
        --readFilesIn [ ... ] \
        --readFilesCommand cat \    
        --outSAMtype BAM Unsorted \
        [ ... other parameters ... ]
```

Like last week, at this point you should have a sam/bam file (depending on what you asked STAR  to give you). In the above I asked for an unsorted bam file.


## Tabulate gene counts

Next, we count the number of times a gene is expressed for each sample. Note, this is for a simple *gene-level analysis*, but you could (and maybe should) do a splice-aware analysis to look at transcript or isoform analyses as well. But for today, we will be focusing on a gene level analysis. To count our expression, we will be using `htseq`. This is a python program, and we can download it easily via python's `pip`.

```
pip install htseq
```

You can then count reads mapping to specific genes by specifying your output alignment files from `STAR` and your GFF file (the annotation file that indicates what are genes).

```
        # run htseq-count
        htseq-count -f bam \
        -t exon \
        -i Parent [ ... ] [ ... GFF ... ] > [ ... gene.counts ... ]
        
# after everything is finished, please rm bam files to free up disk space

```

Paste together the files. This example uses outputs from `htseq-count` that end in `*gene.counts`. Adjust as needed for your output.

```
# count as a base iteration
count=1
for counts in *.sam.gene.counts
do
	if [ $count -eq 1 ]
	then
		printf "gene\t$(echo $counts)\n" > gene.table.tsv
		cat $counts >> gene.table.tsv
		# iterate count to escape the first if/then statement
		# the point of this is to get the gene row names.
		((count++))
	else
		printf "\t$(echo $counts)\n" > tmp
		awk '{print $2}' $counts >> tmp
		paste gene.table.tsv tmp > tmp2
		mv tmp2 gene.table.tsv
	fi
done
```

You have a table of gene counts! Great job! Now we will want to look at differential gene expression. We will do this in R. I prefer to use R on my computer instead of on the cluster because I like the more interactive ability to see plots and statistical outputs. Your choice. Download the count data if you need to and open R/R studio. You will need to download and install some packages first.

```
# install BiocManager if you haven't already 

if (!require("BiocManager", quietly = TRUE)) { install.packages("BiocManager") } 

# install DESeq2 
BiocManager::install("DESeq2")

# install ggplot2
install.packages("ggplot2")

# load libraries
library(DESeq2)
library(ggplot2)
```


Now, we will import the data. In R a `<-` symbol assigns the right side of the arrow to the left (sort of like creating a variable with the "=" in bash, but not equivalent).  So download the data and open it up:

```
counts_matrix <- read.table("gene.table.tsv", header = TRUE, sep = "", row.names = 1)
# reads a tab delimited file and saves the first column as the row names in a data frame
```

Next, you will create a sample dataframe for the data (i've done this in the code) and create a DESeq2 object.

```
# Create a metadata table where row names match the column names of counts_matrix
colData <- data.frame(
  tissue = factor(c("larval Fat body", "larval Hemocytes", "larval Fat body", "larval Hemocytes", "larval Fat body", "larval Fat body", "larval Hemocytes", "larval Fat body", "larval Hemocytes", "larval Hemocytes",  "larval Fat body", "larval Hemocytes", "larval Fat body", "larval Hemocytes", "larval Fat body", "larval Hemocytes")),
  row.names = colnames(counts_matrix)
)

# Build the DESeq2 object
dds <- DESeqDataSetFromMatrix(
  countData = counts_matrix,
  colData = colData,
  design = ~ tissue
)
```

Finally, you will do the really preliminary expression analysis that compares the two tissue types.

```
dds <- DESeq(dds)
res <- results(dds)
res
```


Now, you might want to actually *see* what your data look like. One of the most common types of exploratory analyses is called Principal Component Analyses (PCA). Basically what this does is take highly multivariate data (which your gene expression data is) and collapses it into 2 dimensions and plots each sample in 2D space. Ideally, if you have a treatment, different tissue, different whatever, these will fall out into separate clouds in your PCA. So, lets make a PCA of the data:

```
vsd <- vst(dds, blind=FALSE)
plotPCA(vsd, intgroup="tissue") # this is a command in DESeq2; you want to specify the intergroup comparison with intgroup. Above I used tissue as my name, so I use that here too.
```

OK, what did you get, and what is your interpretation??

At this point folks are often interested in what genes are differentially expressed. In general, folks use a p-value threshold of 0.05 (after multiple comparisons correcting, which is required because you are testing many genes) to identify differentially expressed genes. People will often further filter out their results by looking at log-fold changes of expression. DESeq2 has an easy way to both filter and get an indication of how many genes meet our criteria:

```
filt.res <- results(dds, alpha=0.05, lfcThreshold = 2) # filter by both corrected p-value and log-fold change
summary(filt.res) # get summary of results
```

You can also identify the "top differentially expressed genes", which is a common approach. People will often focus on the ones with the greatest absolute value of logfold change. Here is one way to filter this and then plot a gene with the top differential expression.

```
# filter by decreasing LFC
filt.res.sort <- filt.res[order(filt.res$log2FoldChange, decreasing = TRUE), ]
# Show the top genes
head(filt.res.sort) 
```


Now, you can take the output of that and plot the results using a builtin function from DESeq2:

```
plotCounts(dds, gene=[gene name in quotes from above], intgroup="tissue")
```

Plot the top 5 genes.


