# Intro to terminal and Unix

One of the most fundamental parts of bioinformatics is a basic familiarity with the command line. Many things are Linux and bash based, so these are good skills to develop. 

### Basic Unix rules

* Everything is case sensitive. BIOL4324 is not the same as biol4324.

* Unlike a GUI interface to view files on your personal computer, once you delete something it is **permanently gone**. You can easily delete an entire hard drive's worth of data by accident in seconds. I call this "pulling a Newhouse." A friend of mine did this to a hard drive in his lab, and there was much swearing.

* Related to the above, there is no undo button. If you write over a file or make a change to it, the deed is done and cannot be undone.

* The two points above are a very good reason why you should 1) be extremely careful when in terminal and 2) maintain good backup copies of all data outside of the environment you are working in! It is unreasonably easy to delete months and thousands of dollars worth of work.

* Spaces in file names should be avoided. Spaces are a special character (represented as `\s`) and thus make things wacky. 

* Your `$PATH` is the collection of locations where the computer looks for executables (programs).

* Folders and Files are all you have. If you want to access one of these, you need to tell the computer EXACTLY where it is. 
`/home/stuckert/biol4324/exam1_key.txt` will work (assuming you’ve spelled things correctly, and that the file really exists in that location), but `exam1_key.txt` may not.

* Related to the above, any file/location can be specified relative to your current working directory (folder). So for example if I'm in the `biol4324` folder from the above example, I can point to the exam 1 key with `exam1_key.txt`. If I am in the `stuckert/` directory, I can point to the same file with `biol4324/exam1_key.txt`. Alternatively, I can use the *absolute path*, which specifies the full path to a file. In this case, I can use `/home/stuckert/biol4324/exam1_key.txt` to point to this exam key *no matter what directory I am currently in*. Giving *relative paths* is similar to giving someone directions to the nearest Shipley's from *your current location.* *Absolute paths* are similar to giving someone lat/long coordinates.

* In this document, as in scripts generally, lines that begin with a `#` are comments and are not run.


##### Mac users
* Unix shell: Terminal, which is pre-installed on MacOSX. You can find it through the Finder by going to Go-> Utilities-> Terminal.

