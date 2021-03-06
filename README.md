******************************************************************************
**Updated 12/07/21: This repository must be manually synchronised to the 'docker'
sub-directory of the [OMERO Toolkit repository](https://github.com/SynthSys/omero-toolkit)
whenever updates are made.**
******************************************************************************

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
docker run --name omero-jupyter -p 8888:8888 -v /E/projects/Omero_data:/var/data/omero_data -v /E/projects/omero_connect_demo/notebooks:/home/jovyan/work/connect_notebooks omero_jupyter
```
*N.B.* Remember if you are running these containers on a Windows host, you must enable 'Shared Drives' for the relevant hard drive in the Docker Desktop application settings. You may also have to enable incoming connections in the Windows Firewall or open up access to port 445, according to the instructions at https://docs.docker.com/docker-for-windows/#firewall-rules-for-shared-drives. For initialising the Docker container, it may be necessary to use a Windows-style file path, e.g.:
```
docker run --name omero-jupyter -p 8888:8888 -v 'D:\projects\omero_connect_demo\notebooks:/home/jovyan/work/query_notebooks:rw' omero_jupyter
```

Once the Docker container runs, it will report the URL for access to the command line output, including the Jupyter server access token. If you need to restart the Jupyter Docker container, keep a note of this token because it will not be displayed again on container restart; alternatively, run `docker exec -it omero-jupyter /bin/bash` to ssh into the running container and then run `jupyter notebook list` to retrieve the URL and token. The Jupyter notebook can be accessed through the browser on your host machine via the URL `http://127.0.0.1:8888/?token=[AUTH_TOKEN]`.

# DockerHub Images
All of the images here are pre-built and available in the DockerHub image registry at [https://hub.docker.com/r/biordm/omero-connect](https://hub.docker.com/r/biordm/omero-connect). Usage instructions are found below.

## Installation
Each image can be installed or updated using the Docker pull command, for instance:
```
docker pull biordm/omero-connect:omero_uploader
docker pull biordm/omero-connect:omero_jupyter
```
These pull commands can be executed in any order, since the child images automatically pull any required parent images.

## Operation
Once the OMEROConnect images are downloaded to the local Docker image repository, they can be run with slight modifications to the commands given previously, for instance:
```
docker run -t -d --name omero-uploader -v /E/projects/Omero_data:/var/data/omero_data --entrypoint /bin/bash biordm/omero-connect:omero_uploader
```
```
docker run --name omero-jupyter -p 8888:8888 -v 'D:\projects\omero_connect_demo\notebooks:/home/jovyan/work/query_notebooks:rw' omero_jupyter
```
To access a `bash` terminal in the resulting Docker containers, run commands such as:
```
docker exec -it omero-uploader /bin/bash
```

# OMERO IDE image
The IDE image is useful if you are developing with the OMERO Python library, and perhaps you wish to modify the PyOmeroUpload code itself. The image contains the Pycharm and Codium IDEs and uses X11 forwarding so that you can develop code in the GUI as if it was running natively; this is helpful for isolating your development environment from your host system, especially if your host systems is Windows.

cd ../omero_ide

docker build --tag omero_ide .

docker run -it -u root --name omero-ide -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=localhost:0 -p 2222:22 omero_ide

When a container is instantiated and run, the SSH server is automatically started and you can use an X11-enabled client application (such as MobaXTerm or XMing) or simply `ssh -X jovyan@localhost -p 2222` to access the container; the password for the `jovyan` user is in the Dockerfile. Once inside the container, run `pycharm &` to launch the Pycharm IDE. Alternatively, run the Codium (open source version of Microsoft VS Code) IDE with the command `codium --disable-gpu`. If the container is stopped, simply running `docker start omero-ide` again will restart it and you can resume development (and access any files you have been working on). *N.B.* removing the container will delete any files you have been working on; either mount your code in a rw directory on your host or sync the files you are working on regularly.

To mount a host folder on the guest file system in Windows:

docker run -it -u root --name omero-ide -v /tmp/.X11-unix:/tmp/.X11-unix -v '/C/Users/J Bloggs/Documents/code_projects/omero_connect_demo/notebooks:/home/jovyan/work/query_notebooks:rw' -v '/C/Users/J Bloggs/Documents/code_projects/pyOmeroUpload:/home/jovyan/work/pyOmeroUpload:rw' -e DISPLAY=localhost:0 -p 2222:22 omero_ide

**N.B.** In the event that the Windows Firewall settings are configured to 'Block all incoming connections', that must be disabled and then the drive must be un-shared in the Docker Desktop settings, applied, then shared again and applied. Otherwise, the directory will appear to be mounted but none of the sub-directories and files will be available in the guest.

## DockerHub Image
As with the other images, the IDE image can be installed with the following command:
```
docker pull biordm/omero-connect:omero_ide
```
It can be run with a slightly modified command from before:
```
docker run -it -u root --name omero-ide -v /tmp/.X11-unix:/tmp/.X11-unix -v '/C/Users/J Bloggs/Documents/code_projects/omero_connect_demo/notebooks:/home/jovyan/work/query_notebooks:rw' -v '/C/Users/J Bloggs/Documents/code_projects/pyOmeroUpload:/home/jovyan/work/pyOmeroUpload:rw' -e DISPLAY=localhost:0 -p 2222:22 biordm/omero-connect:omero_ide
```
