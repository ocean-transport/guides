
Here we document a good way to run jupyter lab with the help of singularity on the ginsburg cluster(https://confluence.columbia.edu/confluence/display/rcs/Ginsburg+HPC+Cluster+User+Documentation). 

Roughly follow the notes at: https://hackmd.io/RSgC2eaQS0CXdSfEthKSLg 

# Setup singularity 

You may download this image in your scratch, as it might be large in size. 

```
module load singularity
singularity pull notebook_pangeo.sif docker://pangeo/pangeo-notebook
```

# Start interactive node

```
srun --pty -t 0-03:00 -A abernathey /bin/bash
```

# Start singularity session

```
singularity shell notebook_pangeo.sif
```

# Run jupyter lab 
Setup jupyter notebooks using: https://confluence.columbia.edu/confluence/display/rcs/Ginsburg+-+Job+Examples#GinsburgJobExamples-JupyterNotebooks 

```
unset XDG_RUNTIME_DIR 

hostname -i
# gives the ip that should be used in next line. 

jupyter lab --no-browser --ip=10.43.4.206
```


# SSH tunnel 

```
ssh -L 8080:10.43.4.206:8888 UNI@burg.rcs.columbia.edu (or just say gins if already setup tunnel.)
```

Then in a broswer go to `localhost:8080`. 