##### Windows users
* PuTTY: A terminal which you can install from [the putty website](https://www.putty.org/)
* MobaXterm: A terminal which you can install from [the MobaXterm website](https://mobaxterm.mobatek.net/)
* Built in Linux subsystem in Windows [Windows tech forum](https://techcommunity.microsoft.com/t5/windows-11/how-to-install-the-linux-windows-subsystem-in-windows-11/m-p/2701207)



**BEFORE WE DO THIS, A CRITICAL NOTE:**

If you are on a Mac and working through the terminal, or on a PC an working through MobaXterm, once you open the terminal you are **working in your computer's directories**. This means that you can accidentally delete/corrupt files on your personal computer, **including your operating system.** Please be extremely careful. I am not responsible for any personal sorrow you suffer in this manner.

#### Introduction to Opuntia, the UH training High Performance Cluster for this course.

In order to access this cluster, we have to 1) be on the UH network and 2) open a terminal window. 

You can be "on the UH network" via either the UH Secure wifi network or via the UH VPN. Please install the UH VPN so you may access the cluster when you are not on campus. [Instructions for the VPN can be found here.](https://uh.edu/infotech/services/computing/networks/vpn/)

In order to access the cluster from the terminal, you have to point your computer to the location you want to login to. The address for Opuntia is `opuntia.rcdc.uh.edu `, and to login you specify `cougarnet_accounte@opuntia.rcdc.uh.edu`  Your username is the same as your UH password. You use the ssh command to login. So, to access your account you would use:

```bash
ssh your_username@opuntia.rcdc.uh.edu 
```

Your terminal will then prompt you to provide your login information. Congrats! You are logged into the HPC and ready to SCIENCE!

### Basic commands
* `pwd` print working directory
* `ls` list directory contents
* `nano` open up a text editor
* `mkdir` make a directory (folder)
* `cd` Change directories
* `rm` delete a file
* `mv` move a file
* `cp` copy a file
* `man` gives information about a program. This is very useful.


## LAB ASSIGNMENT

### Practice some basic commands

This section is your `homework` for the day. You can (and should) get details on these commands using `man` as you go so that you can learn their uses. 

**Question 1:** What does this command do? 

```bash
pwd
```

Learn more using `man pwd`.

**Question 2:** The `ls` command lists files and folders in your present directory. First, go back one directory by typing `cd /project/stuckert` Try both `ls -lt` and `ls -lht`. What is the difference between these commands. You may need to use the `man ls` command to learn more.

**Question 3:** Create a new directory named `first_dir`. What command did you use to do this?

**Question 4:** Next we will move into your new directory. What command would you use to do this?

**Question 5:** Now we will create a new file using a text editor called `nano`. Typing the nano command opens up the editor. I want you to save the following as your file text. In order to do this copy it from this page and paste it in to `nano` using a right click. 

```
Hello world!
This is my first ever text file.
It has a number of words.
And a number of lines.
So many lines.
Like, a lot.
Way many.
Much lines.
Just killing space now.
I think we are done.
It is pretty neat.
But a bird in a tree is better
```

Once you have pasted this in, save it as `firstfile.txt`:

```bash
ctrl + x / cmnd + x # to close
y  # y to save edits, n to exit without saving edits
firstfile.txt # whatever you type here will be the file name
```

Copy and paste the output from `cat firstfile.txt`.

**Question 6:** What if you forgot what was in this file, or this is a file that someone else gave you and you had no idea what it was? There are a variety of ways to view files. Try `cat`, `head`, `tail`, `less`, and `more`. For each of these you have to specify the filename second (e.g., ` cat firstfile.txt`). What is the difference between `cat` and `head`?

**Question 7:** Make a copy of your file, then remove the original.

```bash
cp firstfile.txt 2ndfile.txt # make copy
ls -lht # view contents of your directory to see new file
rm firstfile.txt # remove original
ls -lht
```

Now try moving the file. What happens when you try this?

```bash
mv 2ndfile.txt firstfile.txt
```

**Question 8:** What is the difference between `cp` and `mv`?


**Programming note:** _ba dum tss!_

Much of the rest of today's material is heavily borrowed from Dr. Chris Balakrishnan's bioinformatics workshop.

First we will begin by learning to download some data from online. There are a variety of ways to do this, but for today we will use `wget`. This is a very useful and important aspect to bioinformatics. I have a compressed folder of files stored on the course's GitHub page. Download this:

```bash
wget https://github.com/AdamStuckert/Gen711/raw/master/Lab/Files/Lab2.zip
```

Next, view your current directory. Think back to last week to do this. Once you have done this, you should be able to see that you have downloaded a new file! You did this without a user interface or clicking anything! It should be called `Lab2.zip`. Because this is a compressed folder, if you view this with `less Lab2.zip` you should be able to see the contents of the folder. Before accessing any of them, you will have to uncompress this folder first:

```bash
unzip Lab2.zip
```

You can view the contents of your directory--it should now contain all of the same files that you saw when you used `less` to view the zipped directory. We will use these files to start with a bit of a review from last week. Let’s take a peek at the `darwin_chapter.txt` file using the ‘more’ command.

```bash
more darwin_chapter.txt
```

This is the last chapter from Charles Darwin’s last chapter of The Origin of Species’. Press space bar to continue scrolling through the file, type ‘q’ to quit. Now let us say that you want to only look at the first few lines of the file, just to verify it is what you think it is. You can do this by using the ‘head’ command. 

**Question 8: how many lines does head show?**

```bash
head darwin_chapter.txt
```

After viewing this, you may have been surprised to learn how many lines `head` automatically shows! Try this again with a new file:

```bash
head genes.txt
```

This is an interesting example, and highlights something about UNIX files. That is that until there is a return character (called that because you had to physically return the carriage of a typewriter), it is viewed as the same line of text even if it looks like multiple lines on your screen. Return characters are represented with `\n`.

To specify the number of lines viewed you can use the -n flag. Show the first 100 lines of the file.

```bash
head -n 100 genes.txt 
```

To view the last few lines of a file, use the tail command.

```bash
tail darwin_chapter.txt
```

Again, it might be easier to view what tail accomplishes with the `genes.txt` file.

When you are working with data, you will want to sprinkle in many, many little sanity checks. This is especially true if you are trying to manipulate data or files in some fashion. One way I do this frequently is by counting lines of a file (or counting lines in the output of a chunk of code). You can do this with the `wc` command. `wc` is extremely useful! Make sure you look at the various options it has with the `man` program. 

**Question 9:** How many lines are in this text file?

```bash
wc -l darwin_chapter.txt
```

How many characters are the file? 

```bash
wc -c darwin_chapter.txt
```

Darwin is credited with the theory of natural selection. You can see that Darwin used the word theory quite a bit:

```bash
grep "theory" darwin_chapter.txt
```

Now, you may be interested in counting exactly how many times he says this. Luckily, we have a tool called *piping*, which allows us to link commands together. This is one of the most useful tools in your toolbox. What piping does is say take the output from the first command and use this to run the second command. Since grep outputs a single line from the file each time it finds the requested string, we can then pipe this in to the `wc` command to count the number of times he says grandeur. 

**Question 10:** How many times does Darwin use the word "theory"?

```bash
grep "theory" darwin_chapter.txt | wc -l
```

**Question 11:** Why might counting the number of times Darwin used theory in this way be problematic and imprecise? This will take some thinking about things like return characters and the way `wc` works--so make sure you looked at the man page!


For the rest of this lab we will focus on some basic, but very useful information such as paths, the path, and variables. The examples I'm giving you are for a bash Linux system, but the ideas behind these are generally true for programming languages at large (although the syntax will be different).

### Paths

Any file/location can be specified in two ways:

1. Relative to your current working directory (folder). This is known as the *relative path* because it is *relative* to where you are. So for example if I'm in the biol4324 folder, I can point to the exam 1 key with exam1_key.txt. If I am in the `stuckert/` directory, I can point to the same file with `biol4324/exam1_key.txt`. Alternatively, I can use the absolute path, which specifies the full path to a file. In this case, I can use `/home/stuckert/biol4324/exam1_key.txt` to point to this exam key no matter what directory I am currently in.
2. You can specify the *full path* of the file/directory. This is known as the *absolute path*. Using absolute paths is long and arduous, but is generally a better practice than repeated use of relative paths. These can be tricky, especially if you are using several directories/files are a few subdirectories away. You can also "cheat" and use variables to help specify absolute paths (see below).

To demonstrate this, do the following (some of these are commands you will have to figure out based on what we've already learned):

```bash
cd # this will take you straight to your "home" folder (more on this below)
create a directory structure that includes "path/demo/yall"

```

**Question 12:** How did you create the directory structure "path/demo/yall"?
**Question 13:** What is the absolute path of the directory "yall"?
**Question 14:** If you are in the directory "path", how would you specify the relative path to the directory "yall"?

### Variables 

Variables are an incredibly useful programming tool. I personally use them a lot, possibly excessively. They are particularly useful for calling paths, scripts, or specifying output, etc. They are frequently used in scripts, and are frequently defined based on input when you call a script. Variables can be used to specify paths, integers, executables, etc. As mentioned above, you can use variables as shorthands for paths, which is quite convenient. You can set variables like this:

```bash
genome=/full/path/to/genome.fasta
txome=/full/path/to/txome.fasta
hifi=/path/to/hifi/executable/hifi
output_dir=/path/to/output/directory
```

In each of these cases, the variable is the single string before the `=` sign. To call those variables in your script or terminal, you would use this `$genome`. So you could print a variable to screen with either the `cat` or `echo` command (e.g., `echo $genome`).

You can also specify a variable using the output of a command. To do this, you use the command inside of `$()`. For example, to set the current directory as the variable `$DIR` you would use this command:

```bash
DIR=$(pwd)
```

This is convenient, because you can use it to call the absolute path with ease, eg: `${DIR}/raw_data/alskdjfaojf.1.fa` 

**Question 15:** Navigate back to the "path/demo" directory. Use the `pwd` command to create a variable called `CUR_DIR`. What command did you use to do this, how would you print this variable to screen, and what is the result of printing it to screen?

There are several variables that are built in within shell/bash. Prominent among these are yout `$PATH` and `$HOME`. Your `$PATH` includes all the directories in which the computer will automatically look for executables. So if you have a program executable called `supercoolsoftware` that is in your path, you can call it by just typing `supercoolsoftware`. Your `$HOME` is the location you are taken to when you login to the terminal. Note, you can change the directory you are automatically sent to on login, but your `$HOME` remains the same. 

**Question 16:** How would you print your `$PATH` to screen, and what is in your `$PATH`?

**Question 17:** We did quite a lot today! Try using `history` to see all the commands you typed today.

For more information on some of these commands, and to learn some new and useful ones, take a look at [this website](http://swcarpentry.github.io/2014-08-21-upenn/novice/ref/01-shell.html).
