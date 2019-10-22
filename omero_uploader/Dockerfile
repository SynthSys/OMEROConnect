# omero_uploader

ARG BASE_CONTAINER=omero_base
FROM $BASE_CONTAINER

LABEL maintainer="SBS Research Data Management <bio_rdm@ed.ac.uk>"
ARG NB_USER="jovyan"
ARG NB_UID="1000"
ARG NB_GID="100"

USER root

# Install all OS dependencies for notebook server that starts but lacks all
# features (e.g., download as all possible file formats)
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update && apt-get -yq dist-upgrade \
 && apt-get install -yq --no-install-recommends \
    git \
 && rm -rf /var/lib/apt/lists/*

USER $NB_UID
WORKDIR $HOME

# clone the repo with the PyUploader code
RUN git clone --single-branch --branch master https://github.com/SynthSys/pyOmeroUpload /opt/pyOmeroUpload

# Use pip to install the extra dependencies, since Conda updates GCC and breaks OMERO
RUN pip install -r /opt/pyOmeroUpload/requirements.txt && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER
