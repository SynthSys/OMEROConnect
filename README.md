# OMEROConnect

OMEROConnect is a Python toolkit designed to support uploading to and querying OMERO servers, the Open Microscopy Environment image data repository platform.

## Toolkit Structure
The toolkit is intended to be deployed as Docker containers. There are three container images: `omero_base`, `omero_uploader` and `omero_jupyter`. The Jupyter Docker container image inherits from the uploader image which in turn inherits from the base image.

## Building and Running the Toolkit
Once the repository has been cloned, each Docker image must be built in turn in the following order. To build and run the `omero_base` image:
```
# OMERO Base image
cd omero_base

docker build --tag omero_base .

# run in both detached (-d) and foreground (-t) mode
docker run -t -d --name omero-base --entrypoint /bin/bash omero_base
```
Next, build the uploader image and run the container with a local directory containing microscopy images mounted:
```
# OMERO Uploader image
cd ../omero_uploader

docker build --tag omero_uploader .

# run in both detached (-d) and foreground (-t) mode
docker run -t -d --name omero-uploader -v /E/projects/Omero_data:/var/data/omero_data --entrypoint /bin/bash omero_uploader
```
Finally, build and run the Jupyter image with the same mount point in the running container:
```
# OMERO Jupyter image
cd ../omero_jupyter

docker build --tag omero_jupyter .

docker run --name omero-jupyter -p 8888:8888 -v /E/projects/Omero_data:/var/data/omero_data omero_jupyter
```
