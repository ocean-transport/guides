## How to checkout a Pull Request locally and test it

### An even easier method
Within your desired environment you can just do:
```
pip install git+https://github.com/<user>/<repo>.git@refs/pull/<123>/head
```


[source](https://stackoverflow.com/questions/13561618/pip-how-to-install-a-git-pull-request)

Values in `<>` need to be filled in by the user. For the xgcm case this would be:

```
pip install git+https://github.com/xgcm/xgcm.git@refs/pull/205/head

```


> This enables you to quickly test a new feature. If you want to make modifications and push back to github, I recommend the 'full' method below.

### The manual method

First navigate to wherever you store your code and in that directory clone the repository in question, e.g.
```
git clone git@github.com:xgcm/xgcm.git
```
And then navigate to the new folder (in this case it will be named `xgcm`)

To check out a specific PR I follow [these](https://docs.github.com/en/enterprise/2.15/user/articles/checking-out-pull-requests-locally) instructions. 

```
git fetch origin pull/ID/head:BRANCHNAME
git checkout BRANCHNAME
```
The `ID` needs to match the PR number (here 205) and `BRANCHNAME` can you any name you want for the new branch

This should work:
```
git fetch origin pull/205/head:testing_new_feature

git checkout testing_new_feature
```

Now you have to install the package and you are ready to go. Depending on how you manage your packages, this means activating a specific conda environment (where you want to install) and then within the `xgcm` folder do 
```
pip install -e .
```

And that should be it!

![](https://media.giphy.com/media/CjmvTCZf2U3p09Cn0h/giphy.gif)


