### Transcriptome assembly

Today we will be doing a *de novo* transcriptome assembly. Similar to the other week, we are going to compare a couple of different metrics!

Background information: You suddenly found yourself as a newt (you got better), and you want to assemble your transcriptome! This seems like a totally fine thing to do, and there isn't any sort of ethical issues with assembling your own genomic data. 

Usually, you will get quite a bit of sequence data in order to do any transcriptomic type project (often hundreds of millions of reads). For the purposes of this lab, I've downloaded data from this paper by Glass et al. (https://www.biorxiv.org/content/10.1101/653238v1.abstract) and randomly subsampled down to a dataset that we can work with for the confines of our lab time. To do this I used the `seqtk sample` command. `seqtk` is an extraordinarily useful series of scripts for those of us working with genomic data. I can't recommend it enough.


For lab today we will be taking two different approaches to transcriptome assembly. In both we will be removing sequencing adaptors, error correcting reads, and then assembling a transcriptome. One will use a pretty common transcriptome assembler, Trinity (website: https://github.com/trinityrnaseq/trinityrnaseq/wiki). For the other assembly, we will use the SPAdes. This lab has a number of steps, and uses quite a few software packages.

## Software install

Tip: Make separate environments for different assemblers.

First, install Trinity via singularity. Remember that you will want to create the environment similar to how we did in the genome assembly lab.

```bash
singularity pull --dir /PATH/TO/WHERE/YOU/WANT/TO/DOWNLOAD/TRINITY trinity.sif docker://trinityrnaseq/trinityrnaseq:latest
singularity run -B $(pwd) trinity.sif

```

Second, install SPAdes. Do you still have your spades installed from the other week? Great, then you are done.

Third, install compleasm. Oh good, you are already done since you installed it the other week.

Fourth, install trimmomatic. This program does read trimming (barcodes, quality, etc).

```bash
wget https://github.com/usadellab/Trimmomatic/files/5854859/Trimmomatic-0.39.zip
unzip Trimmomatic-0.39.zip 
cd Trimmomatic-0.39/
# test:
java -jar trimmomatic-0.39.jar
```

 
Now we will download our newt data!

```bash
mkdir newt
cd newt
wget https://raw.githubusercontent.com/AdamStuckert/Gen711/master/Lab/Files/Taricha_granulosa_subsampled.1.fq
wget https://raw.githubusercontent.com/AdamStuckert/Gen711/master/Lab/Files/Taricha_granulosa_subsampled.2.fq
```

Verify that you have both datasets and that they are approximately the same size with `ls -lht`. Next, I'd like you to count the total number of reads for both the forward and reverse datasets. 

**Question 1:** how many reads are in each dataset, and what command did you use to count them? Hint: you can use the format of fastq files to your advantage here. Hint #2: We have not talked about `regular expression` yet, but it is a very powerful way to manipulate data for your purposes. The `^` symbol refers to the start of a line, so you can combine that with some search term to find the answer you want (specify it in quotes like this `"^"` with whatever search string you are using inside those same quotes.

Back to the regularly scheduled lab. First we will run Trimmomatic to trim off adaptor sequences and then run Trinity. Note: these need to be run on the compute node and NOT the headnode!

```bash
java -jar trimmomatic-0.39.jar PE -threads 5 -baseout Taricha_granulosa_subsampled.TRIM.fastq Taricha_granulosa_subsampled.1.fq Taricha_granulosa_subsampled.2.fq LEADING:3 TRAILING:3 ILLUMINACLIP:/project/stuckert/bioinformatics/lab7/barcodes.fa:2:30:10:8:TRUE MINLEN:25
```

Read through the documentation to see what files trimmomatic produces, and which ones we want to use for downstream processes.

Finally, we will run Trinity to assemble our transcriptome! Trinity runs in 3 parts (inchworm, chrysalis, butterfly), and will take a little bit of time to run. Again, doe this in a _script or via an interactive session on a compute node._ Finally, remember how you downloaded Trinity, and make sure you call it correctly with the image file. If you need help, you can look above/to the genome assembly tutorial for hints.

```bash
Trinity --SS_lib_type RF --no_version_check --seqType fq --output trinity/ --max_memory 15G --left $OUTPUT_FROM_TIMMOMATIC_LEFT  --right $OUTPUT_FROM_TIMMOMATIC_RIGHT  --CPU 5 --inchworm_cpu 5 --full_cleanup --no_bowtie
# obviously substitute your trimmomatic output for the variables.
```

Trinity will take a while to run! You can follow along with the output using the command `tail -f YOURSLURMOUTPUTFILE.TXT`. 
**Question 2:** What was your script? Paste in the text of the script that worked.

Good work! You've now assembled your first transcriptome! Lets take a look at the assembly quality using the two programs we talked about in class: assemblathon and compleasm. Do these things...

**Question 3:** What lineage did you choose to use? 

**Question 4:** What does assemblathon and compleasm tell us about our transcriptome?


**Question 5:** What is the proportion of each of the following in your assembly?
	1. Complete, single copy genes
	2. Complete, duplicated genes
	3. Fragmented genes
	4. Missing genes


Next, we will move on to another approach to transcriptome assembly using SPAdes. You have installed this program. Please note that you need to use the `rnaspades.py` command. Use the documentation to figure out what command you should use, and you will have to remember/find where you installed it previously. If you can't remember, looking at the man page for the command `find` may help you out.

**Question 6:** Paste in the text of your script here.

Run the same assembly metrics.

**Question 7:** Given this information, which assembly do you think is better and why?

