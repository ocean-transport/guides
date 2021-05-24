# How to start working in the cloud

Alright, time to see what all the hype is about and start some science in the cloud. 

![](https://media.giphy.com/media/hk6czgfmwVJS0/giphy.gif)

## Sign up

For both the [pangeo cloud](https://pangeo.io/cloud.html#sign-up) and [coiled](https://cloud.coiled.io/login?redirect_uri=/) you have to sign in. Its free to do so, so just sign up for both right now.

## Frontend (notebook) in the cloud

You will likely use pangeo cloud as your 'frontend', meaning the spot in the cloud where your jupyter notebook is running. You currently have the choice between two different deployments on Google Cloud or AWS. 
Each of these has a 'staging' and a 'production' deployment, which differ in the versions that are available in the environment ('staging' has the more recent versions').


Each of these has a 'staging' and a 'production' deployment, which differ in the versions that are available in the environment ('staging' has the more recent versions').

You can choose your deployment on the [pangeo cloud federation repo](https://github.com/pangeo-data/pangeo-cloud-federation#clusters). 

Choosing a cloud provider usually is a question of where your data lives. You always want to 'bring your compute to the data' so match the deployment to your dataset location of choice.

> For coiled you currently should use AWS only!

Once you decided you will have to start a `server`
<img width="1366" alt="image" src="https://user-images.githubusercontent.com/14314623/119416244-393de600-bcc1-11eb-8403-433f406ff039.png">

Most of the time you want to choose the server with the `pangeo-Notebook` installed 
> For the google cloud deployments, you will be able to choose different resources for your cloud server
Its usually faster to get a small or medium server (and since we will get the computing power via dask below), those are often enough here.


## Getting that Dask OOOMPF

So the above will usually give you a node with 2-4 cores and few GB of RAM. But obviously we want 

![more](https://media.giphy.com/media/1jXGsHY2EKdL27mEMd/giphy.gif)

You can currently choose between two ways of getting a dask cluster: 

1) Via pangeo cloud:
  - Super easy to [set up](https://pangeo.io/cloud.html#dask) 
  ```python
  from dask_gateway import GatewayCluster

cluster = GatewayCluster()
cluster.adapt(minimum=2, maximum=10)  # or cluster.scale(n) to a fixed size.
client = cluster.get_client()
client
  ```
  - Hard to change dependencies on the dask workers (see [here](https://pangeo.io/cloud.html#dask-software-environment) for some workarounds)
  - Costs pangeo money

2) Via coiled: 
  - Just slightly more complicated setup:
```python
import coiled
from dask.distributed import Client
# make sure to match the 
coiled.create_software_environment(
    name='pangeo-cloud',
    container='pangeo/pangeo-notebook',   # matches Pangeo Cloud AWS staging cluster with latest image
    # container='pangeo/pangeo-notebook:2021.05.04',   # matches Pangeo Cloud AWS staging cluster
    # container='pangeo/pangeo-notebook:2021.05.04',   # matches Pangeo Cloud AWS production cluster
)
cluster = coiled.Cluster(software="pangeo-cloud")
client = Client(cluster)
client



```
  - Limited to AWS at the moment
  - It is super easy to set up custom environment on dask workers [example](https://github.com/jbusecke/cmip6_derived_cloud_datasets/blob/main/first_try.ipynb).
  - We get free credits with our consulting contract 
  <img src="https://user-images.githubusercontent.com/14314623/119415450-a2bcf500-bcbf-11eb-8593-5069afe75498.png" alt="drawing" width="200"/>



## More info/useful links

[Coiled Docs](https://docs.coiled.io/user_guide/index.html)

[Pangeo Cloud Docs](https://pangeo.io/cloud.html#)
