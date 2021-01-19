# Some tips to configure conda


## When conda files blow up your home directory quota
On HPC platforms, your home directory often has quite limited space, which can quickly be filled by conda related files. To free up space you can use 
```
$ conda clean --all
```
to delete temporary files. A better longterm solution is to tell conda to install the environment and package files in another location (this will depend on your setup).
To do this, you can edit `.condarc` in your root directory (or create it if it does not exist yet):

```yaml
envs_dir:
   - /path/to/a/large/dir/conda/envs
pkgs_dirs:
  - /path/to/a/large/dir/conda/pkgs
```
This will install all those large files into this location from now on.
