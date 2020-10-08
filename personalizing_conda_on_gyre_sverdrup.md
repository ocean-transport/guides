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
    # Needed to ensure gyre sets the right environment
  - _libgcc_mutex=0.1=main
  - backcall=0.2.0=py_0
  - ca-certificates=2020.7.22=0
  - certifi=2020.6.20=py37_0
  - decorator=4.4.2=py_0
  - ipykernel=5.3.4=py37h5ca1d4c_0
  - ipython=7.18.1=py37h5ca1d4c_0
  - ipython_genutils=0.2.0=py37_0
  - jedi=0.17.2=py37_0
  - jupyter_client=6.1.7=py_0
  - jupyter_core=4.6.3=py37_0
  - ld_impl_linux-64=2.33.1=h53a641e_7
  - libedit=3.1.20191231=h14c3975_1
  - libffi=3.3=he6710b0_2
  - libgcc-ng=9.1.0=hdf63c60_0
  - libsodium=1.0.18=h7b6447c_0
  - libstdcxx-ng=9.1.0=hdf63c60_0
  - ncurses=6.2=he6710b0_1
  - openssl=1.1.1h=h7b6447c_0
  - parso=0.7.0=py_0
  - pexpect=4.8.0=py37_1
  - pickleshare=0.7.5=py37_1001
  - pip=20.2.2=py37_0
  - prompt-toolkit=3.0.7=py_0
  - ptyprocess=0.6.0=py37_0
  - pygments=2.7.1=py_0
  - python=3.7.9=h7579374_0
  - python-dateutil=2.8.1=py_0
  - pyzmq=19.0.2=py37he6710b0_1
  - readline=8.0=h7b6447c_0
  - setuptools=49.6.0=py37_1
  - six=1.15.0=py_0
  - sqlite=3.33.0=h62c20be_0
  - tk=8.6.10=hbc83047_0
  - tornado=6.0.4=py37h7b6447c_1
  - traitlets=5.0.4=py_0
  - wcwidth=0.2.5=py_0
  - wheel=0.35.1=py_0
  - xz=5.2.5=h7b6447c_0
  - zeromq=4.3.2=he6710b0_3
  - zlib=1.2.11=h7b6447c_3
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
After you have created your own environment, you should be able to access it as kernel in your jupyter lab. It will show up on your launcher:
![](https://i.imgur.com/v64ppy2.png)

In the case conficts emerge with your local and system conda, you can launch Jupyterlab via ssh tunneling. (I no longer have access to Gyre so if someone could check if the following procedures work on it, that's be swell!!)

 - Install miniconda
 - Activate your local miniconda via `source .bashrc`
 - Create your conda environment and activate it (`conda activate users_env`)
 - Boot up Jupyter lab without a browser: `jupyter lab --no-browser`. (You should get a message that the lab is running at `localhost:XXXX` where XXXX will be the port address)
 - Now open a new Terminal window and ssh to that port using: `ssh -A -t -l <username>@gyre.ldeo.columbia.edu  -L YYYY:localhost:XXXX`. YYYY can be any available port on your local laptop.
 - Open your browser and go to `localhost:YYYY`. Et voila, you have a working Jupyterlab.

**Deleting Old Environments**
You can delete you old environment by following:
1.


## FAQs
- I am having trouble with opening and running notebooks, because I believe I have had my jupyter lab instance open for too long, and errors have cropped up. Can I restart it?
    - Yes you can. You will need to go to the `Commands` tab on the left of the jupyter lab, then open `Launch Classic Notebook`. This will open a new window, where you should see a button that says `Control Panel` on the left. You can click it, followed by clicking `Stop My Server`. Now if you logout and login again, you will be on a fresh jupyter lab instance which will be working with the latest libraries on gyre in a fresh instance.
- How can I look at the resource usage on Gyre?
    - You can get an overview of resource usage using the netdata dashboard that can be accessed at: http://gyre.ldeo.columbia.edu:19999/
