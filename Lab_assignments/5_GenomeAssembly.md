
## Genome assembly

### NOTE:

Given resource allocations, please do not run the assemblies on the whole genomic data. Download a SUBSET of the data with the `fastq-dump` or `fasterq-dump` and run your assemblies with that and **NOT** the full dataset.

If you have issues installing hifiasm: `/project/stuckert/bioinformatics/software/hifiasm-0.23.0/hifiasm`

For assembly metric analyses, please use these files:
Spades assembly: `/project/stuckert/astucker/genome_assembly/illumina_assembly/scaffolds.fasta`
hifiasm assembly: `/project/stuckert/astucker/genome_assembly/SRR11442117.hifiasm.bp.p_ctg.fa`


## Genome assembly work

Today we will be using public data to assemble and assess the genome of the fruit fly we worked with last week: *Drosophila sechellia*. These data come from [Tvedte et al. 2021](https://academic.oup.com/g3journal/article/11/6/jkab083/6188627?login=true#304735456). Please note that for this class we are using a small portion of data, less than most people would use for a genome assembly (people often shoot for 30X coverage for assemblies). As a result of this, you should not expect to get particularly amazing genome assemblies!

As an aside, I suggest creating a new directory for every lab we do (e.g., something called "lab5" or "genome_assembly") so that you don't have a bajillion files in your home directory. That can get reaaaaaaally confusing, reallllllly quickly. Good data and file management will save you a lot of hassle in the future.


## Software installs

Wahoo! Today is the first day y'all will deal with software installs. Welcome to hell.

Just kidding, today's is actually really easy. We will download software a few ways. First, we will pull a container "image" via `singularity`. The maintainers of the UH Carya cluster have already installed singularity for us. We can create "containers" which are self-contained units of software. You can think of it like a shipping container on a big ship making a transatlantic crossing with goods (this is why one software management program is called "docker", because it ships containers). The singularity image you download is like a static picture in time of all the software needed for a particular task. The reason the field has moved to these is that it makes computational tasks much more feasible and reproducible. Software that is built on other pieces of software has certain expectations for outputs, the way they work, etc. When new versions iterate, it might break the analytical pipeline. You can imagine that if you had many pieces of software dependent on a single piece of software, and that they all required different versions it could quickly get out of hand. Hence, containers and images. The only caveat is that they are self-contained and need to be run in a particular manner.

Now, install some software. This may take a while...but luckily for you I tested which software packages take forever/are difficult to install and I made this easier/less time consuming for you so you don't spend the whole lab installing software.

```bash
VERSION=0.2.9
singularity pull docker://huangnengcsu/compleasm:v${VERSION}
singularity exec compleasm_v${VERSION}.sif compleasm -h
```

`--dir` tells singularity where to download it. Where did it get downloaded?

Singularity is a bit weird, in that it is entirely self-contained. As a result, you need to "bind" directories that you want to interact with the singularity image. These are comma delimited. A typical example looks like this:

```bash
singularity exec -B ${PWD},/other/directory/tobind/,/a/third/directory/tobind /project/stuckert/software/salmon_1.10.3.sif COMMAND
```

You specify the command (`singularity`), tell it to execute a command (`exec`), bind directories with `-B`, specify the image (the `.sif` file), and give the command of the program you are running. Some people are having issues with an out of memory issue for singularity. You can use this one: `/project/stuckert/bioinformatics/compleasm.sif`

Install `SPades`. This is an assembler for a variety of things, including Illumina data of small genomes.

```bash
wget https://github.com/ablab/spades/releases/download/v4.3.0/SPAdes-4.3.0-Linux.tar.gz
tar -xzf SPAdes-4.3.0-Linux.tar.gz # the downloaded file is a "tarball" which you have to extract (like a zipped folder)
cd SPAdes-4.3.0-Linux/bin/ # this is the directory your program lives in.

```

Test this install out immediately! Run `./spades.py`. If you get the help message from SPades then your install isn't immediately obviously broken. You can and SHOULD test software installs before using them on real data. Software often comes with toy datasets to use. For today, we will "test" it with our small datasets.


What kind of file is `spades.py`?

Install `Hifiasm`. This program is developed to assemble genomes from accurate long read data. Follow the installation instructions on the [Hifiasm GitHub page](https://github.com/chhylp123/hifiasm).

Test this install out immediately! Run `hifiasm`.

If your installs worked, we are now ready to proceed to the analyses/genome assemblies.

### A quick word about "arguments"  

An argument is a piece of information that you give a program you want to run. This can be almost anything - a parameter it needs to know to get the analysis right, an input or output file, the path to a database the program will need, etc. These arguments can be denoted in two different ways. They can be **named** and have a "flag" associated with them: `-i` for the input file, for example. Or they can be **positional** which means that the program knows what you want it to do with that information based on the order that you give. For example, in the command `grep ">" sequence.fasta`, grep knows that `">"` is the thing you want to search for, and `sequence.fasta` is the thing you want to search inside based on the order that you gave it those pieces of information.  

Another thing to know about named arguments in particular is that sometimes there will be a flag that needs something to come after it, and sometimes it is fine on it's own. For example, if `-i` stands for input file (or input directory), then you should give it a file or directory name after the flag (separated by a space). If the flag is something like `-v` in grep (that searches for the inverse of what you specify), then it functions more as a switch that you can turn on or off so that the program will behave in one way or another. The only way to know for sure is to check the manual (which for many programs, can be found online, or by typing the name of the command followed by `--help`).  

Try to watch for these as you go through this lab. It will help you start to know how to put commands together on your own.  
  
  
Now make sure you are in your home directory and we'll get into the actual lab.

## Data downloads

Please download NCBI accession `SRR8840600`. This is a short read Illumina platform whole genome sequence dataset. 

For long read data, we are going to work with PacBio HiFi data. This is the most recent data type from PacBio and it is 🤌. There are no publicly available HiFi data from *D. sechellia* so we are going to work with a congener, *Drosophila ananassae*. We will be using accession `SRR11442117` for today. Please note that this is a big file (~24 Gb), which would take a long time to download and run through the assembler. So, I have provided a very small subsampled, parsed down dataset for you to test your scripts with. All the data are here: `/project/stuckert/bioinformatics/lab5`. This is actually a really good practice the first time you assemble a script/pipeline. This way things can run quickly and its quicker to find errors.

For those of you that are curious, I subsampled using [seqtk](https://github.com/lh3/seqtk) (which is a nice piece of software for working with fasta/fastq files) with this command:

```
seqtk sample -s100  SRR11442117.fastq 50000 >  SRR11442117.subsamp.fastq
```

Please note that I want you to test things with this sparse dataset BUT run your full script with the FULL dataset.

## Lab 5 - Genome Assembly - the meat and potatoes

As mentioned above, today in lab we are going to be assembling a fruit fly (*Drosophila ananassae*) in two different ways. First, with Illumina reads and then using PacBio HiFi reads (highly accurate long reads).

We will then evaluate each genome assembly using `assemblathon_stats.pl` to assess contiguity and `compleasm` to assess genic content. We will use the outputs from these programs to compare the two assemblies.

### Assembling the Illumina reads  
  
This program was developed in part by the Russian man whose De Bruijn graph videos we've been watching in class. His name is Pavel Pevzner.  
This command will take a little while to run, so be patient. It will give you updates as it is running, and you should see if you understand some of the messages it will print to the screen.  
Here is the SPAdes github page, if you're interested in more of the options: https://github.com/ablab/spades#sec2 
- The options you see with `-pe` or `-mp` at the beginning refer to "paired end" or "mate pair" data, like we talked about in class.
- The `-m` is a memory limit for when SPAdes is running. This will depend on what size computer you are running the program.

Can you guess what information `-k` gives this program? You can check the github page above to see if you are right.

- The `-o` option is specifying the output, as usual, but this time it is an output directory, so make sure you name it accordingly (and don't include a file extension).  

**Remember, you must run tasks on the compute node, not the login node. Please do not run any assemblies on the head/login node!**

What is the header you are using in your script to tell SLURM what resources it needs?


Remember, Illumina data is _usually paired end_ so there are two sets of reads, a forward and reverse. 

`spades.py -t 5 -m 55 --mp1-rf -k 95  --pe1-1 FORWARD_READS  --pe1-2 REVERSE_READS  -o illumina_assembly`

Note, the syntax may have changed some since I wrote this syntax out...You may need to update arguments to reflect this. You can get information on the program from either its website OR by running `spades.py -h`. Remember, you need to make sure you are specifying where your spades file is...
  
#### Evaluating our short read assembly  

First we can run a perl script called `assemblathon_stats.pl`. This is a super fast script that can tell us about total bp, contiguity, and scaffold/contig statistics. I frequently use this in my own work.

```bash
export PERL5LIB=/project/stuckert/bioinformatics/software/perl_lib
/project/stuckert/bioinformatics/software/assemblathon_stats.pl GENOME_ASSEMBLY # sub in your assembly name for this on the far right
```

How many total base pairs is your assembly? What is the contig N50?

Look up the *D. sechellia* true genome size online. How does our assembly compare to the estimated genome size from other sources?

Now we will assess genic content using `compleasm`. [Compleasm](https://github.com/huangnengCSU/compleasm) is built on `BUSCO`, a [software program](https://busco.ezlab.org/) designed to look at genic content using conserved orthologs. The fundamental idea is that within a particular lineage, many genes are conserved between species. Genes can be gained and lost over evolutionary history, so more closely related species are more likely to have similar sets of genes. If a genome assembly has the vast majority of these genes, then it is likely to be more complete. If an assembly has a poor proportion of those genes, then it is likely not very complete. Choosing the best lineage is important for this type of analysis. You could choose "vertebrata" for a frog, but there are so many vertebrates with varied gene content that it limits the applicability to test genic completeness. This is slowly becoming less of an issue as more genomes are sequenced, annotated, and added to databases, and it is becoming easier to choose a more appropriate lineage. For more details, [read a recent BUSCO paper](https://academic.oup.com/mbe/article/38/10/4647/6329644).

Look up *Drosophila*'s classification, and then choose the appropriate lineage from [BUSCO's lineage datasets](https://busco-data.ezlab.org/v5/data/lineages/).

What lineage did you choose?

You need to download this data. You can auto-download data with `compleasm`, but many HPCs restrict or don't have compute nodes with internet access to search/download files via SLURM scripts. So lets do that outside of a job (this is a quick download so it is fine to run on the head node). Compleasm has a method to auto-download the busco data using the `download` command. Choose the lineage you want, download it with the link:

```
singularity exec -B [comma delimited list of directories to bind] /path2compleasm/compleasm.sif compleasm [lineage]

# note you may want to include more information, and certainly choose your lineage.
```

Ok, now back to compleasm. Code will be similar to this, change all the variables for your assemnbly, lineage, etc:

```bash
singularity exec -B [comma delimited list of directories to bind] /path2compleasm/compleasm.sif compleasm run  --assembly_path ASSEMBLY --output_dir OUTPUT_DIRECTORY --threads NUM_THREADS --lineage LINEAGE_YOU_CHOSE --library_path /PATH2LIBRARY -m lite
```

### Assembling the PacBio reads

We are working with highly accurate HiFi data from PacBio. These are long, but accurate, the best of both worlds. We will be using `Hifiasm` to assemble these. There are a variety of programs for this, but this is fast and works well.

You need to use `-f0` in you hifiasm command so that we disable the memory intensive first step.

For Hifiasm, I want you to refer to the [Hifiasm GitHub page](https://github.com/chhylp123/hifiasm) to run this. Again, your data are in the directory `D.ananassae.HiFi.sub.25k.fq`, and they have `HiFi` in the name. Make sure that you use the `awk` command they mention to produce the genomic `fasta` file. 

**Question 11:** Paste in your hifiasm script.

Now, run `assemblathon_stats.pl` and `compleasm` on your long read assembly.

### Comparing the two assemblies  

We will use a file from each of our evaluation commands to compare these two assemblies. Check the output files of your slurm runs to figure this out.


Which assembly is better, and which do you have more confidence in. Why? 


For your assignment on Canvas, please send me your script and make it readable. You will also need to discuss the results from both genome assessment methods, and how the two assemblies compare.


