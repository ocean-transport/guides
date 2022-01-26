Usually when installing new environments on clusters, the environment name automatically shows up on 
the available kernels. However, sometimes it does not. In situations it doesn't, some extra steps need to be taken, 
as noted below. Step 1-2 are the standard steps, then the magin happens. 

That's some voodoo-magic right there. The method actually works quite well --

(replace instances of <environment> with your preferred name)

1. Create your conda environment at the command line. 

conda create --name <environment> <module> ... <moduleN>

For this to work I needed the following modules:

$ conda create --name <environment> ipython jupyter

2. Prepare your environment at the command line (varies by your default shell).

Bash:

$ source activate <environment>

T/csh:

$ setenv PATH ~$USER/.conda/envs/<environment>/bin:$PATH

Make sure ipython is pointing to your newly created conda environment.

3. Create a custom kernel

$ jupyter kernelspec install-self --user

4. Rename the python kernel you created. This way it does not override the default python2.7 kernel.

$ cd ~/.local/share/jupyter/kernels
$ vim <environment>/kernel.json

Change the display_name:

{
- "display_name": "Python 2",
+ "display_name": "Python 2 (<environment>)",
 "language": "python",
 "argv": [
  "/home/jsattelb/.conda/envs/<environment>/bin/python",
  "-m",
  "ipykernel",
  "-f",
  "{connection_file}"
 ]
}

Then rename the directory, as well...

$ mv python2 <environment>

The next time you refresh your JupyterHub instance, you'll see your custom environment under New.
