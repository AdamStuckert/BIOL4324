## Genome assembly

Today we will be using public data to assemble and assess the genome of the fruit fly *Drosophila ananassae*. These data come from [Tvedte et al. 2021](https://academic.oup.com/g3journal/article/11/6/jkab083/6188627?login=true#304735456). Please note that I have already downloaded these raw data for you, and additionally I have subsampled the data down substantially so that we will be able to run these assemblies in a short timeframe with limited resources. As a result of this, you should not expect to get particularly amazing genome assemblies!

As an aside, I suggest creating a new directory for every lab we do (e.g., something called "lab4" or "genome_assembly") so that you don't have a bajillion files in your home directory. That can get reaaaaaaally confusing, reallllllly quickly.


## Software installs

Wahoo! Today is the first day y'all will deal with software installs. Welcome to hell.


```bash
/project/stuckert/apps/Anaconda3-2023.07-2/condabin/conda init
```

At this point you should see results that look something like this:

```
[astucker@opuntia condabin]$ ./conda init
no change     /project/stuckert/apps/Anaconda3-2023.07-2/condabin/conda
no change     /project/stuckert/apps/Anaconda3-2023.07-2/bin/conda
no change     /project/stuckert/apps/Anaconda3-2023.07-2/bin/conda-env
no change     /project/stuckert/apps/Anaconda3-2023.07-2/bin/activate
no change     /project/stuckert/apps/Anaconda3-2023.07-2/bin/deactivate
no change     /project/stuckert/apps/Anaconda3-2023.07-2/etc/profile.d/conda.sh
no change     /project/stuckert/apps/Anaconda3-2023.07-2/etc/fish/conf.d/conda.fish
no change     /project/stuckert/apps/Anaconda3-2023.07-2/shell/condabin/Conda.psm1
no change     /project/stuckert/apps/Anaconda3-2023.07-2/shell/condabin/conda-hook.ps1
no change     /project/stuckert/apps/Anaconda3-2023.07-2/lib/python3.11/site-packages/xontrib/conda.xsh
no change     /project/stuckert/apps/Anaconda3-2023.07-2/etc/profile.d/conda.csh
modified      /project/stuckert/astucker/.bashrc

==> For changes to take effect, close and re-open your current shell. <==
```

This has updated our `bashrc` file for us. This is a "hidden" file that lives in our home directory that includes stored information about our setup, you can think of it as analogous to a user profile. Things found in here are functions, aliases, and critically, our `$PATH`. In order for changes to our bashrc to take effect we either need to close and reopen our shell session, or just source our `bashrc` files. Lets do the latter.

```
source ~/.bashrc
```

The changes to our bashrc should now have taken effect.

**Question 1:** Print your path to screen and paste in what is in your path.


Now we'll make a "conda environment" that will hold all of our installations, activate it, and install the programs we'll need for today's lab. Software installs are annoying. And often you end up with conflicting installs to run different tasks/pieces of software. So people have develope software management systems, things like anaconda, docker, and singularity which create environments or packages with which to install and run software without interfering with other software. These are really useful, particularly if you need versions of python 2 and python 3, for example.

The maintainers of the UH Carya cluster have already installed anaconda for us. We can create "environments" which are self-contained units of software. You can think of it like a shipping container on a big ship making a transatlantic crossing with goods (this is why one software management program is called "docker", because it ships containers). The only caveat is that we need to create environments and place them somewhere we have read/write permissions.

```bash
conda create --prefix Absolute/path/ofa/directory/in/your/file/structure/ENV_NAME
```




Sub out the path + ENV_NAME for your path and a reasonable environment name. You might choose to go with `genome_assembly` or similar. It will ask you if you are sure you want to modify this, yadda yadda, and the answer is yes. You can then load that environment with `conda activate Absolute/path/ofa/directory/in/your/file/structure/ENV_NAME`. This is analogous to stepping inside the shipping container. 

Narrator's note: writing all of that path every time you want to do something is probably annoying. You might consider using a variable.....

**Question 2:** What code would you use to set that path as a variable?
**Question 3:**  How would you use that variable in the above `conda activate` code?

You can "exit" the environment by using `conda deactivate`.

**Question 4:** Activate and deactivate the environment a couple of times. What changes about your prompt (everything before where your cursor is to type in the terminal) when you activate the conda environment?

Now, install some software. This may take a while...but luckily for you I tested which software packages take forever/are difficult to install and I made this easier/less time consuming for you so you don't spend the whole lab installing software.

```bash
conda activate ENV_NAME  
conda install -y -c bioconda -c conda-forge compleasm
```

Install `SPades`. This is an assembler for a variety of things, including Illumina data of small genomes.

```bash
wget https://github.com/ablab/spades/releases/download/v4.0.0/SPAdes-4.0.0-Linux.tar.gz
tar -xzf SPAdes-4.0.0-Linux.tar.gz # the downloaded file is a "tarball" which you have to extract (like a zipped folder)
cd SPAdes-4.0.0-Linux/bin/ # this is the directory your program lives in.

```

Test this install out immediately! Run `./spades.py`. If you get the help message from SPades then your install isn't immediately obviously broken. You can and SHOULD test software installs before using them on real data. Software often comes with toy datasets to use. For today, we will "test" it with our small datasets.

Install `Hifiasm`. This program is developed to assemble genomes from accurate long read data. Follow the installation instructions on the [Hifiasm GitHub page](https://github.com/chhylp123/hifiasm).

Test this install out immediately! Run `hifiasm`.

**Question 5:** Did your SPades and Hifiasm installs work?

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

**Question 6:** Can you guess what information `-k` gives this program? You can check the github page above to see if you are right.

- The `-o` option is specifying the output, as usual, but this time it is an output directory, so make sure you name it accordingly (and don't include a file extension).  

**Remember, you must run tasks on the compute node, not the login node. Please do not run any assemblies on the head/login node!**

**Question 7** What is the header you are using in your script to tell SLURM what resources it needs?


The data for this assembly are here: `/project/stuckert/bioinformatics/lab4`. These are the files with `Illumina` in the file name. Remember, Illumina data is _usually paired end_ so there are two sets of reads, a forward and reverse. 

spades.py -t 5 -m 55 --mp1-rf -k 95  --pe1-1 FORWARD_READS  --pe1-2 REVERSE_READS  -o illumina_assembly
  
  
#### Evaluating our short read assembly  

First we can run a perl script called `assemblathon_stats.pl`. This is a super fast script that can tell us about total bp, contiguity, and scaffold/contig statistics. I frequently use this in my own work.

```bash
/project/stuckert/software/assemblathon_stats.pl GENOME_ASSEMBLY # sub in your assembly name for this on the far right
```

**Question 8:** How many total base pairs is your assembly? What is the contig N50?

**Question 9:** Look up the *D. ananassee* true genome size online. How does our assembly compare to the estimated genome size from other sources?

Now we will assess genic content using `compleasm`. Compleasm is built on `BUSCO`, a [software program](https://busco.ezlab.org/) designed to look at genic content using conserved orthologs. The fundamental idea is that within a particular lineage, many genes are conserved between species. Genes can be gained and lost over evolutionary history, so more closely related species are more likely to have similar sets of genes. If a genome assembly has the vast majority of these genes, then it is likely to be more complete. If an assembly has a poor proportion of those genes, then it is likely not very complete. Choosing the best lineage is important for this type of analysis. You could choose "vertebrata" for a frog, but there are so many vertebrates with varied gene content that it limits the applicability to test genic completeness. This is slowly becoming less of an issue as more genomes are sequenced, annotated, and added to databases, and it is becoming easier to choose a more appropriate lineage. For more details, [read a recent BUSCO paper](https://academic.oup.com/mbe/article/38/10/4647/6329644).

Look up *Drosophila*'s classification, and then choose the appropriate lineage from [BUSCO's lineage datasets](https://busco-data.ezlab.org/v5/data/lineages/).

**Question 10:** What lineage did you choose?

You need to download this data. You can auto-download data with `compleasm`, but many HPCs restrict or don't have compute nodes with internet access to search/download files via SLURM scripts. So lets do that outside of a job (this is a quick download so it is fine to run on the head node). You will need to activate your conda environment where you installed compleasm.

```
conda activate ENV
compleasm download LINEAGE_YOU_CHOSE --library_path . # here the "." refers to the current directory. so it will download the dataset to where you currently are. This is common syntax. You may want to download somewhere else though, which you can specify.
```

Now you can populate a shell script for submission to SLURM. Do that please, and run it. There are two tricks you need to know. First is that you need to activate your conda environment for running this. You can do it either before submitting the script **or** in the script I advocate for doing it within your script so that you aren't forgetting to activate it prior to submission. The second trick is that you need to source the conda info within your script. This is kinda annoying, but not too terrible. 

For me, this would look like this:

```bash
. /project/stuckert/apps/Anaconda3-2023.07-2/etc/profile.d/conda.sh # the period is required
conda activate $CONDA_ENVS/genome_assembly # I have saved the directory of my conda environments as a global variable cuz that is just too damn much to type all the time
```

If you do not know where your `conda.sh` is located, you can see via your bashrc file: `cat ~/.bashrc`.

Ok, now back to compleasm. Code will be similar to this, change all the variables for your assemnbly, lineage, etc:

```bash
compleasm run --assembly_path $ASSEMBLY \
--output_dir $BUSCO_OUT --threads 5 --lineage $LINEAGE \
--library_path $LIBRARY # you may need a "-m lite" flag" --even still it may not work if the assembly is too poor
```

**Question 11:** Paste in your slurm script. 

### Assembling the PacBio reads

We are working with highly accurate HiFi data from PacBio. These are long, but accurate, the best of both worlds. We will be using `Hifiasm` to assemble these. There are a variety of programs for this, but this is fast and works well.

You need to use `-f0` in you hifiasm command so that we disable the memory intensive first step.

For Hifiasm, I want you to refer to the [Hifiasm GitHub page](https://github.com/chhylp123/hifiasm) to run this. Again, your data are in the directory `D.ananassae.HiFi.sub.25k.fq`, and they have `HiFi` in the name. Make sure that you use the `awk` command they mention to produce the genomic `fasta` file. 

**Question 12:** Paste in your hifiasm script.

Now, run `assemblathon_stats.pl` and `compleasm` on your long read assembly.

### Comparing the two assemblies  

We will use a file from each of our evaluation commands to compare these two assemblies. Check the output files of your slurm runs to figure this out.


**Question 13:** Which assembly is better, and which do you have more confidence in. Why? Dicuss the results from both genome assessment methods.
