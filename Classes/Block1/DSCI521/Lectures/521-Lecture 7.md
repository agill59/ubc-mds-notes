---
title: "Virtual Environments: Python + Conda"
---

## Learning outcomes

1. Understand what is a computational environment and how can ensure the reproducibility of a project
2. Differenciate Python, Anaconda, MiniConda, Conda and `pip`
3. Manage packages and environments in Python using Conda
4. Manage packages and environments in R using `renv`

**Platform in focus** Conda

:::{.activity}
## Activity 1

Imagine you're working on a project with a colleague, and you've created a reproducible report to share your analysis. You want them to replicate your work on their own machine. To demonstrate this, try running the following code on your system:

```python
from palmerpenguins import load_penguins
penguins = load_penguins()
penguins.head()
```
:::

## Virtual environments

Virtual environments let you have multiple versions of packages
and programs on the same computer
without them creating conflicts with each other.
You will be using virtual Python and R environments
throughout the program to setup your packages for different courses.

## Conda and Its Ecosystem

[**conda**](http://conda.pydata.org/docs/)
is an **open source `package` and `environment` management system**.
It allows you to install libraries and packages as well as their accompanying dependencies.
It can work with _any_ programming language, but is mostly popular in the Python community.

In MDS and quite often, broadly in data science projects, we use Miniforge distribution, which was created to be a bare-bones installation with conda and using `conda-forge` as the package channel. This is what we had you install in the MDS Python installation instructions. There are many variations of many parts of conda, see @sec-conda-history.

:::{.activity}
## Activity 2

Which of the following items is NOT a benefit of using Conda environments?

```
A. Increase code performance
B. Helping with reproducibility
C. Using different versions of the same package
D. Creating isolated computational environment for testing new packages
```

:::

## Managing Conda

Let's first start by checking if conda is installed.

```bash
conda --version
which conda
```

To see which conda commands are available,
type `conda --help`.
To see the full documentation for any command of these commands,
type the command followed by `--help`.
For example,
to learn about the conda update command:

```bash
conda update --help
```

### Update Conda

Let's update our conda to the latest version.
Note that you might already have the latest version since we downloaded it recently.

```bash
conda update conda
```

You will see some information about what there is to update
and be asked if you want to confirm.
The default choice is indicated with `[]`,
and you can press <kbd>Enter</kbd> to accept it.
It would look similar to this the output below.

:::{.tip}
## Tip 
You should see `conda-forge` listed in the top of the output,
if not, set the proper channel before running more conda commands (next section).
You can cancel the current terminal operation by pressing
<kbd>Ctrl</kbd> + <kbd>c</kbd>
:::

```bash
$ conda update conda
Retrieving notices: ...working... done
Channels:
 - conda-forge
Platform: linux-64
Collecting package metadata (repodata.json): done
Solving environment: done

## Package Plan ##

  environment location: /home/dan/.pyenv/versions/miniforge3-latest

  added / updated specs:
    - conda


The following packages will be downloaded:

    package                    |            build
    ---------------------------|-----------------
    ca-certificates-2024.8.30  |       hbcca054_0         155 KB  conda-forge
    certifi-2024.8.30          |     pyhd8ed1ab_0         160 KB  conda-forge
    conda-24.7.1               |  py310hff52083_0         940 KB  conda-forge
    frozendict-2.4.4           |  py310hc51659f_0          48 KB  conda-forge
    libgcc-14.1.0              |       h77fa898_1         827 KB  conda-forge
    libgcc-ng-14.1.0           |       h69a702a_1          51 KB  conda-forge
    libgomp-14.1.0             |       h77fa898_1         449 KB  conda-forge
    openssl-3.3.2              |       hb9d3cd8_0         2.8 MB  conda-forge
    ------------------------------------------------------------
                                           Total:         5.3 MB

The following NEW packages will be INSTALLED:

  frozendict         conda-forge/linux-64::frozendict-2.4.4-py310hc51659f_0
  libgcc             conda-forge/linux-64::libgcc-14.1.0-h77fa898_1

The following packages will be UPDATED:

  ca-certificates                       2024.2.2-hbcca054_0 --> 2024.8.30-hbcca054_0
  certifi                             2024.2.2-pyhd8ed1ab_0 --> 2024.8.30-pyhd8ed1ab_0
  conda                              24.1.2-py310hff52083_0 --> 24.7.1-py310hff52083_0
  libgcc-ng                               13.2.0-h807b86a_5 --> 14.1.0-h69a702a_1
  libgomp                                 13.2.0-h807b86a_5 --> 14.1.0-h77fa898_1
  openssl                                  3.2.1-hd590300_1 --> 3.3.2-hb9d3cd8_0


Proceed ([y]/n)?


Downloading and Extracting Packages:

Preparing transaction: done
Verifying transaction: done
Executing transaction: done
```

In this case,
conda itself needed to be updated,
and along with this update some dependencies also needed to be updated.
There is also a NEW package that was INSTALLED in order to update conda.
You don't need to worry about remembering to update conda,
it will let you know if it is out of date when you are installing new packages.

You can find more information on managing `conda` in the official documentation:
<https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-conda.html>

### Setup / Confirm Conda-Forge Channel

The official conda-forge documentation has the setup instructions to make sure it is
set up as the default conda package channel:
<https://conda-forge.org/docs/user/introduction/>

You can check

```bash
conda config --show channels
```

You should see conda-forge at the top of the list (ideally the only entry)

```
channels:
  - conda-forge
```

From the
[managing channel documentation](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-channels.html):

> There will be no channel collisions if you use only the defaults channel.
> There will also be no channel collisions if all of the channels you use
> only contain packages that do not exist in any of the other channels in your list.
> The way conda resolves these collisions matters only when you have multiple channels
> in your channel list that host the same package.

If you do not see `conda-forge` listed and if it's not listed on the top of the channels list
run the following commands to add and set it to the top.

Add conda-forge as the highest priority channel.

```bash
conda config --add channels conda-forge
```

Activate strict channel priority (strict will be activated by default in conda 5.0).

```bash
conda config --set channel_priority strict
```

You can then re-run `conda config --show channels` to confirm that `conda-forge`
is listed in the highest priority.

## Managing Environments

Using `conda`, you can create an isolated python *environment* for your project.
An environment is a set of packages that can be used in one or multiple projects.
There are several major  benefits of using environments:


- You can guarantee that someone else can reproduce your project
  by specifying which package versions you used
  and making it easy for others to install the same versions.
- If two of your projects rely on different versions of the same package,
  you can install these in different environments.
- If you want to play around with a new package,
  you don't have to change the packages you use for your data analysis
  and risk messing something up.
- When you develop your own packages,
  it is essential to use environments,
  since you want to to make sure you know exactly which packages yours depend on,
  so that it runs on other systems than your own.

The default environment is the `base` environment,
which contains only the essential bare-bones python installation from Miniforge.
You can see that your shell's prompt string is prefaced with `(base)`
when you are inside this environment.
In the setup guide,
we gave your instructions for how to activate this environment by default
every time you open Bash.
There are two ways of creating a conda environment.

1. Manual specifications of packages.
2. An environment file in YAML format (`environment.yaml`).

More information about managing conda environments can be found on the documentation page:
<https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html>

### Creating environment by manually specifying packages

We can create `test_env` conda environment by typing `conda -n <name-of-env>`.
However,
it is often useful to specify more than just the name of the environment,
e.g. the channel from which to install packages, the Python version,
and a list of packages to install into the new environment.

In the example below,
we are creating a `test_env` environment
that uses python 3.12 and a list of libraries: `jupyterlab` and `pandas`.
Miniforge is already set to use the conda-forge channel by default,
but we can use the `-c` flag to specify any package channel (even the anaconda channel)

```bash
conda create -n test_env -c conda-forge python=3.12 pandas=2.2.2 jupyterlab
```

:::{.note}
## Note

In the MDS instructions you should have conda-forge set as a default channel,
so the `-c conda-forge` is redundant,
but you may see it while looking for installation instructions.
:::

conda will solve any dependencies between the packages like before
and create a new environment with those packages.
Usually,
we don't need to specify the channel,
but in this case I want to get the very latest version of these packages,
and they are made available in `conda-forge`
before they reach the default conda channel.

To activate this new environment,
you can type `conda activate test_env`
(and `conda deactivate` for deactivating).
Since you will do this often,
we created an alias shortcut `ca`
that you can use to activate environments.
To know the current environment that you're in you can look at the prefix
of the prompt string in your shell which now changed to (`test_env`).
And to see all your environments,
you can type `conda env list`.

#### Removing environments

If you are creating environments for practice, or you want to recreate an environment you can delete your conda environments by:

```bash
# look for all the installed environments
conda env list
```

```bash
# delete an environment
conda remove --name ENV_NAME --all
```

Similarly, all your environments are installed within your `miniconda3` folder,
which is typically located in `~/miniconda3`
In here you will see an `env` folder.
If you delete the folder with the corresponding environment name (e.g., with `rm`)
you can also delete an environment this way too.

### Sharing Environments with others

To share an environment, you can export your conda environment to an environment file,
which will list each package and its version
in the format `package=version=build`.

Exporting your environment to a file called `environment.yaml`
(it could be called anything,
but this is the conventional name
and using it makes it easy for others
to recognize that this is a conda env file,
the extension can be either `.yaml` or `.yml`):

:::{.warning}
## Warning 

While you can choose to export your _entire_ environment into a file

```bash
conda env export -f environment.yaml
```

You typically do not want to do this in a shared repository because
dependencies may change depending on the operating system used.
If there is an actual dependency that needs to be managed,
you will manually track it in the `environment.yaml`.

The same general rule will apply for a `requirements.txt` file
and using `pip freeze`.
:::

Remember that `.yaml` files are plain text,
so you can use a text editor such as VS Code to open them.
If you do,
you will realize that this environment file has A LOT more packages
than `jupyterlab` and `pandas`.
This is because the default behavior is to also list the dependencies
that were installed together with these packages,
e.g. `numpy`.
This is good in the sense that it gives an exact copy of *everything*
in your environment.

However,
some dependencies might differ between operating systems,
so this file *might* not work with someone from a different OS.
To remedy this,
you can append the `--from-history` flag,
which look at the history of the packages you explicitly told conda to install
and only list those in the export.
The required dependencies will then be handled in an OS-specific manner during the installation,
which guarantees that they will work across OSes.
This `environment.yaml` file would be much shorter and look something like this:

```yaml
name: test_env
channels:
  - conda-forge
dependencies:
  - conda
  - python=3.12
  - pandas==2.2.2
  - jupyterlab
```

Importantly,
this will not include the package version
unless you included it when you installed
with the `package==version` syntax.
For an environment to be reproducible,
you **NEED** to add the version string manually.

:::{.activity}
## Activity 3

What are good practices when exporting environments for git collaboration? (one or more correct answers)

- A. Exporting your entire environment via `conda env export -f environment.yaml`
- B. Including only pip installed packages via `pip freeze > requirements.txt`
- C. Always exporting `base` conda environments for reproducibility
- D. Using `--from-history` flag to reduce the number of explicit requirements and potential conflicts
- E. Including your conda path into `environment.yml`: `prefix /Users/username/miniforge3` to make sure conda knows where to find files

:::

### Creating environment from an environment file

Now, let's install `environment.yaml` environment file above so that we can create a conda environment called `test_env`.

```bash
$ conda env create --file environment.yaml
```

### Copying an environment

We can make an exact copy of an environment to an environment with a different name.
This maybe useful for any testing versus live environments or different Python 2.7 versions for the same packages.
In this example, `test_env` is cloned to create `live_env`.

```bash
conda create --name live_env --clone test_env
```

### Deleting an environment

Since we are only testing out our environment,
we will delete `live_env` to remove some clutter.
*Make sure that you are not currently using `live_env`.*

```bash
conda env remove -n live_env
```

### Making environments work well with JupyterLab

In brief,
you need to install the `ipykernel` package
in any new environment your create,
and the `nb_conda_kernels` package needs to be installed
in the environment where JupyterLab is installed.

By default,
JupyterLab only sees the conda environment where it is installed.
Since it is quite annoying to install JupyterLab and its extensions separately in each environment,
there is a package called `nb_conda_kernels` that makes it possible
to have a single installation of JupyterLab access kernels in other conda environments.
This package needs to be installed in the conda environment
where JupyterLab is installed.

Lastly,
you also need to install a kernel in the new conda environment
so that it can be detected by `nb_conda_kernels`.
This kernel can be installed via the package `ipykernel` for Python
and the `r-irkernel` package for R
([more info in the nb_conda_kernels README](https://github.com/Anaconda-Platform/nb_conda_kernels#installation)).

## Managing Packages

In this section, we will cover how to manage packages within any environment.

### List packages in the current environment

We will now check the packages that are available to us.
The command below will list all the packages in an environment, in this case `test_env`.
The list will include versions of each package, the specific build,
and the channel that the package was downloaded from.
`conda list` is also useful to ensure that you have installed the packages that you desire.

```bash
conda list
```

```bash
# packages in environment at /home/dan/.pyenv/versions/miniforge3-latest/envs/test_env:
#
# Name                    Version                   Build  Channel
_libgcc_mutex             0.1                 conda_forge    conda-forge
_openmp_mutex             4.5                       2_gnu    conda-forge
bzip2                     1.0.8                h4bc722e_7    conda-forge
ca-certificates           2024.8.30            hbcca054_0    conda-forge
ld_impl_linux-64          2.40                 hf3520f5_7    conda-forge
libexpat                  2.6.3                h5888daf_0    conda-forge
libffi                    3.4.2                h7f98852_5    conda-forge
libgcc                    14.1.0               h77fa898_1    conda-forge
libgcc-ng                 14.1.0               h69a702a_1    conda-forge
libgomp                   14.1.0               h77fa898_1    conda-forge
libnsl                    2.0.1                hd590300_0    conda-forge
libsqlite                 3.46.1               hadc24fc_0    conda-forge
libuuid                   2.38.1               h0b41bf4_0    conda-forge
libxcrypt                 4.4.36               hd590300_1    conda-forge
libzlib                   1.3.1                h4ab18f5_1    conda-forge
ncurses                   6.5                  he02047a_1    conda-forge
openssl                   3.3.2                hb9d3cd8_0    conda-forge
pip                       24.2               pyh8b19718_1    conda-forge
python                    3.12.5          h2ad013b_0_cpython    conda-forge
readline                  8.2                  h8228510_1    conda-forge
setuptools                74.1.2             pyhd8ed1ab_0    conda-forge
tk                        8.6.13          noxft_h4845f30_101    conda-forge
tzdata                    2024a                h8827d51_1    conda-forge
wheel                     0.44.0             pyhd8ed1ab_0    conda-forge
xz                        5.2.6                h166bdaf_0    conda-forge
```

:::{.note}
## Note

You will typically want to have an empty `base` environment where no
packages are installed.
All your work in separate classes/projects should exist in their own environment.

The above set of packages is what an empty environment with only Python 3.12 installed
looks like (on a linux machines).
Notably, the typical python data science packages, `pandas`, `numpy`, `jupyter`, etc,
are not in the environment.
:::

### Searching for a certain package

Some packages might not be available in conda, but are available in [pypi](https://pypi.python.org/pypi).
For example, we will search for rasterio within the [anaconda cloud](https://anaconda.org/).
*It is not necessary to create an account with anaconda cloud, unless you'd like to contribute in the future when you are pro with conda.*

In this example, we will use rasterio from conda-forge. The anaconda cloud page for rasterio will show how to install the package, compatible OS, individual files for that package, etc.

With conda you can do this search within the command line:

```bash
conda search rasterio
```

### Installing conda package

Under the name column of the result in the terminal or the package column in the Anaconda Cloud listing,
shows the necessary information to install the package.
e.g. conda-forge/rasterio.
The first word list the channel that this package is from and the second part shows the name of the package.

To install the latest version available within the channel, do not specify in the install command. We will install version 0.35 of `rasterio` from conda-forge into `test_env` in this example. Conda will also automatically install the dependencies for this package.

```bash
conda install -c conda-forge rasterio=1.3.11
```

## Removing a conda Package

We decided that rasterio is not needed in this tutorial, so we will remove it from `test_env`.
Note that this will remove the main package rasterio and its dependencies (unless a dependency was installed explicitly at an earlier point in time or is required be another package).

```bash
conda remove -n test_env rasterio
```

```bash
$ conda remove -n test_env rasterio
Channels:
 - conda-forge
Platform: linux-64
Collecting package metadata (repodata.json): done
Solving environment: - WARNING conda.conda_libmamba_solver.solver:_export_solved_records(832): Tried to unlink __unix but it is not installed or manageable?
done

## Package Plan ##

  environment location: /home/dan/.pyenv/versions/miniforge3-latest/envs/test_env

  removed specs:
    - rasterio


The following packages will be REMOVED:

...

Proceed ([y]/n)?

Preparing transaction: done
Verifying transaction: done
Executing transaction: done

```

:::{.activity}
## Activity 4

What command will activate the test_env environment you just created?

```
A. conda activate environment
B. conda start test_env
C. conda activate test_env
D. conda start environment
```

:::

### When there is no conda package

If a conda package does not exist but does exist as a python package,
you can use `pip` to install packages with `pip install <package name>`.
When a package is installed using pip,
your `environment.yml` will have a section for `pip` that will
list the packages installed via pip.

You do want to be careful not to constantly mix conda install commands
with pip install commands.
You are essentially using 2 different package resolution systems,
and packages may not always be installed in the locations the other package manager
knows about.

If you end up installing `conda` and `pip` packages over the course a project
you may end up with a corrupted python package environment.
This is why the `environment.yml` is important,
you can easily remove the corrupted environment, and install everything you need
in a single step.


## Confirming your work

Multiple times in this lesson when we are searching, installing, removing packages
the output will mention `conda-forge`.
This makes sense since the current setup has only `conda-forge` in the
package channels.
You should not see `anaconda` or `default` in the output text before you
confirm an action.https://conda.org/blog/2024-08-14-conda-ecosystem-explained/

### Conda history and terminology {#sec-conda-history}

There is a lot of history and backstory to conda, but we'll just talk about the main players
in the ecosystem.

#### Anaconda and Miniconda

"Anaconda" can refer to one of three things:
Anaconda, Inc the company,
anaconda the python distribution,
and anaconda the conda package channel.

[Anaconda, Inc](https://www.anaconda.com/) (formerly, Continuum Analytics) is the **company** that
originally created `conda`.
Anaconda also created the
[anaconda distribution](https://www.anaconda.com/download),
which comes with `conda` and many other pre-installed Python packages.
The anaconda distribution uses the _anaconda_ channel as the default place to look for `conda` packages.

#### Conda-forge and Miniforge

[Miniconda](https://docs.anaconda.com/miniconda/) is also maintained by Anaconda, Inc.
It uses `conda` as the package installer, and also anaconda as the default package channel.
The difference is that miniconda does not come with pre-installed python packages.
The "mini" is for a smaller more bare-bones installation.

The anaconda channel is completely maintained by Anaconda, Inc.
People who wanted get their own packages installable by `conda` needed their own mechanism
to share their packages.
Also, because Anaconda needs the anaconda distribution to work for their enterprise customers,
not every package is on the latest version or would take a long time to get the latest versions
of packages you needed.
This led to the creation of
[Conda-Forge](https://conda-forge.org/),
a __community__ maintained channel for conda packages.
Now package maintainers can publish the latest versions of their packages
to a community maintained repository.

Since Miniconda is maintained by Anaconda, Inc with the default anaconda channel,
Miniforge was created to be a bare-bones installation with conda and using conda-forge
as the package channel.
This is what we had you install in the
[MDS Python installation instructions](https://ubc-mds.github.io/resources_pages/installation_instructions/).


### Mamba

`conda` allows you to install libraries and packages along with their dependencies.
One of the main complaints about `conda` was the package dependency resolution took a very long time.
`mamba` was created to be a faster C++ implementation of the dependency solver.
Since then, `mamba` has been a drop-in replacement for `conda`,
and the `mamba` solver is now the default solver used in miniforge.

### In short...

- Anaconda (capital A): is a company
- anaconda (lower-case a): is a python distribution
- anaconda channel: Set of packages and versions maintained by Anaconda
- `conda`: is a program that can install packages and their dependencies
- conda-forge is a community maintained channel for conda packages
- miniconda: is an Anaconda maintained python distribution that defaults to the anaconda package channel
- miniforge: is a community maintained python distribution that defaults to the conda-forge package channel
- `mamba` is a faster drop-in replacement for `conda`

You can read more about these differences here:
<https://conda.org/blog/2024-08-14-conda-ecosystem-explained/>

## Additional Links

- Pip and conda interoperability: <https://docs.conda.io/projects/conda/en/latest/user-guide/configuration/pip-interoperability.html>
- They're working on better conda + pip support: <https://github.com/conda-incubator/conda-pypi>
- micromamba project: <https://mamba.readthedocs.io/en/latest/user_guide/micromamba.html>
- pixi (also defaults to conda-forge): <https://github.com/prefix-dev/pixi>
- Conda and Anaconda terms of use: <https://www.anaconda.com/blog/is-conda-free>

## Attribution

The conda virtual environment section of this guide
was originally published at <http://geohackweek.github.io/> under a CC-BY license
and has been updated to reflect recent changes in conda,
as well as modified slightly to fit the MDS lecture format.

---
title: "Virtual Environments: R"
---

## Learning outcomes

{{< include ../learning_objectives/lo-ch-07.qmd >}}

**Platform in focus** R, RStudio

## R environments

In R,
environments are managed by `{renv}`,
which works with similar principles as `conda`,
and other virtual environment managers,
but the commands are different.
To see which commands are used in `{renv}`,
you can visit the project website:
<https://rstudio.github.io/renv/articles/renv.html>.

Briefly,

- `renv::init()`: create a new env in the current directory/project
- `renv::snapshot()`: save/export the environment to a file (`renv.lock`),
and installing and removing packages are done as usual
via the `install.packages()` and `remove.packages()` commands.
- `renv::restore()`: updates current environment from the `renv.lock` file

`{renv}` assumes and enforces the "one project one environment" mantra of virtual environments.
Unlike conda that lets you "activate" an environment and let you move across different projects,
`{renv}` prefers it when all your work is contained in a single project.

:::{.note}
## Note

For MDS, this might mean you may need a separate `{renv}` file for each lab assignment.
:::

:::{.activity}
## Activity 1

Which of the following commands is used to initialize a new `renv` environment in an R project?

```
A. `renv::create()`
B. `renv::activate()`
C. `renv::init()`
D. `renv::start()`
```

:::


One of the other distinguishing factors between R and Python is CRAN (the package repository for R),
_guarantees_ that on any given calendar day,
all the packages on CRAN are installable.

## An example R Project

Here we'll go through an example of using `{renv}` in the context of an RStudio project.
Unlike conda environments, you don't necessarily, "activate" and "deactivate" it manually.
One of the nice things is `{renv}` works naturally within an RStudio project.

The New Project creation wizard in R Studio has an option to enable
the folder as a git repository, and also to set it up with `{renv}`.

This does the same thing as running `renv::init()` in the current project.

![](img/rstudio_project-renv.png)


### Environment Files

When you either create a new project with renv enabled or run `renv::init()`
in the current working directory (`getwd()`),
a few new files will be created

1. `.Rprofile` in the current project directory
2. `renv/` directory
3. `renv.lock` file

Before we talk about which each file and directory does,
now when you start a new R session or re-open your RStudio project,
you will see a line that will read something like:

```markdown
- Project '~/Desktop/my_r_project' loaded. [renv 1.0.7]
```

How does R know that there is a project renv?
It's because of the `.Rprofile` file.
This is a special file that R will read at the start of __every__ session.
This file exists in 3 places, it will search for them in the following order,
and once it finds 1 of them, it will not look for any more.

1. The current project or working directory
2. The user home directory
3. The system R installation

There is a similar file called `.Renviron` that only stores key-value environment variables.
You can read more about these initialization files
[here](https://support.posit.co/hc/en-us/articles/360047157094-Managing-R-with-Rprofile-Renviron-Rprofile-site-Renviron-site-rsession-conf-and-repos-conf).

The `{renv}` `.Rprofile` will add a single line:

```R
source("renv/activate.R")
```

This is the line that "activates" the renv environment for you automatically,
assuming you are in the correct working directory.

All the environment files and packages are installed in the `renv/` directory.
This directory has its own `.gitignore` file that will ignore the correct things for you in that directory,
so you do not need to worry about checking in your entire set of packages.

Finally, the `renv.lock` file is the file that contains all the packages used in the repository,
its version, and where it was installed from (e.g., CRAN or GitHub).

### Snapshot Packages

When you install and use R packages in your project,
you will occasionally run `renv::snapshot()`.
This will update the `renv.lock` file with all the packages and dependencies for you.
Do not forget to `add`, `commit`, and `push` this file to your remote repository.

### Restore Packages

As you work on a project with other people and packages get updated,
you will occasionally run `renv::restore()`
to install any missing packages on your local r environment.
Occasionally renv may tell you things are out of sync and will
prompt you to run `renv::status()` or `renv::restore()`.

## Takeaway

`{renv}` is not the same as a conda environment where you can activate an environment,
and move anywhere in your filesystem to run code.
Many virtual environment systems actually assume you have a single environment for each
project/directory.
Sometimes IDES allow you do override the currently activated environment,
but it lowers the reproducibility of your projects if you do that.

Finally, the `renv.lock` file does track __every__ package and dependency
you use in your project, which is a bit different from the recommendation we
gave in the conda environment section.
To just track individual packages and not the entire snapshot,
you will learn about the `DESCRIPTION` file used for packaging later on in the program.
However, we can get away with this because CRAN does guarantee all packages
to be installable on any given calendar day.


:::{.activity}
## Activity 2

What would be committed to the repository by default when using renv?

- A. All content of the `renv` folder in your project, because `renv` stores all packages there.
- B. Only `renv.lock` and `renv/activate.R`, because both are necessary to reproduce and activate the environment on another machine.
- C. Only `renv/library` and `renv/staging` directories, because `renv`, in contrast to `conda` needs to have binary packages in the repository.
- D. Only `renv.lock`, because it contains the necessary information to reproduce the environment, and most local `renv/` folders are excluded in `.gitignore`.
:::