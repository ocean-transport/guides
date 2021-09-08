# How to set up a Google Cloud Storage Bucket

In this little guide I will describe how to set up a project specific bucket in google cloud storage. 

For this you need to either have your own account or be invited to the pangeo project (like I was in this example).

## Setting up the Bucket
Open the [Google Cloud Console](https://console.cloud.google.com/) and navigate to the `Storage` tab.

Now you can create a bucket. The form that opens up will ask you a few things. You have to choose a unique name for the bucket first. Then you can decide where to store the data. Since the pangeo cloud instances run on `us-central1` I opted for the `Region` option and the selected `us-central1` under the `Location` drop-down.

Leave all other settings as the defaults.

## Create Service Account
Go to `IAM & Admin` and create a new service account. For now you only need to give it a name and a description. Leave the other things like they are. Copy the generated email, which will be something like `name@pangeo-181919.iam.gserviceaccount.com`

Now navigate back to your bucket, click the `Permissions` tab, add a member and past the email from above. Choose `Storage Admin` (you might have to search for it) as the Role.

The final step is to create a key for the service account. So return to the service account you just created, navigate to the `Keys` tab, click `Add Key` and choose `JSON` as option. This will download the key file to your computer.

## Using the bucket

You can upload the key via drag and drop to your jupyterlab interface and then use it in your notebooks.
> I am not sure this is a secure best practice, this will only affect the data within this bucket, so I will not worry too much about it. If you know a better way to do this, please let me know.

## Instantiate a filesystem with your credentials

```python
import json
import gcsfs
with open('/home/jovyan/keys/<keyname>.json') as token_file:
    token = json.load(token_file)
fs = gcsfs.GCSFileSystem(token=token)
```

As anonymous

```python
fs = gcsfs.GCSFileSystem(anon=True)
```

AWS has its own variation for this.

```python
import s3fs

fs = s3fs.S3FileSystem(key=, secret=)

```

### Working with the filesystem (e.g., Writing/Reading with xarray)
This should give you an idea about how to read and write to the bucket using xarray. Zarr is preferred when working on the cloud.  

```python
import xarray as xr
import numpy as np
import tempfile

da = xr.DataArray(np.random.rand(4)).to_dataset(name='data')

# Write/Read Zarr
mapper = gcs.get_mapper('<bucketname>/something.zarr')
da.to_zarr(mapper, mode='w')
da_zarr = xr.open_zarr(mapper)

# Write/Read other data objects
f = NamedTemporaryFile()
da.to_netcdf(f.name) # this could be any object, this example uses netCDF
fs.put(f.name, '<bucketname>/something.nc') # same as fs.upload(...)
open_f = fs.open('<bucketname>/something.nc')
ds = xr.open_dataset(open_f)
```