## Using phylogenetic tools to create a gene tree

****


For this lab, we will need a way of viewing a phylogenetic tree. While there are some online options for doing this, most of them are not that fun. So I am going to introduce you to a new program for analyses: R (and its counterpart the lovely R studio). First you will need to [download R](https://www.r-project.org/) and then [download R studio](https://posit.co/download/rstudio-desktop). This may seem like a lot, but in reality it is fairly straightforward. We will then need to download several "packages" in R. These are similar to different programs in the linux environment, and each package has several functions within.

In R Studio do this:

```R
if (!require("BiocManager", quietly = TRUE))
    install.packages("BiocManager")

BiocManager::install("ggtree")

install.packages("ape") #installs package ape

library(ape) # loads package ape
library(ggtree) 
```


If this does not work for you, or for whatever reason you are unable to do this, please let me know and we'll get you setup. 

Next you will want to install the other programs we will use via conda. I suggest a fresh environment for todays lab.

```bash
conda install -c bioconda mafft
conda install -c bioconda iqtree
```

On to the lab!  


### First, we need data

Go ahead and log onto your Opuntia account.  

Let's make a new directory for this lab so that your files won't be in with everything else.  

    mkdir phylo_lab  
    cd phylo_lab

Since the genomes and protein sets they have on Ensembl are already annotated with the sequence identities, they are a great place to start for a tree like this. We are going to be working with rodents again, mostly because they are easy to find, pretty diverse, and ensembl has a LOT of them.  

We will need quite a few of them for this tree, and you will have to do some commands to each dataset that we download, so strap in.   Note, **for loops** would be very useful and this could be an excellent chance to use them!

First, download the datasets from these links using some of the commands we've used before (if you can't remember them, look at how we downloaded SPades in lab 4):
- American beaver - ftp://ftp.ensembl.org/pub/release-99/fasta/castor_canadensis/pep/Castor_canadensis.C.can_genome_v1.0.pep.all.fa.gz  
- Arctic ground squirrel - ftp://ftp.ensembl.org/pub/release-99/fasta/urocitellus_parryii/pep/Urocitellus_parryii.ASM342692v1.pep.all.fa.gz  
- Degu - ftp://ftp.ensembl.org/pub/release-99/fasta/octodon_degus/pep/Octodon_degus.OctDeg1.0.pep.all.fa.gz  
- Guinea pig - ftp://ftp.ensembl.org/pub/release-99/fasta/cavia_porcellus/pep/Cavia_porcellus.Cavpor3.0.pep.all.fa.gz  
- Lesser Egyptian jerboa - ftp://ftp.ensembl.org/pub/release-99/fasta/jaculus_jaculus/pep/Jaculus_jaculus.JacJac1.0.pep.all.fa.gz  
- Long-tailed chinchilla - ftp://ftp.ensembl.org/pub/release-99/fasta/chinchilla_lanigera/pep/Chinchilla_lanigera.ChiLan1.0.pep.all.fa.gz  
- Mouse - ftp://ftp.ensembl.org/pub/release-99/fasta/mus_musculus/pep/Mus_musculus.GRCm38.pep.all.fa.gz  
- Naked mole-rat (female) - ftp://ftp.ensembl.org/pub/release-99/fasta/heterocephalus_glaber_female/pep/Heterocephalus_glaber_female.HetGla_female_1.0.pep.all.fa.gz  
- Northern American deer mouse - ftp://ftp.ensembl.org/pub/release-99/fasta/peromyscus_maniculatus_bairdii/pep/Peromyscus_maniculatus_bairdii.HU_Pman_2.1.pep.all.fa.gz  
- Prairie vole - ftp://ftp.ensembl.org/pub/release-99/fasta/microtus_ochrogaster/pep/Microtus_ochrogaster.MicOch1.0.pep.all.fa.gz  
- Rabbit - ftp://ftp.ensembl.org/pub/release-99/fasta/oryctolagus_cuniculus/pep/Oryctolagus_cuniculus.OryCun2.0.pep.all.fa.gz  
- Rat - ftp://ftp.ensembl.org/pub/release-99/fasta/rattus_norvegicus/pep/Rattus_norvegicus.Rnor_6.0.pep.all.fa.gz  
- Squirrel - ftp://ftp.ensembl.org/pub/release-99/fasta/ictidomys_tridecemlineatus/pep/Ictidomys_tridecemlineatus.SpeTri2.0.pep.all.fa.gz  
- Upper Galilee mountains blind mole rat - ftp://ftp.ensembl.org/pub/release-99/fasta/nannospalax_galili/pep/Nannospalax_galili.S.galili_v1.0.pep.all.fa.gz

