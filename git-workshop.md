# Introduction to Git

This worksheet provides a slightly simplified view of how git is used "in real life".

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

## Make our first commit

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
> 
> This is the "standard loop" of git: generate content; fill up an index of modified files; commit those changes to the repository.

## Let's test the repository

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

## Adding a feature

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
