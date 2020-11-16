# Gyre Conda Environments

You can access gyre after going through the [LDEO vpn](https://ldeo-it.ldeo.columbia.edu/content/vpn-virtual-private-network), and then 
accessing via terminal:
```ssh -x <username>@gyre.ldeo.columbia.edu```
 or accessing the jupyter lab server directly:
 ```https://gyre.ldeo.columbia.edu/```. 

This will bring you to a place where you have access to the a system wide python (`/usr/local/anaconda/bin/python`) and conda (`/usr/local/anaconda/bin/conda`), to install modules. 

However, often when using gyre or sverdrup, the user will want to install python modules of their own choice and manage their own environments for their projects. 

**Step by step to create your own environment**:
1. Create conda environment at command line 
`conda env create -f environment.yml` 
where the `environment.yml` file is user defined, an example that works on gyre is shown below. It is ok if you don't get all your requirements at this point, you can install them later using `conda install <package_name> -c conda-forge` or `pip install <package_name>` or directly by getting the binaries from github or other location. 
This step should run successfully and when it finishes give you instructions on how to activate and deactivate this conda environment. And this is all that you need to do. The environment will show up as an option on your jupyter lab called `Python [Conda env:user_env]` to open a new notebook or terminal in. 

Example `envrionment.yml` file:
```
name: users_env # can choose any name
channels:
  - defaults
dependencies:
  # These packages are required if you want your environment to show up in jupyterlab
  - python
  -ipywidgets
  # Commonly used packaged by oceanographers
  - xarray
  - numpy
  - scipy
  - dask
  - netcdf4
  - pandas
  - matplotlib
  - xgcm
  - xmitgcm
```

2. You can activate your environment by `conda activate users_env`. You will need to do this before you can install anything else in your own environment. 

This is all you need to do. Now you can open notebooks that have access to this environment, as discussed next.

**Opening a notebook with your environment**
After you have created your own environment, you should be able to access it as kernel in your jupyter lab. You can do this by opening a notebook and then selecting your new Kernel from the dropdown in the top right. Your environment may take a minute or so to propagate to this dropdown after you install it. 

**Deleting Old Environments**
You can delete you old environment by following: 
1. 


## FAQs
- I am having trouble with opening and running notebooks, because I believe I have had my jupyter lab instance open for too long, and errors have cropped up. Can I restart it? 
    - Yes you can. You will need to go to the `Commands` tab on the left of the jupyter lab, then open `Launch Classic Notebook`. This will open a new window, where you should see a button that says `Control Panel` on the left. You can click it, followed by clicking `Stop My Server`. Now if you logout and login again, you will be on a fresh jupyter lab instance which will be working with the latest libraries on gyre in a fresh instance.
- How can I look at the resource usage on Gyre? 
    - You can get an overview of resource usage using the netdata dashboard that can be accessed at: http://gyre.ldeo.columbia.edu:19999/
