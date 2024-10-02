## Genome assembly

Today we will be using public data to assemble and assess the genome of the fruit fly *Drosophila ananassae*. These data come from [Tvedte et al. 2021](https://academic.oup.com/g3journal/article/11/6/jkab083/6188627?login=true#304735456). Please note that I have already downloaded these raw data for you, and additionally I have subsampled the data down substantially so that we will be able to run these assemblies in a short timeframe with limited resources. As a result of this, you should not expect to get particularly amazing genome assemblies!

As an aside, I suggest creating a new directory for every lab we do (e.g., something called "lab6" or "genome_assembly") so that you don't have a bajillion files in your home directory. That can get reaaaaaaally confusing, reallllllly quickly.


## Software installs

Wahoo! Today is the first day y'all will deal with software installs. Welcome to hell.

Just kidding, today's is actually really easy. We will download software a few ways. First, we will pull a container "image" via `singularity`. The maintainers of the UH Carya cluster have already installed singularity for us. We can create "containers" which are self-contained units of software. You can think of it like a shipping container on a big ship making a transatlantic crossing with goods (this is why one software management program is called "docker", because it ships containers). The only caveat is that they are self-contained and need to be run in a particular manner.

Now, install some software. This may take a while...but luckily for you I tested which software packages take forever/are difficult to install and I made this easier/less time consuming for you so you don't spend the whole lab installing software.

```bash
VERSION=0.2.6
singularity pull --dir $(pwd) compleasm.sif docker://huangnengcsu/compleasm:v${VERSION}
```
**Question 1:** `--dir` tells singularity where to download it. Where did it get downloaded?

**Question 2:** How did you tell it where to download this, and what does the syntax mean/do?

Singularity is a bit weird, in that it is entirely self-contained. As a result, you need to "bind" directories that you want to interact with the singularity image. These are comma delimited. A typical example looks like this:

```bash
singularity exec -B ${PWD},/other/directory/tobind/,/a/third/directory/tobind /project/stuckert/software/salmon_1.10.3.sif COMMAND
```

You specify the command (`singularity`), tell it to execute a command (`exec`), bind directories with `-B`, specify the image (the `.sif` file), and give the command of the program you are running.

Install `SPades`. This is an assembler for a variety of things, including Illumina data of small genomes.

```bash
wget https://github.com/ablab/spades/releases/download/v4.0.0/SPAdes-4.0.0-Linux.tar.gz
tar -xzf SPAdes-4.0.0-Linux.tar.gz # the downloaded file is a "tarball" which you have to extract (like a zipped folder)
cd SPAdes-4.0.0-Linux/bin/ # this is the directory your program lives in.

```

Test this install out immediately! Run `./spades.py`. If you get the help message from SPades then your install isn't immediately obviously broken. You can and SHOULD test software installs before using them on real data. Software often comes with toy datasets to use. For today, we will "test" it with our small datasets.


**Question 3:**  What kind of file is `spades.py`?


Install `Hifiasm`. This program is developed to assemble genomes from accurate long read data. Follow the installation instructions on the [Hifiasm GitHub page](https://github.com/chhylp123/hifiasm).

Test this install out immediately! Run `hifiasm`.

**Question 4:** Did your SPades and Hifiasm installs work?

If your installs worked, we are now ready to proceed to the analyses/genome assemblies.

### A quick word about "arguments"  

An argument is a piece of information that you give a program you want to run. This can be almost anything - a parameter it needs to know to get the analysis right, an input or output file, the path to a database the program will need, etc. These arguments can be denoted in two different ways. They can be **named** and have a "flag" associated with them: `-i` for the input file, for example. Or they can be **positional** which means that the program knows what you want it to do with that information based on the order that you give. For example, in the command `grep ">" sequence.fasta`, grep knows that `">"` is the thing you want to search for, and `sequence.fasta` is the thing you want to search inside based on the order that you gave it those pieces of information.  

Another thing to know about named arguments in particular is that sometimes there will be a flag that needs something to come after it, and sometimes it is fine on it's own. For example, if `-i` stands for input file (or input directory), then you should give it a file or directory name after the flag (separated by a space). If the flag is something like `-v` in grep (that searches for the inverse of what you specify), then it functions more as a switch that you can turn on or off so that the program will behave in one way or another. The only way to know for sure is to check the manual (which for many programs, can be found online, or by typing the name of the command followed by `--help`).  

Try to watch for these as you go through this lab. It will help you start to know how to put commands together on your own.  
  
  
Now make sure you are in your home directory and we'll get into the actual lab.


## Lab 4 - Genome Assembly - the meat and potatoes

As mentioned above, today in lab we are going to be assembling a fruit fly (*Drosophila ananassae*) in two different ways. First, with Illumina reads and then using PacBio HiFi reads (highly accurate long reads).

We will then evaluate each genome assembly using `assemblathon_stats.pl` to assess contiguity and `compleasm` to assess genic content. We will use the outputs from these programs to compare the two assemblies.

### Assembling the Illumina reads  
  
This program was developed in part by the Russian man whose De Bruijn graph videos we've been watching in class. His name is Pavel Pevzner.  
This command will take a little while to run, so be patient. It will give you updates as it is running, and you should see if you understand some of the messages it will print to the screen.  
Here is the SPAdes github page, if you're interested in more of the options: https://github.com/ablab/spades#sec2 
- The options you see with `-pe` or `-mp` at the beginning refer to "paired end" or "mate pair" data, like we talked about in class.
- The `-m` is a memory limit for when SPAdes is running. This will depend on what size computer you are running the program.

**Question 5:** Can you guess what information `-k` gives this program? You can check the github page above to see if you are right.

- The `-o` option is specifying the output, as usual, but this time it is an output directory, so make sure you name it accordingly (and don't include a file extension).  

**Remember, you must run tasks on the compute node, not the login node. Please do not run any assemblies on the head/login node!**

**Question 6** What is the header you are using in your script to tell SLURM what resources it needs?


The data for this assembly are here: `/project/stuckert/bioinformatics/lab4`. These are the files with `Illumina` in the file name. Remember, Illumina data is _usually paired end_ so there are two sets of reads, a forward and reverse. 

`spades.py -t 5 -m 55 --mp1-rf -k 95  --pe1-1 FORWARD_READS  --pe1-2 REVERSE_READS  -o illumina_assembly`

Note, the syntax may have changed some since I wrote this syntax out. You may need to update arguments to reflect this. You can get information on the program from either its website OR by running `spades.py -h`. Remember, you need to make sure you are specifying where your spades file is...
  
#### Evaluating our short read assembly  

First we can run a perl script called `assemblathon_stats.pl`. This is a super fast script that can tell us about total bp, contiguity, and scaffold/contig statistics. I frequently use this in my own work.

```bash
/project/stuckert/software/assemblathon_stats.pl GENOME_ASSEMBLY # sub in your assembly name for this on the far right
```

**Question 7:** How many total base pairs is your assembly? What is the contig N50?

**Question 8:** Look up the *D. ananassee* true genome size online. How does our assembly compare to the estimated genome size from other sources?

Now we will assess genic content using `compleasm`. Compleasm is built on `BUSCO`, a [software program](https://busco.ezlab.org/) designed to look at genic content using conserved orthologs. The fundamental idea is that within a particular lineage, many genes are conserved between species. Genes can be gained and lost over evolutionary history, so more closely related species are more likely to have similar sets of genes. If a genome assembly has the vast majority of these genes, then it is likely to be more complete. If an assembly has a poor proportion of those genes, then it is likely not very complete. Choosing the best lineage is important for this type of analysis. You could choose "vertebrata" for a frog, but there are so many vertebrates with varied gene content that it limits the applicability to test genic completeness. This is slowly becoming less of an issue as more genomes are sequenced, annotated, and added to databases, and it is becoming easier to choose a more appropriate lineage. For more details, [read a recent BUSCO paper](https://academic.oup.com/mbe/article/38/10/4647/6329644).

Look up *Drosophila*'s classification, and then choose the appropriate lineage from [BUSCO's lineage datasets](https://busco-data.ezlab.org/v5/data/lineages/).

**Question 9:** What lineage did you choose?

You need to download this data. You can auto-download data with `compleasm`, but many HPCs restrict or don't have compute nodes with internet access to search/download files via SLURM scripts. So lets do that outside of a job (this is a quick download so it is fine to run on the head node). You will need to activate your conda environment where you installed compleasm.

```
singularity exec -B ${PWD},/other/directory/tobind/,/a/third/directory/tobind /path/to/image/compleasm.sif download LINEAGE_YOU_CHOSE --library_path .
conda activate ENV
compleasm download LINEAGE_YOU_CHOSE --library_path . # here the "." refers to the current directory. so it will download the dataset to where you currently are. This is common syntax. You may want to download somewhere else though, which you can specify.
```


Ok, now back to compleasm. Code will be similar to this, change all the variables for your assemnbly, lineage, etc:

```bash
compleasm run --assembly_path $ASSEMBLY \
--output_dir $BUSCO_OUT --threads 5 --lineage $LINEAGE \
--library_path $LIBRARY # you may need a "-m lite" flag" --even still it may not work if the assembly is too poor
```

**Question 10:** Paste in your slurm script. 

### Assembling the PacBio reads

We are working with highly accurate HiFi data from PacBio. These are long, but accurate, the best of both worlds. We will be using `Hifiasm` to assemble these. There are a variety of programs for this, but this is fast and works well.

You need to use `-f0` in you hifiasm command so that we disable the memory intensive first step.

For Hifiasm, I want you to refer to the [Hifiasm GitHub page](https://github.com/chhylp123/hifiasm) to run this. Again, your data are in the directory `D.ananassae.HiFi.sub.25k.fq`, and they have `HiFi` in the name. Make sure that you use the `awk` command they mention to produce the genomic `fasta` file. 

**Question 11:** Paste in your hifiasm script.

Now, run `assemblathon_stats.pl` and `compleasm` on your long read assembly.

### Comparing the two assemblies  

We will use a file from each of our evaluation commands to compare these two assemblies. Check the output files of your slurm runs to figure this out.


**Question 12:** Which assembly is better, and which do you have more confidence in. Why? Dicuss the results from both genome assessment methods.
