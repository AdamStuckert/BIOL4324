## Population genomics

Today we are going to be doing some basic population genomics. This is often some of the first  types of things people look at when doing genomic analyses, either between treatments or in natural populations. Here, we are going to be using a small dataset of samples from *D. sechellia* whole genome resequence data that are on NCBI. We have already trimmed and mapped these reads, and used them before in our mapping lab. Today, we are going to move on to the next steps to run some basic population genomic analyses.

We will be taking our aligned files (bam format) and calling genotypes from those alignment files. Calling genotypes is one of the first steps to more genomic analyses because then we know the DNA sequence of the individuals that we sampled. We can use those to look at signatures of selection, identify candidate causative loci for phenotypes or cancer via GWAS, etc. 

## Call genotypes

Now that we have aligned the whole genome sequence data we have to our reference genome, we can call genotypes (ie, each, probably diploid, base in the genome). There are a variety of pipelines for this, but today we will use software called `Angsd`. Angsd outputs genotype likelihoods based on sequencing depth, the bases observed, and phred scores, whereas other software "hard calls" genotypes (ie, does not include a metric of uncertainty). Downstream applications can take advantage of this uncertainty. This is particularly useful for low-coverage WGS data. 

For angsd, we will need to supply several things, including a list of aligned files to genotype from with each bam file on its own line. The bam files are here: `/project/stuckert/bioinformatics/popgen`. The angsd executable is here: `/project/stuckert/bioinformatics/software/angsd/angsd`. I've put parameters that you want to modify in `[]`, the rest I supply. 
* `-minMapQ 30 -minQ 30` filter by mapping quality and base quality 
* `-doMajorMinor 1` calculate major and minor alleles
* `doMaf 1 -minMaf 0.05` calculate minor allele frequency, filter for minor alleles present at least 5% of the time
* `-skipTriallelic 1 ` skip loci with 3 alleles
* `-SNP_pval 1e-6 ` filter for SNPs that have a low p-value for being polymorphic
* `-doGlf 2 -doBcf 1` Print a compressed beagle and a BCF file
* `-dopost 1  -dogeno 1 -docounts 1` Flags to get a hard-called genotype file

```
angsd -GL 1 -out [ ... Out files ... ] -nThreads 5 \
-ref [ ... genome ... ] \
-bam [ ... text file of bam files ... ] \
-minMapQ 30 -minQ 20  \
-doMajorMinor 1 \
-SNP_pval 1e-6 -doMaf 1 \
-minMaf 0.05 \
-skipTriallelic 1 \
-doGlf 2 \
-doBcf 1 \
-dopost 1  -dogeno 1 -docounts 1 \
-r [ ... Input region(s) ... ]
```

The genome we used to align to is here: `/project/stuckert/bioinformatics/popgen/D.sechellia.fa` 

## Population structure

We will be doing some really basic population genomic analyses as a starting point. Often these are some of the first things I personally do when we get our genomic data. We will be looking at two common analyses, a PCA (which we covered last week) and what is called an admixture or structure plot. The latter infers the number of ancestry units in your data via SNP data and shows what proportion of the genome of a sample is derived from each of those ancestral genotypes. To do this, we will use the beagle file you got from `angsd` and run `pcangsd`. Download pcangsd this way:

```
# you need a newer version of python
module load python/3.12
git clone https://github.com/Rosemeis/pcangsd.git
cd pcangsd
python setup.py build_ext --inplace

```

Once you run this, you should be able to call it as just `pcangsd`, assuming its in your `$PATH`. The call is straightforward, but make sure that you specify `--admix` to get admixture proportions.

```
pcangsd  -b [ ... beagle file ... ] --admix --out [ ... output naming schema ... ] 
```

The most important files are the `*.cov` and `*.admix.*.Q` files. We will use these for PCA and admixture plots, respectively. Download both of them to your local computer.

## Fst analyses

For this, we will be using `VCFtools`, which is a common program to filter, subset, and analyze vcf files. VCF files are "variant call format" files, and include genotypes and variants. Often, we are interested primarily in places where there are *differences in the genome*.

Download BCFtools and VCFtools:

