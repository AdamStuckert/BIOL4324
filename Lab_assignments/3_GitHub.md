### GitHub and version control

For today's lab, we will be learning GitHub and Git commands. You will be using Git/GitHub for work the rest of the semester, so it is important to learn to use them well now.

You will follow this [GitHub tutorial to start](https://docs.github.com/en/get-started/using-github/hello-world). Work through these steps to see how GitHub works on your web browser. I think actually seeing the process is helpful.

Next, you will read this second [GitHub tutorial](https://docs.github.com/en/get-started/using-git/about-git) to continue learning about Git, GitHub, and version control. You'll see why it is useful and helpful. Stop when you get to the examples with code, you will return here to work on a tutorial I provide you (based on the ones by GitHub, might as well use the training from the experts :smile: ).


## Create a new repository via command line

Let's create your own new repository! You will be doing this for the labs and the class final project, so time to learn it well. Bonus, you can use this for your research and programming so that you can track the changes you are making to your scripts and follow along with your efforts to improve functionality or fix bugs.

Before you ever do any git work on the cluster, first you must login...

```
# we are using the `gh` software, it lives here: /project/stuckert/bioinformatics/software/gh_2.98.0_linux_amd64/bin/gh
# you can either add it to path or call it manually every time or do something else to make it easier to call...

# first login
gh auth login
```


Ok, now you are logged in to GitHub on the HPC. Yay. Lets make your first repository. You can initiate (`init`) a repo from any directory, but best practice is to create the repo by name and automatically make the new directory.

```bash
git init YOUR-REPO-NAME

# now change into that repo--you figure this out.
```

Once you are in there, you can start creating your metadata, scripts, etc. Lets create a `README` file, using the same markdown formatting I use for these labs.

```bash
printf "## Lab 3" > README.md # printf usually prints to screen, but here we are using the ">" to redirect the output to a file
printf "This is my first GitHub Repo" >> README.md # using ">>" redirects the output and APPENDS to a file instead of CREATING or OVERWRITING a file
printf "Darwin would be so proud" >> README.md 

# you can also make these fancy brackets:
printf "```" >> README.md
printf "echo Hello world" >> README.md
printf "```" >> README.md
```

Now you've written your README, which is usually used to tell readers important information. Hence the read me in all caps. Now that we have done this, you are ready to submit them to the repo. You'll need to follow the two-step process of staging and pushing the specific files you've made.

```bash
# stage the changed files
git add README.md

# take a snapshot of the staging area (anything that's been added)
# note that this SHOULD be informative so that collaborators (both others and future you) can easily determine what you did
git commit -m "my first repo"

# now you have to push changes to github
# the first time you do this you will need to create the repo via command line
gh repo create YOUR-REPO-NAME --private --source=. --remote=origin --push # here i have added the --private flag to make it a private repo
```

OK, now you should check online and see your repo!

Once you've done this, you will want to practice that process a few times. The only difference is next time 

```
git push -u origin 
```


## Contribute to an existing repository

One of the main benefits of Git/GitHub is that you can work with collaborators in safe ways. And if your cat steps on your keyboard and deletes your entire script you can easily recover a previous version (this has happened to me). So, lets imagine for a minute you are collaborating with a classmate on a big project. You've divided the work so that one of you is working on some code to verify the quality of your readsets and another one of you is working on code to analyze gene expression. If you were both working on the same script at the same time it would be chaos, unless there was a way to preserve the initial script architecture while you each independently work. Luckily there is! Lets each work on branches on this GitHub repository.

First, you will `clone` the repository. When you clone a repository you are basically downloading an online repository onto your local machine.

Note: since today you are just making your accounts and I haven't added anyone as collaborators you will be modifying and *existing public repo* (it may look familiar). To do so when you don't have permissions, you can *fork* the repo when you clone it. If you ever find errors or want to add some functionality in people's code you can always take this approach.

```bash
# download/fork at the same time
gh repo fork https://github.com/AdamStuckert/BIOL4324.git --clone


# if you are working with a repo you have access to, or you just want to download and run software you can clone it like this:
# git clone https://github.com/AdamStuckert/BIOL4324.git

# change into the directory of the repository (you do this on your own for practice)

```

Next up, you will create your own branch of this repository. The branch is your personal way of working on the code without affecting other users or the class repo as a whole. I want you to create a branch with your last name, please. In the code below, substitute the string of text `YOUR-LAST-NAME` with, well, your last name.

```bash
# create a new branch to store any new changes
git branch YOUR-LAST-NAME

# switch to that branch (line of development)
git checkout YOUR-LAST-NAME
```

Next, I want you to add two new files. Lets make empty files, and name them something unique to you. You can make empty files, or refresh a file with an updated time stamp, with the command `touch`. Let's make files for 1) your favorite band/musical artist; and 2) your favorite meal. Use the format `band.` and `meal.` to start your files. So, for example if your favorite artist was Taylor Swift and your favorite meal was butter chicken you would create files called `band.Taylor-Swift` and `meal.butter-chicken`. Note that I removed spaces and replaced them with a "-", because spaces are special characters and NOT GOOD for your file names, etc.

```bash
# make your new files here
```

Ok, so you've made your changes and now you are ready to submit them to the repo. You'll need to follow the two-step process of staging and pushing the specific files you've made.

```bash
# stage the changed files
git add YOUR-FILES-HERE

# take a snapshot of the staging area (anything that's been added)
# note that this SHOULD be informative so that collaborators (both others and future you) can easily determine what you did
git commit -m "my fav band and meal"

# now you have to push changes to github
git push -u origin YOUR-LAST-NAME # reminder that your last name here is the branch name
```

Now you have committed your changes to your repository. At this point, you have a static image of what you've done. You can even login on the web and see your repository under the list of your repos. Finally, you will want to create a pull request in the repo. This is how you will receive a grade this week :frog: 

```
# push and do pull request (PR)
gh pr create --title "YOUR TITLE HERE" \
            --body "SOMETHING GOOD HERE" \
             --head YOUR-USERNAME:YOUR-LAST-NAME \
             --base main
```


