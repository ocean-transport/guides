# Setting up conda and jupyterlab on remote clusters
----------------------------------------------------

A problem that lots of us have to deal with is how to run our python code on a cluster (e.g. yellowstone, discover, habanero, etc.). On our local machine or on sverdrup, we know how to manage packages using [conda](https://docs.conda.io/en/latest/) and [pip](https://pypi.org/project/pip/). But most clusters do not have conda available, and the system or module-based python distribution probably doesn't have all the packages we need (e.g. xarray, dask, etc.). How can we get around these limitations to get a fully functional, flexible python environment on any cluster?

Here is a solution, which we have used succesfully on habanero and pleiades. We assume something similar will work on most clusters.


## Step 1: Install [miniconda](https://docs.conda.io/en/latest/miniconda.html) in user space.

Miniconda is a mini version of Anaconda that includes just conda and its dependencies. It is a very small download. If you want to use python 3 (recommended) you can call

    wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh -O miniconda.sh

## Step 2: Run Miniconda3

Now you actually run miniconda to install the package manager. The trick is to specify the install directory within your home directory, rather in the default system-wide installation (which you won't have permissions to do). You then have to add this directory to your path. (Miniconda may do this automatically for you when running the bash script.)

    bash miniconda.sh -b -p $HOME/miniconda
    export PATH="$HOME/miniconda/bin:$PATH"

Make sure to check the path to miniconda is correct in the `.bashrc` file which should be in your home directory.

## Step 3: Create a custom conda environment specification

You now have to define what packages you actually want to install. A good way to do this is with a custom [conda environment file](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-from-an-environment-yml-file). The contents of this file will differ for each project. Below is the `environment.yml` that used for the [xmitgcm](https://github.com/MITgcm/xmitgcm) project.

    name: xmitgcm
    dependencies:
     - numpy
     - scipy
     - xarray
     - netcdf4
     - dask
     - jupyterlab
     - matplotlib
     - pip:
       - pytest
       - xmitgcm

Create a similar file for your project and save it as `environment.yml`. You should chose a value of `name` that makes sense for your project.

## Step 4: Create the conda environment

You should now be able to run the following command

     source .bashrc
     conda env create --file environmen.yml

This will download and install all the packages and their dependencies.

### <mark>It is occasionally the case where the security on the remote cluster will not allow one to install packages via conda. In that case, we recommend using [conda-pack](https://conda.github.io/conda-pack/) to install the environment. You can also find some tips [here](https://github.com/meom-group/tutos/blob/master/occigen/jupyter-notebook-on-occigen.md#how-to-run-a-jupyter-notebook-directly-on-occigen).</mark>

## Step 5: Activate the environments

The environment you created needs to be activated before you can actually use it. To do this, you call

     conda activate xmitgcm

(You replace `xmitgcm` with whatever name you picked in your `environment.yml` file.) This step needs to be repeated whenever you want to use the environment (i.e. every time you launch an interactive job or call python from within a batch job).

## Step 6: Boot up jupyterlab

You can launch Jupyterlab via ssh tunneling.

    - Boot up Jupyter lab without a browser: `jupyter lab --no-browser`. (You should get a message that the lab is running at `localhost:XXXX` where XXXX will be the remote port address.)
    - Now open a new Terminal window and ssh to that port using: `ssh -A -t -l <username>@gyre.ldeo.columbia.edu  -L YYYY:localhost:XXXX`. YYYY can be any available port on your local machine.
    - Open your browser and go to `localhost:YYYY`. Et voila, you have a working Jupyterlab.
    
    
## Step X: Adding packages to environment later

Sometimes we will need to add packages to the environment after it has already been created using the above steps. This can happen because we didn't know of some requirements we had early on or some new exciting packages got launched. One strategy in this case is to create a new environment, but this can often lead to a number of new environments and make things very confusing to keep track of. If possible, one can also update the environment from the environment file. This involves two steps:

    - Update the environment.yml file, adding new package names to it. 
    - run `conda env update -f environment.yml --prune` 

If you just want to update the packages that are already there and maybe new versions are available, then use `conda update --all`.
