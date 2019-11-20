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

If you also wish to mount a directory of Jupyter notebooks, for example the notebooks from this Git project:
```
docker run --name omero-jupyter -p 8888:8888 -v /E/projects/Omero_data:/var/data/omero_data -v /E/projects/OMEROConnect/omero_jupyter/notebooks:/home/jovyan/work/connect_notebooks omero_jupyter
```
*N.B.* Remember if you are running these containers on a Windows host, you must enable 'Shared Drives' for the relevant hard drive in the Docker Desktop application settings. You may also have to enable incoming connections in the Windows Firewall or open up access to port 445, according to the instructions at https://docs.docker.com/docker-for-windows/#firewall-rules-for-shared-drives. For initialising the Docker container, it may be necessary to use a Windows-style file path, e.g.:
```
docker run --name omero-jupyter -p 8888:8888 -v D:\projects\OMEROConnect\omero_jupyter\notebooks:/home/jovyan/work/query_notebooks:rw omero_jupyter
```

Once the Docker container runs, it will report the URL for access to the command line output, including the Jupyter server access token. If you need to restart the Jupyter Docker container, keep a note of this token because it will not be displayed again on container restart; alternatively, run `docker exec -it omero-jupyter /bin/bash` to ssh into the running container and then run `jupyter notebook list` to retrieve the URL and token. The Jupyter notebook can be accessed through the browser on your host machine via the URL `http://127.0.0.1:8888/?token=[AUTH_TOKEN]`.

