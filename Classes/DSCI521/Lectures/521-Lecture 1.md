[[521-Lab 0a]]
[[521-Lab 0b]]
[[521-Lab 1]]

---
title: Introduction to MDS software and Bash
---

## Learning outcomes

1. Recognize the directory hierarchy as it is commonly represented in diagrams, paths, and file explorer software.
2. Distinguish common operators and representations of the different filesystem elements typically used in Bash.
3. Explore the filesystem using Bash commands as `ls`, `pwd` and `cd`.
4. Translate an absolute path into a relative path and vice versa.
5. Use command-line arguments to produce alternative outputs of commands.
6. Create, edit, move, and delete files and folders using the command line and VS Code.

**Platform in focus** BASH

## Introduction

Welcome to Computer Platforms in Data Science!

This week we are going to learn about exploring
the filesystem, using the command line, and some other useful tools.

## Instructions for Accessing iClicker

Please follow these instructions for accessing iClicker: [iClicker Cloud Student Guide](https://lthub.ubc.ca/guides/iclicker-cloud-student-guide/)

:::{.activity}
## Lecture 1 Activity 1

What is a Data Scientist?

A. Someone whose job is to extract value from data through reproducible and auditable processes.

B. Someone whose job is to apply machine learning algorithms.

C. Someone whose job is to analyze large volumes of often unstructured data to produce meaningful insights. <https://www.springboard.com/blog/data-science/data-science-definition/>

D. All of the above.

E. Someone whose job is to get well paid.
:::


Most of the tools we are going to learn during DSCI 521 are tools that programmers used.
Programming involves interacting with the system using commands

## The MDS software stack

The following are the core programs that you will be using throughout the MDS program.

- **Bash**:
  Helps you navigate filesystem, run programs,
  and automate tasks on your computer.
- **Git and GitHub**:
  Used to track changes in your code
  and collaborate with others on projects.
- **Python and R**:
  Popular programming languages for
  analyzing data, creating visualizations, performing statistical analysis,
  and building machine learning models.
- **Visual Studio Code, JupyterLab, RStudio, and Positron**:
  Tools that provide an integraded development environemnt (IDE)
  that make writing and running code easier.
  They also help create interactive notebooks and reports.

These programs also have active development communities and are open source,
which means that anyone can read the source code and contribute to these projects.
Open source programs have many benefits,
such as

- Being able to use the software after leaving school
  without paying exorbitant amounts of money.
- Having access to the source code can help you learn and fix bugs
  (even closed-source commercial software has bugs,
  but they are just harder to see and fix).
- Open-source software is customizable and flexible.
- It is becoming the standard in many tech companies.
- Being a part of an awesome and active community!


:::{.activity}

## Lecture 1 Activity 2

In what scenario is the command line preferable over JupyterLab?

A. When scripting and automating tasks due to its quick execution of commands and ease of integration into workflows.

B. For interactive data exploration due to its rich user interface.

C. For beginners because of its simpler interface.

D. For large-scale data processing due to its efficiency in handling data.

E. Both are equally effective.

:::


## Introduction to computing

Before we get into the different applications,
let's talk about computing.
Essentially, computing is about humans communicating
with machines to modulate flows of current in hardware.
Early examples of human computer communication were quite primitive
and included actually disconnecting a wire
and connecting it again in a different spot.

Luckily,
we are not doing this anymore;
instead we have graphical user interface with menus and buttons,
which is what you are commonly using on your laptop.

These graphical interface can be thought of as a layer (or *shell*)
around the internal components of your operating system.
Shells exist as an intermediate
that both makes it easy for us to express our thoughts,
and for computers to interpret them.


### Text-based communication

We will learn how to communicate to a computer via a text-based shell,
rather than a graphical one.

Using a text-based shell might at first seems counter-intuitive,
since the reason for creating a shell in the first place
was to facilitate user interaction with the computer.
So now that we have these easy to use graphical user interfaces (GUIs),
why would anyone in their right mind go back to using a text based interface?

Well,
it’s partly a misconception,
GUIs are nice when you are new to something,
but text based interfaces can be faster
and easier to use when you know what you are doing,
and provide a lot more flexibility.

We can compare it to learning a language,
in the beginning it’s nice to look things up in a dictionary
(or a menu on the computer),
but once I know what I want to say,
it is just easer to say or type it directly,
instead of looking in submenues.

And by extension,
it would be even faster to speak or even just think of what you want to do
and have it executed by the computer,
this is what speech- and brain-computer interfaces are concerned with.

### Bash

`bash` is the most commonly used text shell.
You have it installed on your computer by default if you are using Mac or Linux machine,
and if you are on a Windows machine you downloaded bash as part of the setup instructions.
For Macs the default shell is `zsh`,
but we had you change it to bash in our setup instructions.
Sometimes we might use "prompt",
“command line”,
or "terminal",
which for the purposes of this lecture,
refers to the same thing.

The abbreviation BASH stands for Bourne Again SHell.
Other shells existed before Bash,
and one of the most successful early shells
was invented by Stephen Bourne at Bell Labs in 1977,
which he called the Bourne Shell.
In 1989,
the Free Software Foundation improved the Bourne Shell and as a pun,
named it the Bourne Again Shell
to symbolize it was now "reborn" with new features.

Text-based shells are also called command-line interfaces (CLI).
The heart of every CLI is a read-evaluate-print loop (REPL).
When we type a command and press <kbd>Return</kbd> (also called <kbd>Enter</kbd>)
the CLI reads the command,
evaluates it (i.e., executes it),
prints the command’s output,
and loops around to wait for another command.
Let's see how to do that next!

## Your first words

The default prompt character:

```bash
$
```

Typing in the `whoami` command for showing your username and pressing enter to run it:

```bash
whoami
```

```out
username
```

The `pwd` command stands for Print Working Directory.

```bash
pwd
```

```out
# Linux
/home/username

# MacOS
/Users/username

# Windows
/c/Users/username
```

Our first shell commands will let us explore our folders and files,
and will also introduce us to several conventions that most command line tools follow.
To start,
when `bash` runs it presents us with a prompt
to indicate that it is waiting for us to type something.
This prompt is a simple dollar sign by default (`$`).
However,
different shells may use a different symbol:
for example, the default `zsh` shell on Macs uses `%`.

If we run the command `whoami`,
the computer will tell us who we are (our login name),
and if we run `pwd` (Print Working Directory)
the shell tells us where we are
(the `home` directory by default when we launch the shell).

Each user has a `home` directory;
the function and location of this directory differs a little bit between operating systems.
On Linux it is usually `/home/username`,
on MacOS it is `/Users/username`,
and on Windows it will show up as `/c/Users/username`
(if you have Git Bash installed).
Our examples in this module shows the Linux directory structure,
but you will see that some of the other modules
show what we would see on MacOS or Windows.

## Exploring the files system

Now that we know where we are,
let’s see what we have using the command `ls` (short for “listing”),
which prints the names of the files and directories in the current directory.

```bash
ls
```

```out
Downloads     Music
Documents     todo.txt
Pictures      my_program
```

Again,
our results may be different depending on our operating system
and what files or directories we have.


## Using commands with options

```bash
ls -F
```

```out
Downloads/    Music/
Documents/    todo.txt
Pictures/     my_program*
```

We can modify output of `ls` by providing optional "flags".
In this example, the `-F` option
tells `ls` to decorate the output to show what type of file each entry is.
A trailing `/` indicates a directory,
while a trailing `*` tells us the file is a runnable program.
Depending on our setup,
the shell might also use colors to indicate whether each entry is a file or directory.


:::{.activity}
## Lecture 1 Activity 3

Can you recognize the following folders in the picture?

1. Home directory
2. Parent directory
3. Working directory
4. Root directory



:::

## Navigating and manipulating the filesystem using Bash

Navigating the filesystem can be done
via your visual operating system's user interface
(*e.g.,* Finder, Explorer, Nautilus, *etc*)
or using a text-based shell such as Bash.
In the lecture,
we cover some of the most useful commands
with the main goal of making you familiar with moving around using Bash.
The most important commands that we covered are listed in the table below.
Remember that you can press <kbd>TAB</kbd>
to automatically complete commands and file names as you are typing.

### Shell commands for navigating and manipulating the filesystem

| Command | Purpose                 | Example use                     |
|---------|-------------------------|---------------------------------|
| `pwd`   | Print Working Directory | `pwd`                           |
| `ls`    | LiSt contents           | `ls Documents`                  |
| `cd`    | Change Directory        | `cd Desktop`                    |
| `mkdir` | MaKe a DIRectory        | `mkdir my-new-directory`        |
| `rm`    | ReMove a file           | `rm file-to-remove`             |
| `mv`    | MoVe a file             | `mv file-to-move new-location`  |
| `cp`    | CoPy a file             | `cp file-to-copy copy-location` |


To read the documentation for a command,
you can type `man <your_command>` or `<your_command> --help`
(e.g., to get the help pages for the `cp` copy command, `man cp` or `cp --help`).
In the bash configuration that you copied from the installation instructions,
we have also included a few shortcuts (also called "aliases")
in the MDS installation instructions
to make it easier to type commonly used commands.
The two relevant ones for this section are

- `l` which lists details of all the files in the current directory
and sorts them by modification time.
- `jl` which is an alias for `jupyter lab`.

Don't worry if you have a hard time remembering some of these in the beginning.
The key is to practice often,
and when you forget a specific command
you can type in the help command we have created `mds-help`
in you terminal to see a list of all commands we use in MDS.

### Useful shell special characters

| Symbol | Definition                |
|--------|---------------------------|
| `.`    | Current working directory |
| `..`   | Parent directory          |
| `~`    | HOME directory            |

:::{.activity}
## Lecture 1 Activity 4
Which command would move you to the parent directory?

```
A. cd .
B. cd ..
C. cd ~
D. cd
```
:::

:::{.exercise}
### Lecture 1 Exercise 1

**Navigating the Command Line**

**Objective:** By completing this exercise, students will practice basic navigation commands in the command line interface.

**Instructions:**

1. **Figure Out Where You Are When You Open Your Command Line:**
   - Open your command line interface (Terminal on Mac/Linux, Command Prompt/PowerShell on Windows).
   - Use the command to display your current directory.

   ```bash
   pwd
   ```

   This command will print the working directory you are currently in.

2. **Navigate to Your Documents Folder:**
   - Change your directory to the Documents folder. Use the appropriate command based on your operating system.

   ```bash
   cd ~/Documents
   ```

   This command changes your directory to the Documents folder located in your home directory.

3. **Navigate from Your Documents Folder to Your Desktop:**
   - From the Documents folder, change your directory to the Desktop folder.

   ```bash
   cd ../Desktop
   ```

   This command uses `..` to move up one level to the home directory and then enters the Desktop folder.

:::



:::{.exercise}
### Lecture 1 Exercise 2

**Understanding the `ls` Command and Directory Navigation**

**Objective:** By completing this exercise, students will learn to use the `ls` command with options and understand directory structures.

**Instructions:**

1. **Check the Documentation of the `ls` Command:**
   - Open your command line interface (Terminal on Mac/Linux, Command Prompt/PowerShell with Git Bash on Windows).
   - View the manual or help documentation for the `ls` command to understand its options.

   ```bash
   man ls
   ```

   or

   ```bash
   ls --help
   ```

   - Identify what the `-a` option does.

2. **Use the `ls -a` Command:**
   - In your command line, navigate to your home directory (if not already there).

   ```bash
   cd ~
   ```

   - List all files, including hidden files, using the `-a` option with the `ls` command.

   ```bash
   ls -a
   ```

   - Note the presence of `./` and `../` in the output.

3. **Understand `./` and `../`:**
   - Explain what `./` and `../` represent in the context of the directory structure.

**Questions:**

1. What does the `-a` option do in the `ls` command?

2. What do the `./` and `../` folders represent as output of the `ls -a` command?

:::


## Working with Files and Directories

We now know how to explore files and directories,
but how do we create them?
That's what we will find out in this chapter.

```bash
cd ~/Documents

# MaKe Directory
mkdir notes
```

### Naming guidelines

Let's go back to the `~/Documents` directory and create a subfolder called `notes`.
For this we first use the `~` home  directory shortcut,
and then the `mkdir` command.

We will talk more about useful naming conventions in module 6.
For now it is enough if you remember these three guidelines.

1. Don't use spaces
2. Don't begin the name with a `-` (hyphen)
3. Stick to digits and letters (preferably lower case)

### Creating a file

Since we just created the `notes` directory,
`ls` doesn't display anything when we ask for a listing of its contents.

```bash
ls -F notes
```

Let’s change our working directory to `notes`,
then use the VS Code editor to create a file called `my-first-note.md`.
The command to invoke VS Code from the command line is `code`,
and if we pass it a filename as an argument,
the file will be created in the current directory.
Now try launching it yourself!

```bash
cd notes
code my-first-note.md
```

The `.md` syntax means that we want to create a markdown file.
File extensions like `.md` and `.py` don't change anything about the content of the file,
but they are an indicator to other program such as VS Code what type of content
we are going to put inside the file.
These program can then activate special functions for certain files,
such as using the appropriate colors to highlight headings and functions.

Let's move on to the next slide to see how it looks when we are editing file in VS Code.

### Editing a file in VS Code

This is what it looks like to edit a text file in VS Code.
You will see that there is a small white circle next to the file name.
This is an indication that the file is not saved yet.

When we are done adding our edits,
we can hit
<kbd>Ctrl</kbd> + <kbd>s</kbd> (Windows/Linux) <kbd>⌘</kbd> + <kbd>s</kbd> (Mac)
to save the file,
which will make the white circle disappear.
After saving the file,
you can optionally close VS Code before returning to the terminal.

We are only using VS Code as a simple text editor here,
but it is worth knowing that you can use use it as a powerful editor for code as well.


### Viewing the content of a file in the terminal

If we now type `ls` we will see the name of the file we just created in VS Code.

```bash
ls
```

```out
my-first-note.md
```

To see the content of the file,
we can use the `head` command.
`head` show the first few lines of a file
(just as when we use `df.head()` in pandas),
and optionally takes an argument for how many lines to show:
`head -n 10` (the default is five).

```bash
head my-first-note.md
```

```out
These are the first
few lines of content
in the file.
```

`head` works with all plain text files,
such as code and markdown files,
your results will vary with binary files
such as word documents, spreadsheets, or images.
Generally, binary file output will be incomprehensible.

To see the last few lines,
you can use `tail` instead of `head`.

### Moving files and directories

To **m**o**v**e a file around,
we can use the `mv` command.
For example,
to move the text file we just created to the parent directory,
we would type:

```bash
mv my-first-note.md ..
ls ..
```

```out
Downloads/    Music/        my-first-note.md
Documents/    todo.txt
Pictures/     my_program*
```

This moves it to the parent directory,
and keeps the name the same as we can see if we do `ls ..`

If we want to move the file back to the current directory,
we can use the `.` shortcut:

```bash
mv ../my-fist-note.md .
```

### Renaming a file

Renaming a file can be thought of as moving it to a new filename
either in the same or a different directory.
This might initially seem a bit different from a graphical file browser,
where renaming and moving are two distinct functions,
but the underlying operation is actually the same.

To give our file a more descriptive name,
we can use `mv` to include the date we created this file,

```bash
mv my-fist-note.md notte_2022-06-20.md # intensional typo
```

We intentionally made a typo here,
let's fix it while showing off the functionality of the `-v` (verbose) option,
which prints what was moved/renamed so that it is easier to keep track of.

```bash
mv -v notte_2022-06-20.md note_2022-06-20.md
```

```out
renamed 'notte_2022-06-20.md' -> 'note_2022-06-20.md'
```

We must be careful when specifying the destination
because `mv` will overwrite existing files without warning.
The option `-i` ("interactive") makes `mv` ask us for confirmation before overwriting,.
`mv` also works on directories where it renames the directory without changing its contents.

### Copying files and directories

The `cp` command create a **c**o**p**y of a file.
Just as with `mv`,
we optionally rename the file as we are copying it.

Here, we first copy the note we created to the parent directory and then
show how we could use `cp` to create a backup of our file in the same directory.
Remember that the file extension does not change the content of the file,
so this is only a visual indicator for the file type,
and helps the computer pick a default program when you open it.

```bash
# Copy the file to the parent directory
cp note_2022-06-20.md ..
```

```bash
# Create a backup copy in the same directory
cp note_2022-06-20.md note_2022-06-20.md.bkp
ls
```

```out
note_2022-06-20.md
note_2022-06-20.md.bkp
```

If we ever wanted to copy a directory,
we would have to specify the `-r` flag,
which indicates that we want to copy the directory recursively,
i.e. also making a copy of all the directory contents.

### Wildcards

Wildcards (also called "globbing") simplifies targeting multiple files with similar names
in the same command.
The most commonly used wildcard is `*` (a single asterisk).
It matches zero or more characters,
so typing `ls *.md` list all of the markdown files in the current directory.

If we wanted to list all the files created in July (month 7),
we would need to type ls `ls *07*`,
which means that the filename can include anything before and after `07`.
If we would have left out the second `*` and type `ls *07`
we would not have seen any matches because there is not file that end in 07,
they all have some characters after.

Using wildcards is helpful when we want to delete, move, or copy files
with a predictable naming pattern.

<!-- TODO: not particularly happy with this example, because it's forcing you
           to use + learn touch (another thing we'd have to explain)
           if we start off from the swcarpentry bash directory,
           we can show this example much easier.
-->

```bash
touch note_2022-06-20.md.bkp note_2022-07-02.md note_2022-07-02.md.bkp
ls *.bkp
```

```out
note_2022-06-20.md.bkp
note_2022-07-02.md.bkp
```

```bash
ls *07*
```

```out
note_2022-07-02.md
note_2022-07-02.md.bkp
```

### Searching for text patterns in files

We can use the `grep` command to search for text in files.
To search for the word "the" in our notes file,
we can type `grep "the" note_2022-06-22.md`.
This will return every line in the file that contains the word "the".

```bash
grep "the" note_2022-06-22.md
```

```out
note_2022-06-22.md:These are the first
note_2022-06-22.md:in the file.
```

### Seeing the history of commands

Occasionally,
we might want to re-use a command that we used in the past,
but don't remember exactly what it was.
The `history` command can help us with this
as it lists all the previous commands we have run
and in which order we ran them.

```bash
history
```

```out
1  pwd
2  ls
3  cd Documents/
4  cd
5  ls -F -a
6  history
```

### Reading the manual

How can we find out what options like `-t` and `-r` do if we don't know of them already?
By reading the built-in help manual!
Typing `man ls` brings up the manual help page for the `ls` command.
For some systems, you may need to run `--help` instead of `man`, e.g., `ls --help`.


```bash
man ls
```

```out
LS(1)                                                    User Commands

NAME
       ls - list directory contents

SYNOPSIS
       ls [OPTION]... [FILE]...

DESCRIPTION
       List  information  about the FILEs (the current directory by default).  Sort entries alphabetically if none of -cftuvSUX
       nor --sort is specified.

       Mandatory arguments to long options are mandatory for short options too.

       -a, --all
              do not ignore entries starting with .

       -A, --almost-all
              do not list implied . and ..

       --author
              with -l, print the author of each file

       -b, --escape
              print C-style escapes for nongraphic characters

       --block-size=SIZE
              with -l, scale sizes by SIZE when printing them; e.g., '--block-size=M'; see SIZE format below

       -B, --ignore-backups
              do not list implied entries ending with ~
...
```

Some helpful commands while navigating the manual pages in `less`

<!-- TODO: paste in the keyboard up and down arrow -->

- <kbd>q</kbd>: Quits the manual and takes you back to the shell prompt
- <kbd>Space</kbd>: Go down the page
- <kbd>b</kbd>: Go **B**ack up the page
- <kbd>Up Arrow</kbd> / <kbd>Down Arrow</kbd>: you can also use the arrow keys to go up and down the page
- <kbd>/</kbd>: starts a search
    - Type in the search term and press <kbd>Enter</kbd> to go to the first hit
    - <kbd>n</kbd>: To continue to the **n**ext search hit
    - <kbd>Shift</kbd> + <kbd>n</kbd>.: Go to the previous hit

:::{.callout-note}
If you notice a `:` in the bottom left of your terminal,
you are browsing the file in a program called `less`.
To quit, you can press the letter <kbd>q</kbd> on your keyboard.

You can enter `less` in multiple ways when working in the terminal,
it is not exclusive to only looking at the `man` pages.
:::

### Relative vs absolute path

We've seen a bit of usage around `.` and `..`.
Let's talk a bit more on how to reference files and directories.

Let's say you are here:

```bash
pwd
```

```
/Users/timberst/Documents
```

These are the files in your current working directory:

```bash
ls
```

```
DSCI-100  MDS-2018-19   textbooks   research
```

You can navigate to the research directory using one of two options, a relative path:

```bash
cd research
```

or, using an absolute path:

```bash
cd /Users/timberst/Documents/research
```

What is the difference?

When you use a relative path with a command like `ls` or `cd`,
it tries to find that location from where we are,
rather than from the root of the file system.
However, when you specify the absolute path to a directory
by including its entire path from the root directory,
which is indicated by a leading slash,
the leading `/` tells the computer to follow the path from the root of the file system,
so it always refers to exactly one directory,
no matter where we are when we run the command.

## References

* https://merely-useful.tech/py-rse/bash-tools.html
* https://missing.csail.mit.edu/
* https://datascienceatthecommandline.com/2e/chapter-1-introduction.html
* https://datascienceatthecommandline.com/resources/50-reasons.pdf