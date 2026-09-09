## NCBI command line tools

Today we are beginning our work with *actual genomic data*. We are going to be using NCBI (the National Center for Biotechnology Information) to do this. NCBI is maintained by NIH, as part of its congressionally mandated mission to produce and disseminate scientific research. In addition to maintaining a variety of data sources, NCBI releases and maintains critical software. Among these are [BLAST](https://blast.ncbi.nlm.nih.gov/Blast.cgi) which is commonly used to identify genes or sequences. There is both a command line and web based version of this, and we used BLAST on the very first day of class to identify our genetic sequence of interest.

Now, the majority of data on NCBI is produced via federally funded research, so it is GOOD that anyone can access it. In fact, these databases are extraordinarily powerful, especially in combination with the command line tools NCBI maintains that we will work with today. You can easily downloaded hundreds of genome assemblies or thousands of gene sequences or hundreds of whole genome sequence datasets. In fact, my lab group routinely uses the NCBI databases to store our data for public (re)use, download genomes, and download data. Today we are going to run through some basics of this, by focusing on finding genomic data and genomic reads from a _Drosophila_.

We will start with the genomes of _Drosophila_. We can use the `datasets` command to find the genomes for this genus. The `summary` flag provides summaries, `genome` for genomes, and `taxon` for taxon. Pretty intuitive.

```
# datasets is in /project/stuckert/bioinformatics/software
datasets summary genome taxon "drosophila"
# all relevant software for today is in EITHER:
# /project/stuckert/bioinformatics/software
# OR
# /project/stuckert/bioinformatics/software/edirect
```

Hmmm. Seems like that's not enough input for datasets. Use the output to fix this, and take a look at the output.

Oof. That is...a lot, and hard to look through, right?? Ok, so lets say that we want to limit our search to chromosome level assemblies that are annotated. Use the help to figure out how to do this (`datasets summary genome taxon --help`). 

Now, once you have done this, it ... may still be difficult to read. So, lets use the `dataformat` software to format the data. We will output a tsv or tab separated file for ease of reading. here is your command, we are *piping* ( with the `|` character) the output of the previous command you used into `dataformat`.

```
datasets summary genome taxon [ ... your parameters ... ] --as-json-lines | dataformat tsv genome
```

That is still challenging to look at! It seems like a lot. Redirect the output into a file, so you can visualize it. How many total lines are there, representing how many total genomes? And once you have done that, look at just the header information (i.e., what metadata the table contains) using `head -n1` to get just the first line of your file. You can see there is a whole lot of metadata to parse through. In many cases, its better to just pull what you want, and you can see what columns of metadata you can pull with `dataformat tsv genome --help`.

What metadata do you think might be useful? 

As today's output, we will write a script that goes through some NCBI tools and creates a variety of outputs. Please make this with if statements. This is beneficial as you can move your script forward incrementally! So, you might make the above code to download the file an if statement by starting with:

```
if [ ! -f YOUR-FILE-HERE ]
```

which does the first condition only if that file *does not exist.* Include this in your Lab4 script.


Ok, now lets move on to downloading data. Lets choose a single genome to download, so we aren't spending a long time downloading data and hogging hard drive space. You can download the fasta file of a specific genome using the datasets command.

```
datasets download genome [ ... your parameters ... ]
```

For today, I want you to download the genome of a cool island endemic drosophilid, _Drosophila sechellia_. This species not only is an island endemic, but it lays eggs on the toxic noni fruit (*Morinda citrifolia*). Add an if/then statement to download *only the genome and protein* datasets from *D. sechellia* to your script.

Look at the output of that command, see what else you may need to do in order to actually access and use the data. Make sure you add this to your script too!

OK, now we have a genome! Now, lets grab some reads to download whole genome resequence (WGS) data that was produced on an Illumina machine. We can use NCBI's `esearch` to find this. `esearch` is great because you can add a bunch of boolean operators (and/ors) to parse out what you want programatically. You can search a variety of databases such as SRA (the short read archive), PubMed, protein, etc. Here we are only interested in data from the SRA (mostly genomic and transcriptomic) so we will specify this with the flag `-db sra`.  We can then pipe that into `efetch` to format the data from the runinfo:

```
esearch -db sra -query '"Drosophila sechellia"[Organism]' \
  | efetch -format runinfo 
  
  # remember you can redirect the output...

```

How many total runs do you have in the resulting table? What do the output data look like? 

As we are/will talk about in class, there are a variety of different sequencing platforms. The way sequencing is done has changed over time and is dependent, so we will probably want to account for this. Here, below I change the command to get **ONLY** samples that are whole genome sequencing done on an Illumina machine.


```
esearch -db sra -query '"Drosophila sechellia"[Organism] AND "strategy wgs"[Properties] AND "platform illumina"[Properties]' \
  | efetch -format runinfo
```

Now, we might want to parse out only certain "elements" of the data and parse them into a more human readable format, like tab-delimited. The previous is csv (comma seperated values), which is totally fine and easy for machines to parse, but hard to read. Here we are fetching the data in xml format and piping it into `xtract` to extract information. Here we are pulling the unique accession number, the instrument it was sequenced on, where it was prepped/submitted from, and location data IF there is any. Sometimes metadata is not very good or consistent. There is a very important push to make this much better and standardized for data reuse.

```
esearch -db sra -query '"Drosophila sechellia"[Organism] AND "strategy wgs"[Properties] AND "platform illumina"[Properties]' \
  | efetch -format xml \
  | xtract -pattern EXPERIMENT_PACKAGE \
    -element RUN@accession INSTRUMENT_MODEL Center_name RUN@published \
    -block SAMPLE_ATTRIBUTE -if TAG -equals "geo_loc_name" -element VALUE 

```

Ok, now we should have some basic information about EVERY whole genome resequence dataset from *D. sechellia* that is WGS data from an Illumina machine in a human readable format! Honestly, super cool. 

OK, now lets choose 5 accession numbers (first column) to download. How would you get either the first or last 5 lines? We only want the accession number for downloads, so parse the data programmatically so you ONLY get accession numbers and ONLY five of them. Hint, you can use `cut` which we've talked about or `awk` which is powerful primeval magic. 

Now we will download the data via the NCBI's SRA toolkit. This software is downloaded and maintained by the HPC IT folks (thanks Jeff!). Many Linux based HPC have software modules downloaded which you can find and load with the `module` command. So, for our purposes we are going to find it with:

```
module spider sra
```

which will then output the possible matches in their software modules. Now, we can load it with `module load [MODULE NAME]`. You can then `prefetch` the data from the accession and then download the actual fastq files from the sequencer with:

```bash
prefetch [ACCESSION]
fasterq-dump --split-files [ACCESSION] # note the split-files flag splits forward and reverse reads
```

Write a while loop to take your accession numbers from the file you made before and use those to `prefetch`/`fasterq-dump` the files. 

For lab 4 homework, drop the **absolute path** of your shell script for the entirety of today's lab into the homework on canvas.