```
# BCFtools
https://github.com/samtools/bcftools/releases/download/1.24/bcftools-1.24.tar.bz2
cd bcftools-1.24/
./configure --prefix=$HOME/.local
make
make install
cd ..

# VCFtools
git clone https://github.com/vcftools/vcftools.git
cd vcftools
./autogen.sh
./configure --prefix=$HOME/.local
make
make install
cd ..
```

First thing we need to do is subset our variant call data. You asked angsd to give you a bcf file, so we will use that. But first, we need to figure out which flies are in each population. Our fly metadata is in `/project/stuckert/bioinformatics/popgen/flies.tsv`. Let's look at genomic divergence between the islands of Anro and PNF. We can easily get the specific flies we need via a one-liner combining `grep` to search files and `cut` to...cut them. For example, for PNF:

```
grep "PNF" flies.tsv | cut -f1 -d " " 
# cut gets first field (-f1) and uses a space as a delimiter (-d " ")
```

Now, that tells us what flies we need from PNF. But, at least my bcf has different naming schema, including the absolute path of the bams. Check that using bcftools to query the samples:

```
bcftools query -l [ ... bcf ... ]
```

Assuming you have a file of your PNF flies, you can pipe the output of `bcftools query` into grep and search the file of flies using `-f file`. 

```
bcftools query -l CM015602.1_out.bcf | grep -f [ ... PNF samples ... ] -
```

`bcftools query` can also be used to list a lot of other things about samples. For example, if I want to get all the chromosomes in the file, I could do:

```
bcftools query -f "%CHROM" [ ... bcf file ... ] |uniq
# `uniq` gets unique lines which represents chromosomes in this case
```

We can also use the sample names we extracted from our bcf file to subset our bcf file using bcftools (note that we are outputting a compressed vcf file here):

```
bcftools view -S [ ... sample file ... ] [ ... bcf file ... ] -O z -o [out].vcf.gz
```

For now, we are going to convert our bcf into a vcf so that we can run `vcftools`. Most of population genomics is just...converting data between file formats.

```
bcftools view  [ ... bcf file ... ] -O z -o [out].vcf.gz
```

We've now explored bcf files, now we will use `vcftools` to compute Fst between our Anro and PNF populations. We are goin

```
./vcftools --gzvcf [ ... input_data.vcf.gz ... ] --weir-fst-pop population_1.txt --weir-fst-pop population_2.txt --out pop1_vs_pop2 \
  --fst-window-size 50000 \
  --fst-window-step 10000 
```

You will need to download ### FILE
## Plotting in R

Lets go to R! First thing we will do is look at the PCA of the data.