**Question 1:** One of these organisms is the one we are going to use as the outgroup for this particular phylogenetic tree. It is the only one that is not a rodent. Which is it?   

Next, go ahead and unzip all these files using the `gzip -d` command.  

These files (annoyingly) have sequences that are wrapped - meaning they are not all on one line in the file. We will have to unwrap them before we move on. Below is the code you will need to do the first file; make sure you do this to all of them. Remember that the `>` command redirects the output to a new file.

    awk '{if(NR==1) {print $0} else {if($0 ~ /^>/) {print "\n"$0} else {printf $0}}}' Castor_canadensis.C.can_genome_v1.0.pep.all.fa > beaver_unwrap.fa  


### Pulling out the sequences we want  

Now that we have the data downloaded and its formatting changed, we are ready to extract the sequences we want for this tree. Today we're going to be making a gene tree for the Sonic Hedgehog gene. Yes, this is a real thing. You can read about it if you like (https://ghr.nlm.nih.gov/gene/SHH#synonyms), but it is important in development so we know all of these rodents will have at least one version.  

**Question 2:** Use `grep` to search for this gene ("sonic" works well as a search term) in each rodent file, and put the corresponding header(s) and sequence(s) into a new file **that is specific to that particular animal**. Paste a copy of one of these commands for this question. Make sure each rodent has its ***own*** file with its sonic hedgehog gene sequences inside.  

Take a look at one of these files. There is a lot of information in each of the headers, but most of it is no longer very useful to us. What we *do* want in the headers is a really clear species name, so that we'll be able to tell which sequence is which on the tree later. The command below will change the header of a fasta file; do this to all of the sonic hedgehog files. Note: this works for mine because this is what my file was called - make sure you change the input and output file names to match what you want yours to be called.  

    awk '/^>/{print ">beaver_" ++i; next}{print}' beaver_sonic.fa > header_beaver_sonic.fa  

Alternatively, if you wanted to use a command we've seen before (sed) you can do this:

    sed "s/>.*$/>${spp}/" $file > $spp.fa # replace both instances of $spp and the $file

`sed` is a string editor and it follows this format: 

"s/STRINGTOREPLACE/REPLACEMENTSTRING/". If you want to make it global (ie, across every instance in the file instead of just the first instance per line) you would do this: "s/STRINGTOREPLACE/REPLACEMENTSTRING/g". The `sed` command is much better for use in the for loops (or at least I think easier).

Take a look at your files for each species and the sonic hedgehog gene. If you are finding that you still have multiple sequences, use this command

    sed -n '1,2p' $file > ${file}.dedupd.fa

**Question 3:** Tell me about your experience preparing all these files. Did you write these all out by hand? Or did you choose to prepare these in a different way? What would you do/have done differently if you had to redo this with a dataset with 4x the number of species you are downloading?

**Question 4:** When you have all of your headers edited appropriately, combine all of those files together. The file you end up with should be a single fasta file with all of the sequences for the sonic hedgehog gene, and all of the headers simple and informative. Paste in the code you used to do this.  


### Aligning the sequences and making the tree  

We'll use mafft again with the default settings for the multiple sequence alignment. Put your own input and output file names in the appropriate places.  

    mafft --auto input.fa > output.fa  


And finally, we will use a program called iqtree to make the tree. Iqtree has a **LOT** of options, but we will try to keep things relatively simple for today. Your output file from the last command will be your input file for this one.  

- `-s` is the name of the input file  
- `-m` is the model we will use - this one is just the initials of the people who created it  
- `-bb` tells iqtree ...
- `-pre` is the prefix of all the output files - iqtree will produce a number of files, and this will be the beginning of all of them   

    `iqtree -s input -m LG -bb 1000 -pre output`  

**Question 5:** What is the `-bb` flag and what does it do?

### Viewing the tree  

One of the files iqtree produces has "contree" at the end of it. Cat this file onto the screen, and copy everything you see including all the strange looking punctuation.    
Go to R studio. New language, wahoo!

In R, write this:

    library(ape)
    library(ggtree)
    shh_tree <- "" # BUT PASTE IN YOUR CONTREE WITHIN THE QUOTES
    shh_tree <- read.tree(text=shh_tree)
    # if you need to, you can reroot your tree using this:
    shh_tree <- ape::root.phylo(shh_tree, outgroup = "OUTGROUP NAME")
    ggtree(shh_tree) + geom_tiplab() 


**Question 6:** Paste in your tree file.

**Question 7:** Based on your tree, what is the species most closely related to the beaver?
