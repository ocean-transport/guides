## Ginsburg HPC Cluster 


### Access
To gain access to the cluster, you must first get approval by emailing your UNI and account name (Ocean Climate Physics: Abernathey) to [hpc-support@columbia.edu](hpc-support@columbia.edu) while also cc'ing Ryan.

### Log In
Log in to the secure [LDEO vpn](https://ldeo-it.ldeo.columbia.edu/content/vpn-virtual-private-network) with your LDEO username and password. From your terminal, SSH into the submit node: ```ssh -x <UNI>@ginsburg.rcs.columbia.edu``` ("-x" enables X11 forwarding). Enter your Columbia password and you will be in the directory ```/burg/home/<UNI>```. 

### Start a Job with Slurm

Ginsburg uses [Slurm](https://slurm.schedmd.com/documentation.html) to manage the cluster workload. A batch script is used to allocate resources and execute your job. You must specify an account when you're ready to submit a job to the cluster. 

| Account |  Full Name  |  
|----------|-------------|
| abernathey |    Ocean Climate Physics: Abernathey   |  

The example submit script runs the job ```Date```, which prints the time and date to an output file (slurm-####.out) written in your current directory. The #'s correspond to the job ID given by Slurm. 

```bash
#!/bin/bash -l
#
# Replace ACCOUNT with your account name before submitting.
#
#SBATCH --account=ACCOUNT        # Replace ACCOUNT with your group account name
#SBATCH --job-name=Date          # The job name
#SBATCH -N 1                     # The number of nodes to request
#SBATCH -c 1                     # The number of cpu cores to use (up to 32 cores per server)
#SBATCH --time=0-0:10            # The time the job will take to run in D-HH:MM
#SBATCH --mem-per-cpu=5G         # The memory the job will use per cpu core

# Run program
date
 
# End of script
```

Once saved, scripts like this one can be submitted  to the cluster:
```bash
$ sbatch date.sh
```
To view your jobs on the system:
```bash
$ squeue -u <UNI>
```
To view information about a job:
```bash
$ scontrol show job [job ID]
```
To cancel a job:
```bash
$ scancel [job ID]
```

### Python 
An example script using anaconda:
```bash
#!/bin/sh
#
#SBATCH --account=ACCOUNT         # Replace ACCOUNT with your group account name
#SBATCH --job-name=HelloWorld     # The job name.
#SBATCH -c 1                      # The number of cpu cores to use
#SBATCH -t 0-0:30                 # Runtime in D-HH:MM
#SBATCH --mem-per-cpu=5gb         # The memory the job will use per cpu core
 
module load anaconda
 
#Command to execute Python program
python example.py
 
#End of script
```
Save the script as `helloword.sh` and submit it:
```bash
$ sbatch helloword.sh
```


### GPU 
OCP owns 4 GPU servers with priority access and the ability to run up to 5 days jobs. If the OCP GPU servers are not available, Slurm will allocate non-OCP GPU nodes.

Specify the OCP GPU partition in your submit script:
```bash
#SBATCH --partition=ocp_gpu       # Request ocp_gpu nodes first. If none are available, the scheduler will request non-OCP gpu nodes.
#SBATCH --gres=gpu:1              # Request 1 gpu (Up to 2 gpus per GPU node)

```

If you want to take advantage of the GPUs with TensorFlow, make sure you install the correct release:

```conda install tensorflow-gpu```

### Some useful modules

| Name | Version  | Module  |  
|----------|-------------|-------------|
| Anaconda Python 3.8.5 2020.11 | Python 3.8.5 | module load anaconda/3-2020.11 |  
| Anaconda Python  2.7.16 2019.10 | Python 2.7.16 | module load anaconda/2-2019.10 |  
| netcdf/gcc | 4.7.4 | netcdf/gcc/64/gcc/64/4.7.4 |

To see all available modules already installed on the cluster, run `module avail` from the command line. [More module commands](https://lmod.readthedocs.io/en/latest/010_user.html).

### Jupyter Notebooks
Jupyter notebooks run on a semi-public port that is accessible to other users logged in to a submit node on Ginsburg. Therefore, it is strongly reccomended to set up a password using the following steps:
1. load the anaconda python module
```bash
$ module load anaconda
```
3. initialize your jupyter environment
```bash
$ jupyter notebook --generate-config
```
4. start python or ipython session
```
$ ipython
```
6. generate a hashed password for web authentication. After you enter a unique password, a hashed password will be displayed. The string should be of the form ```type:salt:hashed-password```. Copy this password. 
```python
from notebook.auth import passwd; passwd()
```
8. paste the hash password into `~/.jupyter/jupyter_notebook_config.py` with the line starting with `c.NotebookApp.password =`. Make sure this line is uncommented.

**Run a Jupyter Notebook**

Start an interactive job. This example uses a time limit of one hour. 
```bash
$ srun --pty -t 0-01:00 -A <ACCOUNT> /bin/bash
$ unset XDG_RUNTIME_DIR                          # get ride of XDG_RUNTIME_DIR environment variable
$ module load anaconda                            # load anaconda
$ hostname -i                                     # This will print the IP of your interactive job node
$ jupyter notebook --no-browser --ip=<IP>         # Start the jupyter notebook with your node IP
```
**SSH port forwarding**

At this point, the port number of you notebook should be displayed. Open another connection to Ginsburg that forwards a local port to the remote node/port. For example:
```bash
$ ssh -L 8080:10.43.4.206:8888 UNI@burg.rcs.columbia.edu
```
where 8888 is the remote port number, 8080 is the local port, and 10.43.4.206 is the IP of the interactive job node.

To see the notebook, navigate to a browser on your desktop and go to *localhost:8080*. 

### Where do I save my files?
Ginsburg has a shared storage server named "burg".

| Location | Storage  | What to save  |  
|----------|-------------|-------------|
| /burg/home/<UNI> | 50 GB | small files, documents, source code, and scripts | 
| /burg/abernathey | 1 TB | large data files !NOT BACKED UP! | 


Transfer data to the cluster using SCP:
```bash
$ scp MyDataFile <UNI>@motion.rcs.columbia.edu:<DESTINATION_PATH>
```
You can access data on **Habanero** by navigating to the /rigel directory on Ginsburg's transfer or login node. 

### Still need help?
- [Ginsburg HPC Cluster User Documentation](https://confluence.columbia.edu/confluence/display/rcs/Ginsburg+HPC+Cluster+User+Documentation)
- hpc-support@columbia.edu
- [Example of running Jupyter Notebooks on Hobanero](https://rabernat.github.io/research_computing/running-jupyter-notebooks-on-habanero.html)
