## Lab 8: Population genomics 

For today's lab we will be doing some rudimentary analyses of population genomics with some data from a super cool group of frogs. 

### Background information

My work deals a lot with the evolution and maintenance of mimicry in this really cool poison frog that lives in Peru. It is called _Ranitomeya imitator_, or the mimic poison frog (because it mimics other species, obviously). There are 4 mimetic morphs of this species.

![Mimetic morphs of Ranitomeya imitator](https://github.com/AdamStuckert/Gen711/blob/master/Lab/Files/imitator_morphs.png)

Because of it's very interesting evolutionary history, we are very interested in the genomics of coloration and toxicity in this species. Between these pure mimetic morphs are hybrid zones or clinal zones in which the frogs are super phenotypically variable because there seems to be some gene flow between morphs. In 2017 I sampled along one of the hybrid zones between the orange banded morph (top left frog) and the yellow striped morph (top right frog). I sampled 2 pure yellow morph populations, 2 pure orange morph populations, and 2 populations in the hybrid zone. 

![Mimetic morphs of Ranitomeya imitator](https://github.com/AdamStuckert/Gen711/blob/master/Lab/Files/combined_map_figure.png)

For the purposes of this lab, what I've done is selected 5 random individuals from the most distant yellow and orange morph populations. I then subsampled the data down to XX random reads for each individual. I did this just so you all can actually run this in a reasonable time frame (you are welcome!).

OK, background over, time to do some population genomics.

The data for this lab live here: `/project/stuckert/bioinformatics/RAD_data`.

**Question 1:** How many reads are they, and how many reads do they each have? If you are using `grep`, there is an equivalent version that allows use of gzipped files called `zgrep`.

There are a few programs that people use to analyze RADseq data, most notably Stacks and ipyrad. For this lab we will be using Stacks. What Stacks does is assemble data from reads into "stacks" of reads that align (hence the name). Conveniently, Stacks can run both to a genome as well as produce these stacks de novo. So you can utilize extant genomic resources for your species of interest, but if those don't exist you can still examine RADseq data in Stacks. 

We will be downloading and installing Stacks from source. Download the program from the [Stacks website](https://catchenlab.life.illinois.edu/stacks/) and [read the manual to understand installation and usage](https://catchenlab.life.illinois.edu/stacks/manual/). **Please note that you will want to use the prefix option in the configure command. Specify a directory you have read permission in.** If you do not do this and follow the install instructions closely then everything will fail later which is no fun for you.


The first step in Stacks is to demultiplex samples + make sure the appropriate restriction enzyme cut site exists. This will drop any reads that are inappropriately attributed to a sample/don't have a cut site. Our reads were demultiplexed by the sequencing core facility, so we don't need to demultiplex, but we do need to check and verify that our cut sites exist for each read. Stacks has an executable for this called `process_radtags`. 

```bash
# We want to put processed reads somewhere:
mkdir processed_reads
# check for cut sites
process_radtags -p ./reads/ --paired  -i gzfastq -o ./processed_reads/ --renz_1 sbfI --renz_2 claI 
```

**Question 2:** What do the parameters `--renz_1` and `--renz_2` refer to? Why are they important for this process? Stacks documentation: http://catchenlab.life.illinois.edu/stacks/manual


Stacks can calculate a variety of population genetics metrics between populations. In order to do this we have to specify populations. In this case we have 2 populations of frogs, each of which represents a color morph of our poison frog. Each read is named in a way to give us lots of information about the sample, including year collected, locality, and individual number. 

**Question 3:** What format are reads in, and if you had to guess what do you think the different parts of the read names refer to?

In order for Stacks to calculate population metrics we have to give it information about which population each sample belongs to. So, lets make a population map with our data. Our data come from two populations that refer to two color morphs. We will specify them as the color morph instead of population, becuase that is ultimately what we (read: Adam) cares about here. Population information.

SA == Sauce == orange banded morph
PO == Pongo == yellow striped morph


```bash
# make the population list...
inds=$(basename -a processed_reads/*R1_001.1.fq.gz | sed "s/_R1_001.1.fq.gz//g")
```

I haven't gone over the program `sed` yet, but it is a string editor. I often use it to remove common strings. In this case, I am removing the end of the read names (`_R1_001.1.fq.gz`) to get each sample ID.  

**Question 4:** What is inds in the above example?

Now that  you have a list of all the sample IDs, write a for loop that iterates through all of the sample IDs and populates a ... population list. Use the following two lines of code within your for loop.

```bash
# make the population list...
pop=$(echo $ind | cut -d - -f2 | sed "s/PO/yellow-striped/g" | sed "s/SA/orange-banded/g")
printf "%s\t%s\n" "$ind" "$pop" >> poplist.tab
```

**Question 5:** Paste in your for loop. 

**Question 6:** Paste in the first 10 lines of your population file, and the code that you used to print this to screen. 

**Question 7:** What does the content of your population map tell you? 

Hopefully you are now beginning to see the utility of conditionals!

Next up: run Stacks! This program is super nice and integrates all the steps into a single command. In this case we are going to run a de novo Stacks assembly. We actually have a brand new, chromosome-level genome for this species, but that is extra steps for you that will take some time! So, we will skip that and run de novo.


```bash
mkdir stacks_out
denovo_map.pl -T 24 -M 6 -o ./stacks_out/ --samples ./processed_reads/ --popmap ./poplist.tab --paired
```

Note: if you get an error about a `_BINDIR_`, this is an issue with the STACKS installation. Lets fix this.

```bash
sed -i "s&_BINDIR_&YOUR_BIN_PATH&" /PATH2STACKS/stacks-2.65/scripts/denovo_map.pl
# Change  “YOUR_BIN_PATH” to be where the stacks bin is (probably and /PATH2STACKS/stacks-2.65/bin) and “PATH2STACKS”
```

**Question 8: * What happened??? Paste in your output.

If I am being honest, I mostly made you do that because I am petty. I had to figure out what was going on, so you had to at least see what happened.

So I guess we have to fix the read names. Fine. 

First off, it makes a bunch of empty fastq files. Just delete those `rm processed_reads/*fastq`. Now a bit of code to rename all the gzipped files.

```bash
cd processed_reads/
inds=$(ls *R1_001.1.fq.gz | sed "s/_R1_001.1.fq.gz//g")

for ind in $inds
do
mv "$ind"_R1_001.1.fq.gz "$ind".1.fq.gz 
mv "$ind"_R2_001.2.fq.gz "$ind".2.fq.gz 
done
```

Now rerun the de novo pipeline (remember where you are in your system). In order to do this, you will need to remake your population map.

```bash
# delete the population map
rm poplist.tab
# make the population list...
inds=$(basename -a processed_reads/*.1.fq.gz | sed "s/.1.fq.gz//g")

# YOUR for loop here
```

Double check that this is the right number of lines with the right number of files. If there are too many, you may have to look at the file names and use `grep -v` 

```bash
denovo_map.pl -T 5 -M 6 -o ./stacks_out/ --samples ./processed_reads/ --popmap ./poplist.tab --paired
```

We are probably interested in the overall difference between populations. How different are they in the overall genome? One way we examine this is the fixation index, which we can calculate via the population commmand in stacks. Including teh `--fstats` flag will give us some fixation index output.

```bash
populations -P ./stacks_out -t 24 -M ./poplist.tab --fstats
```

**Question 9:** What is the Fst between populations in this study?

**Question 10:** What do you think this means about these two color morphs? Are they genetically similar or dissimilar? Why do you think this? Note: you may have to watch the pop gen lectures and/or read up on Fst values in order to answer this!


**Question 11:** What does Stacks actually do?
