

Today we will learn the basics of scripts, how to use an HPC, and conditionals/variables.

## Scripts

Scripts are the foundation of bioinformatics. They provide automation, flexibility, and speed in analyses--particularly tasks that are repeated over days, months, or years. Scripts can run a single command, or a hundred million (don't do this, that makes debugging impossible and you will hate your past self). Although in some cases it may be possible to run commands one by one, and in fact this can be really helpful for assembling pipelines/troubleshooting, it is not fun or feasible in most cases. 

Scripts add a lot of flexibility and speed. In Linux environments we use shell scripts, which typically end in `.sh` (but do not have to.

All scripts must start with a `shebang` which is a `#!`. This is a weird term. Here is what Wikipedia says about it:

> The name shebang for the distinctive two characters may have come from an inexact contraction of SHArp bang or haSH bang, referring to the two typical Unix names for them. Another theory on the sh in shebang is that it is from the default shell sh, usually invoked with shebang.

Anyway, the shebang statement must occur starting at the first character of the first line in the script. This statement tells the computer which interpreter to use. You'll notice that it looks like a following the shebang is the location of the interpreter. You may end up using different interpreters for python, r, c, bash, etc. We will use bash in class, though you are welcome to use other languages for your final project if you feel comfortable doing so.

Assuming that your interpreter is in your `/bin/` (which is a good assumption, generally), your shebang statement would look like this:

```bash
#!/bin/bash
```

By Linux law, we are contractually obligated to create a Hello World script as our first script. Open up your nano text editor and write the following in the file (hint, you may need to go back to lab 1 for help with this):

```bash
#!/bin/bash

echo Hello World!
```

Save this as `hello_world.sh`.

**Question 1:** What command did you use to open the text editor and how did you save it?

You've now saved your first script! 

Let's take a look at this script with the `ls -l` command (the `-l` flag is for "long"). It should look something like this:

```
drwxr-xr-x 8 adamstuckert adamstuckert 4096 Aug 20  2024 sshfs-win
```

The first group of characters and hyphens are the permissions. The first character is either a `d` for directory or `-` for file. The next 9 characters are in  groups of 3 (like codons!), and represent wether the owner, group, and everyone have read (r), write (w), and executable (x) privileges in that order. If those are represented by a `-` then that person/group does not have permission for that task. The first number represents the number of links or files in a directory. 

After this there are two string characters. These represent the owner of the file/directory, and finally what group they are in. 

The two right most fields are the name (rightmost field) and the date this was last modified (second field from the right).

**Question 2:** What group are you in?

I've mentioned executables several times now. Executables are scripts that you can run, and often live in your `$PATH`. 

**Question 3:** What is your path (`$PATH`)?

You can change the executable status of your script with the `chmod` command. To set it to executable for yourself you would use the command for user (`u`) and add executable (`+x`) like this:

```bash
chmod u+x SCRIPT_NAME
```

If you run `ls -l` again you should see that permissions changed. Depending on the terminal you are using, it may now show your script as green, this is a common convention for executables. You can now run your script with either `sh hello_world.sh` or `./hello_world.sh`. The `./` signifies your current directory, so you may need to change the path you give it if you are in a differeent directory.

**Question 4:** Paste a screenshot of calling your script and the output.

You've now made and run your first script! You can do all the same things within a script that we've already talked about: conditionals like if/else statements, variables, etc. 


## Designing functional scripts

Often you will want to do a number of tasks in serial in order to analyze data. This might include downloading data, analyzing quality, trimming adapters, and then mapping reads to an assembly. You may have 100 read sets, and you don't want to manually do those 4 tasks in serial by manually writing it all out. We can use a variety of programmatic magic to make this happen. For today we will start with some examples of conditionals, put them into a script, and then build on this with more complex scripts in future weeks. The examples I'm giving you are for a bash Linux system, but the ideas behind these are generally true for programming languages at large (although the syntax will be different).

## Conditionals


Conditionals are a programming staple. There are three main types that we will cover in today's class:

1. For loops
2. While loops
3. If then statements

Each of these have their own unique uses in programming, and programming languages all have their own For loops are meant to run through every item in a list and do a series of tasks. While loops are similar to for loops, but they run only _while a condition is true_. If then statements are basically a decision flow chart, but in code. We will be talking about each of these in class, and here we will go through examples of them. Note, these are not the only conditionals we will cover this semester, we will cover others later.

**Note: for the rest of this lab please add these statements to your main script for the day. At the end of the class I expect you to be able to run them all from a single script. We are building on our work as we go!**

## If then statements.


If then statements are extremely useful for parsing data and enacting different actions, regardless of the programming language. You can use them to do a task if a value is less than 5, enable an action if a file does not exist, etc. The most basic way to use these is to enact a particular command only in one condition. For example, you can test if a file exists:

```shell
touch testfile.txt # the touch command creates an empty file
if [ -f testfile.txt ]
  then
    echo "testfile.txt exists."
fi
```

A more complicated version of this is to do a command if a condition is met, and a followup command if that condition is not met.

```shell
rm testfile.txt # reminder, rm removes files. this is permanent, so just be careful about that...
if [ -f testfile.txt ]
  then
    echo "testfile.txt exists."
else
    echo "testfile.txt does not exist."
fi
```

There is also a third variation of the if/then statement: the elif statement. These are where you can really add in a lot of conditionals. As many as you want! Or at least as many as you can keep track of. The basic structure is this:

```shell
if test
  then
    perform actions
elif test
  then
    perform elif action(s)
else
    perform else action(s)
fi
```

Design code to run through three conditions:

1. If a number is greater than 10 ( `-gt 10` ) print that the number is greater than 10
2. If a number is less than 10 ( `-lt 10` ) print that the number is less than 10
3. If a number is equal to 10 ( `-eq 10` ) print that the number is equal to 10 Use a variable (e.g., `$num` or similar) that you specify, per last week.

Make sure to test it under each condition in your script!

## For loops


For loops execute a series of commands repeatedly. They run through iterations of a variable/file and run the same code over and over, generally until it has gone through all possible iterations. At its simplest, a for loop looks like this:

```shell
for i in 2 4 6 8 10
do
echo "$i coog"
done
```

Run the above, and consider the output:

* What is the output of this command?
* What is `i` in this command? What does `i` represent the 4th time this loop is run?

A fun trick, you can set something to run through a range of numbers using the format `{START..END..INCREMENT}`. So you could run numbers 1 to 10 by 2.

Write a for loop to print every 5th number from 5 to 100 (eg, it would go 5,10,15 and so on until 100). Put this in your script for today's lab.

Last week we ran commands within a single line of code to set the output as a variable. We can do the same thing within a for loop. I will often use this to run a series of commands on all the raw data I have for a particular project. For example, if I want to run through all the fasta files in a I can use the `ls` function to identify all the fasta files in a directory and set them as my variable. Assuming that I have all my data in a directory called `raw_data` and they all end in `.fasta` I can use the `ls` command and a wildcard character `*` to find all the fasta files like this: `ls raw_data/*.fasta`. This finds all the files that are in the `raw_data` directory whose file names end in `.fasta`. You can do this with any file type (e.g., csv, tsv, txt, pdf, etc). You can also do this to match other partial strings, for example if all the reads for a project started with `BIOL434` you could do `ls raw_data/BIOL4324*`, assuming that there were no other files in that directory that started with that name. If there were, you could find all the relevant fastq files, via: `ls raw_data/BIOL4324*.fastq`.

Write a for loop that uses `ls` to print all the fasta file names in the directory `/project/stuckert/bioinformatics/lab2/fasta_files` to screen using this format:

`Filename: shasta_i.fasta` where every fasta file prints `Filename:`  then the filename, all on its own line.

## While loops


While loops are meant to run repeatedly _while a condition is true_. The simplest form of this is to run while a number is less than X. This can be useful if you want to run a set number of iterations of code. While loops follow the same format as for loops, but switch out `while` for `for`. Syntax thus looks like this:

```shell
while CONDITION
do
DO THE THING
done
```

If, for example, you wanted to run from 1 to 10, you could make a "counter" variable, and run a while loop as long as that `$counter` is less than or equal to 10.

```shell
counter=1
while [ $counter -le 10 ]
do
echo $counter
((counter++))
done
```

The line `((counter++))` takes the current value of `counter` and adds 1 to it. This is a useful tool.

We can also use the `while` command to read through a file and repeat commands iteratively. The while command and condition would thus look like `while read line`; and you have to direct the input into the while loop from file using a directional carrot (`done < input.txt`).

Say for example you have a lot of fasta files in a directory, but after doing quality control on them, some of the files do not pass QC. You could iterate through fasta files like we did above, but you don't want to waste computational time running poor sequence data through your pipeline (or include it in your downstream analyses). So you've populated a text file with all the "good" files to use called `samples_pass_QC.txt`.

So, as an example, it would look similar to this:

```shell
while read line
do
your fancy pants commands on $line here
done < input.txt
```

Similar to how we can `cat` or `echo` or send the output of whatever command to a file with the `>` symbol, we can do the opposite and send a file to a while loop with `<`.

Create a while loop that reads through the file `/project/stuckert/bioinformatics/lab2/samples_pass_QC.txt` and prints "Running pipeline on sample (good sample here)" in which "(good sample here)" should be replaced with the line from the text file. Paste in your code.

## Nesting loops


Loops can be nested within each other like a set of Russian dolls. You can have a series of for loops within each other, you can intermingle for loops and if/else statements. etc.

I used the following command to generate all the shasta fasta files:

```shell
for file in i ii iii iv v vi vii
do
touch shasta_${file}.fasta
done
```

BUT, we are missing Louie! We need to add the itty bitty kitty to the itty bitty kitty committee.

Use my for loop above and a nested conditional to make files that start with "shasta" for each shasta, but do not start with "shasta" for Louie. Hint: start a command with `for file in i ii iii iv v vi vii louie` and go from there. Paste in your code.

Next time we will get into scripts and using the HPC.




## Using the HPC

High performance computing clusters require some additional education to use at all, much less effectively. You've all already taken the first step, which is to login to the cluster via the internet. The second thing you need to know, after how to login, is a bit about the infrastructure of the cluster.  Clusters have a dedicated set of "login nodes" which everyone gets directed to upon logging in. Like your personal computer, this login node has a set number of threads, and a set amount of memory that is available for working processes. Unlike your laptop, this must be shared among ALL the users of the HPC, and some clusters may have hundreds of active users.

![HPC architecture](https://tacc-reproducible-intro-hpc.readthedocs.io/en/latest/_images/hpc_schematic.png)

Because of this, you need to be careful what tasks you are doing on the head node. Be a good citizen of the cluster! They are allowing us to use this for teaching, and we don't want to lose this privilege. The only tasks you should be running on the head node are things like text editing, viewing files, and occasionally downloading data. Anything that takes 1) a lot of memory or 2) more than a minute or two should be a job run on a compute node.

On that note, it is time to learn about job scheduling software!

### SLURM

SLURM is one of several job scheduling software programs which are in charge of managing who gets access to compute nodes and resources based on set criteria. The eytmology of SLURM is lost in the annals of time.

**Question 5:** What do you think SLURM means?

SLURM takes specialized scripts as input and will allocate needed resources based on the script. To submit this task to the scheduler, we use the `sbatch` command. This creates a *job* which will run the *script* when dispatched to a compute node which the queuing system has identified as being available to perform the work. So you could submit your Hello world script like this:

```bash
sbatch hello_world.sh
```

You will get a message that looks like this if it was successfully submitted:

```
Submitted batch job 1284611
```

These jobs may run a long time, so you can check the status of these jobs with the `squeue` command. I find this command difficult to type, there are an entirely unreasonable number of vowels strung together. It will be most helpful to specify your username with the `-u cougarnet_name` to filter results down to only your jobs. Unless you are nosey and you wanna see what that overachiever 2 seats down is doing. The output will be like this:

```
[astucker@opuntia ~]$ squeue -u astucker
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
           1284612     teach hello_wo astucker  R       0:04      1 compute-0-30
```

It shows the job ID, the partition (in our case we have a dedicated partition for the course), name of the script running, etc. TIME refers to total running time. While the script runs it produces output to a logfile. By default, these are named "slurm-batchjob#.out". These log files are extremely useful for troubleshooting your code. You may want to rename them something more informative and easier to find (via `#SBATCH` commands like below).

Many bioinformatic processes will take a lot of resources to properly run. Among the things you may want to modify are the memory allotment, number of threads requested, and the runtime. These are probably the most useful for the purposes of our class. But there are also ways to request multiple tasks that get spread out over several compute nodes. We have limited resources for the class that we need to share and will mostly be working with pared down datasets, so this is not much of an issue for us, but may be for you if you do bigger analyses. All of these are modified via `#SBATCH` at the very beginning of your script.

```bash
#!/bin/bash
#SBATCH -t 1-00:01 # total requested time the job may run in D-HH:MM
#SBATCH --cpus-per-task=4 # requests 4 threads/cpus per task. You can also set ntasks, but by default it is 1
#SBATCH --mem=10Gb # specify the amount of memory to request for the job
#SBATCH --output=%x.%j.out # specify the output file name. You can specify it however you want, but this is a nice automatic way to specify output files. In this case %x = job name; %j = job id
```

Note: if you run over time requested or the memory requested you will get an error message similar to this:

```
slurmstepd: error: *** JOB ID# ON PARTITION CANCELLED AT TIME DUE TO TIME LIMIT ***
```

The error may be for several possible reasons, for us most likely due to memory or time. In general, you should aim to request only *slightly* more resources than are needed for your job. In practice, this can be hard to determine, particularly the first time you are attempting a task. 

You may also cancel jobs with the `scancel JOBID` command (sub your job id number for JOBID). Save the following script to a file.

```
#!/bin/bash
#SBATCH --cpus-per-task=1

sleep 10m
```

**Question 6:** Submit this script as a job. View your running jobs with one of the commands you've learned today, and cancel it with scancel. Screenshot your terminal to show me that you did this and paste it in.

Available resources are typically determined by the physical setup of the cluster (ie, how many compute nodes there are, how many cpus each has, how much memory, etc). For this class we have been given a teaching allocation, so we have very limited resources. Since we are all in lab together running things at nearly the same time, I am going to cap our resources. **Unless I tell you otherwise, please use no more than 5 threads (`--cpus-per-task=5`) and 15 Gb of memory (`--mem=15Gb`).**

**Question 7:** Please write the following: I, (your name here), do solemnly swear to use no more than 5 cpus and 15 Gb of memory per job I submit.

## Interactive jobs

Unfortunately, it is fairly common to have your jobs wait in the queue to be allocated resources to run. Depending on the cluster, number of active users, the infrastructure, and the resources you request, this may be from minutes to days. We are not likely to be in this position as they have devoted several compute nodes to the class for us so we will get priority on those nodes. But, if you are running a new script or building a script and you aren't sure that it will run through, it can be really annoying (and a huge timesink) to submit a job, wait a day for resources so your job can start, and then have your job immediately error out. Enter: interactive jobs!

```
srun --pty bash
```

This will put you in the queue, and once you are allocated resources you will be given a prompt like normal. But instead of being on the login node, you are on the compute node. This allows you to test your code, and make edits as you go. It can be quite useful. You can also provide the same resource allocation requests within `srun` that you do within `sbatch` or your shell script. A good practice is to use a toy dataset that is significanlty pared down from the full dataset to test your code. You can then request minimal resources and run in an interactive session.

**Question 8:** Take a look at the prompt on the login node. Now run an interactive job and look at the prompt. Is there anything different?

Just like on the login node or on a compute node, you can execute scripts/commands. But when you are in an interactive session it is easier to piece together pieces of a pipeline or troubleshoot. To demonstrate that you can execute scripts/commands in the same manner, execute this shell script:

```bash
/project/stuckert/bioinformatics/lab3/while_read.sh
```

**Question 9:** What is the output of this?

Last thing to note for interactive sessions, it is very important to exit your interactive session so that other people can use the resource by typing the `exit` command.

Now, I want you to put everything you have learned together and submit a job to SLURM which contains a conditional statement. Let's go! 

**Question 10:** Write and submit a SLURM script which uses an if/else statement to download two files (`Taricha_granulosa_subsampled.1.fq` and `Taricha_granulosa_subsampled.2.fq`) into a directory called `newt` only if they have not already been downloaded. This if/else statement should also tell you which file you are on and if it is already present/if it is downloading the new file. This job should ask for 10 gigs of memory, 5 hours of run time, and write to an output file named "She_turned_me_into_a_newt"). Use the below code to download the files:

```
wget https://raw.githubusercontent.com/AdamStuckert/Gen711/master/Lab/Files/Taricha_granulosa_subsampled.1.fq
wget https://raw.githubusercontent.com/AdamStuckert/Gen711/master/Lab/Files/Taricha_granulosa_subsampled.2.fq
```

Paste in your script.

Look through those two files, to figure out what they are and what information they might contain.
