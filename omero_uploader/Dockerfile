# omero_uploader
ARG PYOMERO_UPLOAD_VERSION=5.4.10_1.2.0

ARG BASE_CONTAINER=biordm/omero-connect:omero_base
FROM $BASE_CONTAINER

LABEL maintainer="SBS Research Data Management <bio_rdm@ed.ac.uk>"
ARG NB_USER="jovyan"
ARG NB_UID="1000"
ARG NB_GID="100"

USER root

# Install all OS dependencies for notebook server that starts but lacks all
# features (e.g., download as all possible file formats)
ENV DEBIAN_FRONTEND noninteractive
ENV PYOMERO_UPLOAD_VERSION=$PYOMERO_UPLOAD_VERSION

RUN apt-get update && apt-get -yq dist-upgrade \
 && apt-get install -yq --no-install-recommends \
    git \
 && rm -rf /var/lib/apt/lists/*

USER $NB_UID
WORKDIR $HOME

# clone the repo with the PyUploader code
# RUN git clone --single-branch --branch master https://github.com/SynthSys/pyOmeroUpload /opt/pyOmeroUpload

# Use pip to install the extra dependencies, since Conda updates GCC and breaks OMERO
# RUN pip install -r /opt/pyOmeroUpload/requirements.txt && \
#     fix-permissions $CONDA_DIR && \
#     fix-permissions /home/$NB_USER
RUN conda install --quiet --yes \
    "pyomero-upload=${PYOMERO_UPLOAD_VERSION}" && \
    conda clean --all -f -y && \
    npm cache clean --force && \
    rm -rf /home/$NB_USER/.cache/yarn && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER
