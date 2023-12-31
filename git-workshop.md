# Introduction to Git

This worksheet provides a slightly simplified view of how git is used "in real life".

> **NB**: This is a brief overview of a very complex topic.
> For a further (and more in-depth) lesson, please look at the [Version Control with Git](https://swcarpentry.github.io/git-novice/) lesson on Software Carpentry.

## 0. Prerequisites

Before you start, you should have the following things set up:

1. Your computer needs to have `git` installed;
2. You need to be able to type commands into a terminal on your computer;
3. You need to have a suitable text editor (such as [`VSCode`](https://code.visualstudio.com)) installed; and
4. You need to have created a [GitHub](https://github.com) account for yourself

## 1. Create a workspace directory

> The first thing we're going to do is to create an empty folder to hold our code in one place.  This folder is going to become the git *workspace*.
> We'll then create a git repository within that folder.
> There are many different ways we can do this, but for now we're going to use the terminal.

Open the terminal application on your laptop and use the `cd` command to navigate to a sensible location for your work.
> If you're on a Windows computer you can shortcut this process by using File Explorer to go to a location and then right-clicking on a folder and selecting "Open Git bash here".
>
> **NB**: If you're struggling to navigate around your computer using the shell, either ask one of the demonstrators, or have a look at the [Software Carpentry Unix Shell refresher workshop](https://swcarpentry.github.io/shell-novice/index.html).

Create a new directory to hold our code using the bash terminal.
In this example, we're going to call it `random-reads`.
Once we've created the directory, we can change directory into it so that it becomes the current working directory:

~~~bash
mkdir random-reads
cd random-reads
~~~

Make sure that it is empty, by listing all of the directory contents:

~~~console
$ ls -a .
./    ../
~~~

> The `-a` argument to `ls` show hidden files and folders. Without this then the difrectory would not contain anything.

## 2. Initialise an empty local repository

Now that we've made a workspace directory, we need to initialise an empty git repository:

~~~console
$ git init
Initialized empty Git repository in /Users/apd500/Desktop/random-reads/.git/
~~~

> This command has created a new directory in the `random-reads` directory called `.git`.
> As it starts with a dot, it is hidden (at least in bash), so we ened to use the `-a` argument to `ls` to see it:

~~~console
$ ls -a
./    ../   .git/
~~~

Check that git thinks that the repository is OK:

~~~console
$ git status
On branch main

No commits yet

nothing to commit (create/copy files and use "git add" to track)
~~~

> The above message is saying that there is a repository here, but that it is empty.

## 3. Write some code

Now that we have a repository set up it is time to write some code.
This can be anything, but here we'll use a python script to generate random fastq data.

Open your text editor and write the python code into a new file.
Make sure you save the file into your workspace directory, calling it `random-reads`.

~~~python
#!/usr/bin/env python3

# Include the necessary modules:
import argparse
import logging
import sys
from random import choices

# Define a version for your code:
__version__ = "0.1.0"

# Define a function that returns an error with a sensible error message:
def error(msg, exit_code=1):
    logging.error(msg)
    sys.exit(exit_code)

if __name__ == "__main__":
    # Set the CLI. This is the primary way that your users will interact with your program.
    parser = argparse.ArgumentParser(description="Generate random FASTQ data")
    parser.add_argument("-V", "--version", action="version", version=f"%(prog)s {__version__}")
    parser.add_argument("-v", "--verbose", dest="verbosity", default="warning", choices=["error", "warning", "info", "debug"], help=f"Set logging level (default warning")
    parser.add_argument(dest="n_reads", type=int, metavar="N", help="Number of reads to generate")
    args = parser.parse_args()

    # Set up logging:
    logging.basicConfig(format="%(levelname)s: %(message)s", level=args.verbosity.upper())
    
    # Define the nucleotide set:
    nucleotides = "ACGT"
    logging.info(f"selecting from nucleotide character set: {nucleotides}")

    # Define the phred score set:
    phred = """!"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJ"""
    logging.info(f"selecting from PHRED character set: {phred}")

    # Generate the random reads:
    logging.info(f"generating {args.n_reads} reads")
    for read_i in range(args.n_reads):
        print(f"@READ_{read_i + 1}")
        print("".join(choices(nucleotides, k=10)))
        print("+")
        print("".join(choices(phred, k=10)))
~~~

Now that we've created a file, let's see what git tells us:

~~~console
$ git status
On branch main

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        random-reads

nothing added to commit but untracked files present (use "git add" to track)
~~~

> This is telling us that there is a new file in the workspace (`random-reads`) but that it has not yet been added to the index.

## 4. Make our first commit

We now need to commit this new file to the index, and commit it to our repository.
The first step is to add it to the index:

~~~console
$ git add random-reads
$ git status
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   random-reads
~~~

> This message tells us that the new file is now in the index ready to be committed.

We can now store this set of changes into a commit:

~~~console
$ git commit -m"Initial commit of the random-reads script"
[main (root-commit) 0200bb9] Initial commit of the random-reads script
 1 file changed, 45 insertions(+)
 create mode 100755 random-reads
~~~

Now, let's check the log:

~~~console
$ git log
commit 0200bb9c18a83f8f3d8341d472f58b4a72563fe4 (HEAD -> main)
Author: Alastair Droop <alastair.droop@york.ac.uk>
Date:   Thu Aug 31 14:43:35 2023 +0100

    Initial commit of the random-reads script
~~~

> Your log will look different.
> Your commit will have a different ID, and your username should be different to this one!
> This is the "standard loop" of git: generate content; fill up an index of modified files; commit those changes to the repository.

## 5. Let's test the repository

One use of git is as a method of backup.
Let's test this by deleting the `random-reads` file.

~~~console
$ rm random-reads
$ ls
~~~

Oops!  We've accidentially deleted our precious code!
Not to worry; we have a copy in the repository.
First, let's check what git thinks about this:

~~~console
$ git status
On branch main
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        deleted:    random-reads

no changes added to commit (use "git add" and/or "git commit -a")
~~~

> This message is telling us that a file that is recorded in the repository is missing.
> We have the option of recording its deletion in the index and committing that (we'd use `git rm random-reads`), but in this case the deletion was accidental so we'll simply get the file back from the latest commit.

We can retrieve the file from the latest commit:

~~~console
$ git checkout random-reads
Updated 1 path from the index
$ ls
random-reads*
~~~

> Great! We've got our code back again.
> This is a major benefit of `git`.

## 6. Adding a feature

Now, we need to update our random-reads code to allow us to alter the length of the reads returned.
We'll do this in a new branch in case our modifications break something.

First, let's create a new branch my checking out the latest commit into a new branch (called `specify-readlength`):

~~~console
$ git checkout -b specify-readlength
Switched to a new branch 'specify-readlength'
~~~

Now, when we check `git status` we will see we're on the new `specify-readlength` branch:

~~~console
$ git status
On branch specify-readlength
nothing to commit, working tree clean
~~~

Update the `random-reads` script to take an optional length argument by updating the following line numbers:

Update line 10:

~~~python
__version__ = "0.2.0"
~~~

Add a new line 24:

~~~python
    parser.add_argument("-l", "--length", dest="length", type=int, default = "100", metavar="N", help="Read length to yield")
~~~

Update line 43:

~~~python
        print("".join(choices(nucleotides, k=args.length)))
~~~

Update line 45:

~~~python
        print("".join(choices(phred, k=args.length)))
~~~

Now that we've made some modifictions, we make a new index and commit is as before:

~~~console
$ git add random-reads
$ git commit -m"Update random-reads to allow user-specified read length"
[specify-readlength a2fe48f] Update random-reads to allow user-specified read length
 1 file changed, 3 insertions(+), 2 deletions(-)
~~~

> Obviously, in the real world, features are a little more complex to implement, and would likely require multiple rounds of code editing and committing.

Although we've now edited `random-reads`, we can still get the unedited version by switching branches back to main:

~~~console
$ git switch main
Switched to branch 'main'
$ ./random-reads --version
random-reads 0.2.0
~~~

> As you can see, this version of `random-reads` is the old version!

Once we're happy with this new feature, we can merge the branch back into the main branch:

~~~console
$ git merge specify-readlength
Updating 0200bb9..a2fe48f
Fast-forward
 random-reads | 5 +++--
 1 file changed, 3 insertions(+), 2 deletions(-)
~~~

> OK, we've now merged the `specify-readlength` code back into our main branch. We're back in the main branch but our code has been updated.

## 7. Create a Github Repository

Now that we have an active local repository, we need to get it uploaded to a remote repository for safekeeping and security.
We will do this using [GitHub](https://www.github.com).

* Log into your Github account, and click on the "New" button (or go directly to [https://github.com/new](https://github.com/new)).
* You should be greeted by the "Create a new repository" page.
* Choose a sensible name for your repository (for example "random-reads").
* Add a brief description of the repository to help you remember what it contains.
* Select "Private" to make your repository private.
* For now, leave the "Add a README file" unticked, and select "None" for the `.gitignore` and license options.
* Click the "Create repository" button at the bottom.

Once you've done this, your new repository will be created.
Currently there is nothing in this repo.
SO, we need to push the contents of our local repository to this remote one.
For this to happen, we need to link this new remote repository the local repository.
We do this using the Github repository URL.

* Write down or copy the remote repository URL. It will be located in the "Quick Setup" section of the remote repository page.

Now that we have the remote repository created and have its URL, we can link it to the local repository.
We will give the remote repository a name to identify it.
By default, the main remote repository is called `origin`.
You could use whatever name you wanted though.

Link the local repository to the new remote repository:

~~~console
git remote add origin git@github.com:alastair-droop/random-reads.git
~~~

> **NB**: You will have a different URL, as this repository is private to me.

## 8. Push the local repository to GitHub

Now that the local repository is linked to a remote location, we can push it to the cloud:

~~~console
$ git push -u origin main
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 8 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (6/6), 1.35 KiB | 1.35 MiB/s, done.
Total 6 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), done.
To github.com:alastair-droop/random-reads.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
~~~

Congratulations! You've just pushed your local repository to GitHub!
If you go back to the GitHub and refresh the repository page, you should see that your code has been uploaded.

## 9. Adding a README file

Currently, the repository is not very well documented.
By convention, a file named `README` or `README.md` in the root of your workspace is taken to contain a description of the project.
If such a file exists, GitHub will display it on the main repository page.
Let's create such a file for our repository.

Go back to your text editor, and create a second file called `README.md` in the workspace folder.

Add the following text to the file:

```markdown
# Generate Random FASTQ Reads

The `random-reads` python script generates random nucleotide FASTQ data.

## Usage

~~~python
usage: random-reads [-h] [-V] [-v {error,warning,info,debug}] [-l N] N

Generate random FASTQ data

positional arguments:
  N                     Number of reads to generate

options:
  -h, --help            show this help message and exit
  -V, --version         show program's version number and exit
  -v {error,warning,info,debug}, --verbose {error,warning,info,debug}
                        Set logging level (default warning
  -l N, --length N      Read length to yield
~~~
```

As we've added a file, we need to make a new commit:

~~~console
$ git add README.md
$ git commit -m"Add README file"
[main d75b82e] Add README file
 1 file changed, 21 insertions(+)
 create mode 100644 README.md
~~~

We can now push this new commit to GitHub:

~~~console
$ git push
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 598 bytes | 598.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:alastair-droop/random-reads.git
   a2fe48f..d75b82e  main -> main
~~~

> We didn't need to specify a branch or remote name, as git will use the current branch and the only remote (as there is only one).

Go back to Github and refresh the page again.
Hopefully, you'll see the new README documentation appear.

## 10. Markdown

> The `README.md` file we've just created uses a format called Markdown.
> This is a very simple text file format that allows a computer to render our text nicely.
> You can dig into the format by reading the [documentation on GitHub](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/about-writing-and-formatting-on-github).
