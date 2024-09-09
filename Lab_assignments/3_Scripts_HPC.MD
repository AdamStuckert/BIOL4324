## Lab 3

Today we will learn the basics of scripts, as well as how to properly use an HPC. 

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
drwxr-xr-x 8 adamstuckert adamstuckert 4096 Dec 20  2022 sshfs-win
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

It shows the job ID, the partition (in our case we have a dedicated partition for the course), name of the script running, etc. TIME refers to total running time. While the script runs it produces output to a logfile. By default, these are named "slurm-batchjob#.out". So the 

Many bioinformatic processes will take a lot of resources to properly run. Among the things you may want to modify are the memory allotment, number of threads requested, and the runtime. These are probably the most useful for the purposes of our class. But there are also ways to request multiple tasks that get spread out over several compute nodes. We have limited resources for the class that we need to share and will mostly be working with pared down datasets, so this is not much of an issue for us, but may be for you if you do bigger analyses. All of these are modified via `#SBATCH` at the very beginning of your script.

```bash
#!/bin/bash
#SBATCH t 1-00:01 # total requested time the job may run in D-HH:MM
#SBATCH --cpus-per-task=4 # requests 4 threads/cpus per task. You can also set ntasks, but by default it is 1
#SBATCH --mem=10Gb # specify the amount of memory to request for the job
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

Last thing, it is very important to exit your interactive session so that other people can use the resource by typing the `exit` command.
