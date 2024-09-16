## Lab 5

Today we will learn the basics of using version control via git and GitHub

## Git

Git is a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency.

**Question 1:** Is git installed on the cluster?
**Question 2:** How do you tell if git is installed on the cluster?
**Question 3:** What is the absolute path of the git install you will be using?

## GitHub

GitHub is maintained software built on git to version control software. It is one of the most popular hosts for git repositories. We will use GitHub in class.

## Sign up for GitHub

Please [sign up for GitHub](https://docs.github.com/en/get-started/start-your-journey/creating-an-account-on-github) _using your cougarnet email address_. Please remember these credentials, you will need them!

**Question 4:** Post a screenshot of your newly minted GitHub account (if you already had one that uses your cougarnet email address for some other purpose that is obviously fine, but still post a screenshot).

## Configure your git

The next step is to configure your git on the cluster.

```
git config --global user.name "Your Name" # this is the name you will have publicly shown...
git config --global user.email yourname@cougarnet.uh.edu # change the email, obviously
git config --global core.editor nano # set nano as your text editor because vim is a hellscape
```

You will now initialize a local git repository in a new directory, called `git_training`. Create and enter the directory. Now, initiate the git repo in that directory:

```
git init
```

You should see something like this on the screen:

```
hint: Using 'master' as the name for the initial branch. This default branch name
hint: is subject to change. To configure the initial branch name to use in all
hint: of your new repositories, which will suppress this warning, call:
hint: 
hint:   git config --global init.defaultBranch <name>
hint: 
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint: 
hint:   git branch -m <name>
```

Rename your git branch to "main" using the above message.

**Question 5:** What line of code did you use to rename your git branch?

At this point you have an unnamed, local git repository. But it is empty. So time to populate this directory. Lets make a bunch of "files" to show how git works with files. Run this code:

```
touch samples.txt # a fake spreadsheet of samples
touch analysis.sh # a fake script to analyze sample data
mkdir raw_data
```

Now, lets make a bunch of fake fastq files and put them in our `raw_data` directory. Let's imagine we are working with microbial samples from different yeast strains to determine what genes contribute to different esters (a fruity flavor produced during fermentation that can vary in taste) in commercial beers. You have paired end reads, which often come in the format `*_R1.fastq.gz` and `*_R2.fastq.gz` for the forward and reverse reads. Write a conditional to populate 10 samples (`yeast_001` through `yeast_010`) with the format `yeast_001_R1.fastq.gz` and `yeast_001_R2.fastq.gz`. Quick note: large files often come compressed using either `zip` or `gzip`, these files typically have a suffix of `.zip` or `.gz` respectively.

**Question 6:** How did you create these yeast fastq files?

Raw data: NO TOUCHY. Make all of those files READ ONLY.

**Question 7:** What command did you use to make these fastq files read only?

Now you have populated the directory of your local git repo. Git will know these new files exist, but it will not be tracking these files for version control. To see this, check the repo status with `git status`. You'll see some key information about your files here.

Now we want to add files that we want to version control to our _staging environment._ This is just the group of files that git will prepare to "commit" to the hosted environment. We may want to limit the files that we actually stage and commit, because most hosts have file size limitations and read files are often BIG (>1 gb, even gzipped). You can add files to the staging environment using `git add FILENAME`. Add your spreadsheet and your analysis script.

**Question 8:** What command did you use to stage these files?

Now we will "commit" them and send them to the host. You must include a commit message when you commit. Run this code: `git commit -m "Initial commit"`

Now we will "push" this online to GitHub. 

>**Warning:** Never git add, commit, or push sensitive information, for example passwords or API keys, to a remote repository.