```R
# required packages
library(tidyverse)
library(ggplot2)

# read in data from pcangsd
C <- as.matrix(read.table("pcangsd.cov")) # substitute you cov file

# calculate variance for PC1 and PC2
e <- eigen(C)
eigvals <- e$values
totvar <- sum(eigvals)
pc1var <- round((eigvals[1]/totvar)*100, 1)
pc2var <- round((eigvals[2]/totvar)*100, 1)

# pull out pc1 and pc2
pcadat <- e$vectors[,1:2]
colnames(pcadat) <- c("PC1","PC2")

# merge pc1/pc2 with metadata, but use corrected metadata:
# read in samples IN ORDER of bams for angsd/pcangsd
bams <- read.table("bams.txt",  sep = "\t")
colnames(bams) <- "id" # name the column in our dataframe

# I had to rename my samples, because I don't want the path data
# I used gsub, which takes a pattern and replaces it
# For these commands, yours may be different. But what you want is to be able to **match** your bam order to your sample order in the metadata, otherwise your results are nonsense
bams$id <- gsub(pattern = "^.*/", replacement = "", bams$id) # This goes from the start of the line (^), then greedily takes any character (.*) up until the **last** "/" in the string
bams$id <- gsub(pattern = ".sorted.bam", replacement = "", bams$id) # this just removes the end of my bam names, yours may be different

 # Read raw string directly into a data frame
flies <- read.table(text = "
SRR5860573 Anro
SRR5860583 PNF
SRR5860625 Denis
SRR5860626 Denis
SRR5860627 Denis
SRR5860628 Denis
SRR5860629 Denis
SRR5860630 Denis
SRR5860631 Denis
SRR5860632 Denis
SRR5860633 Denis
SRR5860634 Denis
SRR5860639 Denis
SRR5860640 Denis
SRR5860643 Denis
SRR5860644 Denis
SRR5860645 Anro
SRR5860660 Anro
SRR5860661 Anro
SRR5860663 Anro
SRR5860665 Anro
SRR5860666 PNF
SRR5860667 PNF
SRR5860668 Denis
SRR5860669 Denis
SRR5860670 Denis
SRR5860671 Denis
SRR5860672 PNF
SRR5860673 PNF
SRR5860674 PNF
SRR5860675 PNF
SRR5860676 Anro
", header = FALSE, col.names = c("id", "locality"))

# merge the two dataframes **in the order of the bam files!!**
flies <- left_join(bams, flies,by="id")

# bind the pca data columns to our fly metadata
pcadat <- cbind(pcadat, flies) 

# now wei will use ggplot to ... plot
pca <-   ggplot(pcadat, aes(x = PC1, y = PC2, color = locality)) + # this tells ggplot our dataframe, and then what x and y to plot, as well as what variable to plot the colors by
    geom_point(size = 6, show.legend = TRUE, alpha = 0.5) +  # this adds points for our samples
      xlab(paste0("PC1 (", pc1var, "%)")) + # x label with the proportion of variation explained
    ylab(paste0("PC2 (", pc2var, "%)")) +
    theme_bw() + theme(text=element_text(size=23)) + # this is text size modification
  scale_color_manual(values = c("blue","red", "black"))  # specify colors
# ,legend.position="none"
pca

```

Next up, we will examine admixture via a stucture plot:

```

# import best K information
admix <- as.matrix(read.table("pcangsd.admix.5.Q"))
colnames(admix) <- c("genotype1", "genotype2") # you need to add as many genotypes as you have


admix <- cbind(admix, flies) # bind fly data to admix, it should be in the same order
admix$dorsal_pattern <- as.factor(admix$dorsal_pattern)

# now I need to mutate the data for plotting
admix2 <- admix %>% pivot_longer(cols = starts_with("geno"), names_to="genotype", values_to="probability")


str_plot <-
  ggplot(admix2, aes(probability, factor(id), fill = factor(genotype))) +
  geom_col(color = "gray", size = 0.1) +
  facet_grid(dorsal_pattern ~ ., switch = "y", scales = "free", space = "free") + # facet grid for each locality
  theme_minimal() + 
  labs(y = "Individuals", x = "Genotype probability") +
  # Added a 5% expansion to the right side of the x-axis
  scale_x_continuous(expand = expansion(mult = c(0, 0.05))) +
  scale_y_discrete(expand = expansion(add = 1)) +
  theme(
    panel.spacing.y = unit(0.1, "lines"),
    axis.text.y = element_blank(),
    panel.grid = element_blank(),
    legend.position = "none",
    strip.text.y.left = element_text(angle = 0),
    text = element_text(size = 23),
    # Added plot.margin to create a buffer on the right side of the image
    plot.margin = margin(t = 10, r = 40, b = 10, l = 10) 
  ) 

str_plot
```

Finally, we will examine genomic divergence between two populations using our Fst results from VCFtools. For this we will use the `qqman` package, which can make a Manhattan plot for us easily.

```R
library(qqman)

# import data
fst <- read.table("PNF_vs_Anro_50kb.windowed.weir.fst", header = TRUE)

# Create a dummy SNP column if it's missing (needed by qqman)
fst$SNP <- paste0("snp_", 1:nrow(fst))

# set chromosome to numeric (required by qqman library)
fst$CHROM <- 1

# remove rows with NA for Fst value:
fst <- drop_na(fst, MEAN_FST)

# Generate the plot
manhattan(
  fst,
  chr = "CHROM",
  bp = "BIN_START",
  p = "MEAN_FST",
  snp = "SNP",
  logp = FALSE,          # plots raw FST values (0 to 1)
  ylab = expression(F[ST])
) 
```




