# omero_base
ARG BASE_CONTAINER=openjdk:8
FROM $BASE_CONTAINER

ARG OMERO_VERSION=5.4.10

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
    sudo \
    locales \
    dos2unix \
    mc \
    nano \
    vim \
 && rm -rf /var/lib/apt/lists/*

RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

# Configure environment
ENV CONDA_DIR=/opt/conda \
    SHELL=/bin/bash \
    NB_USER=$NB_USER \
    NB_UID=$NB_UID \
    NB_GID=$NB_GID \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8
ENV PATH=$CONDA_DIR/bin:$PATH \
    HOME=/home/$NB_USER \
    OMERO_VERSION=$OMERO_VERSION

# Add a script that we will use to correct permissions after running certain commands
COPY fix-permissions /usr/local/bin/fix-permissions
# make build robust by removing any potential Windows CRLF chars from script
RUN dos2unix /usr/local/bin/fix-permissions && apt-get --purge remove -y dos2unix && rm -rf /var/lib/apt/lists/*
# ensure script has executable permissions
RUN chmod +x /usr/local/bin/fix-permissions

# Enable prompt color in the skeleton .bashrc before creating the default NB_USER
RUN sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc

# Create NB_USER wtih name jovyan user with UID=1000 and in the 'users' group
# and make sure these dirs are writable by the `users` group.
RUN echo "auth requisite pam_deny.so" >> /etc/pam.d/su && \
    sed -i.bak -e 's/^%admin/#%admin/' /etc/sudoers && \
    sed -i.bak -e 's/^%sudo/#%sudo/' /etc/sudoers && \
    useradd -m -s /bin/bash -N -u $NB_UID $NB_USER && \
    mkdir -p $CONDA_DIR && \
    chown $NB_USER:$NB_GID $CONDA_DIR && \
    chmod g+w /etc/passwd && \
    fix-permissions $HOME && \
    fix-permissions "$(dirname $CONDA_DIR)"

# requires NPM?
RUN curl -sL https://deb.nodesource.com/setup_10.x | bash -
RUN apt-get install -y nodejs

# Create Symbolic Links to libcrypto and libssl libraries so that OMERO client works
RUN ln -s /usr/lib/x86_64-linux-gnu/libssl.so.1.0.2 /usr/lib/x86_64-linux-gnu/libssl.so.1.0.0
RUN ln -s /usr/lib/x86_64-linux-gnu/libcrypto.so.1.0.2 /usr/lib/x86_64-linux-gnu/libcrypto.so.1.0.0

USER $NB_UID
WORKDIR $HOME

# Setup work directory for backward-compatibility
RUN mkdir /home/$NB_USER/base

# Fix permissions
RUN fix-permissions /home/$NB_USER

# Install conda as jovyan and check the md5 sum provided on the download site
ENV MINICONDA_VERSION=4.6.14 \
    CONDA_VERSION=4.7.10

RUN cd /tmp && \
    wget --quiet https://repo.continuum.io/miniconda/Miniconda2-${MINICONDA_VERSION}-Linux-x86_64.sh && \
    echo "faa7cb0b0c8986ac3cacdbbd00fe4168 *Miniconda2-${MINICONDA_VERSION}-Linux-x86_64.sh" | md5sum -c - && \
    /bin/bash Miniconda2-${MINICONDA_VERSION}-Linux-x86_64.sh -f -b -p $CONDA_DIR && \
    rm Miniconda2-${MINICONDA_VERSION}-Linux-x86_64.sh && \
    echo "conda ${CONDA_VERSION}" >> $CONDA_DIR/conda-meta/pinned && \
    $CONDA_DIR/bin/conda config --system --prepend channels conda-forge && \
    $CONDA_DIR/bin/conda config --system --prepend channels bioconda && \
    $CONDA_DIR/bin/conda config --system --set auto_update_conda false && \
    $CONDA_DIR/bin/conda config --system --set show_channel_urls true && \
    $CONDA_DIR/bin/conda install --quiet --yes conda && \
    $CONDA_DIR/bin/conda update --all --quiet --yes && \
    conda list python | grep '^python ' | tr -s ' ' | cut -d '.' -f 1,2 | sed 's/$/.*/' >> $CONDA_DIR/conda-meta/pinned && \
    conda clean --all -f -y && \
    rm -rf /home/$NB_USER/.cache/yarn && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Install OMERO Python libs
# Cleanup temporary files
# Correct permissions
# Do all this in a single RUN command to avoid duplicating all of the
# files across image layers when the permissions change
RUN conda install --quiet --yes \
    "python-omero=${OMERO_VERSION}" && \
    # 'openssl=1.0.*' && \
    conda clean --all -f -y && \
    npm cache clean --force && \
    rm -rf /home/$NB_USER/.cache/yarn && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

USER root

# Found we have to install the old SSL and crypto libraries, probably due to OpenJDK base image update
RUN cd /tmp && \
    wget --quiet http://nl.archive.ubuntu.com/ubuntu/pool/main/g/glibc/libc6-udeb_2.27-3ubuntu1_amd64.udeb && \
    wget --quiet http://nl.archive.ubuntu.com/ubuntu/pool/main/o/openssl/libcrypto1.0.0-udeb_1.0.2g-1ubuntu4_amd64.udeb && \
    wget --quiet http://nl.archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.0.0-udeb_1.0.2g-1ubuntu4_amd64.udeb && \
    echo "52e0bb2c1f552da8b7120ff3d6a2b38a *libc6-udeb_2.27-3ubuntu1_amd64.udeb" && \
    echo "d8c283d6b2b587c6b58e163de75a7f66 *libcrypto1.0.0-udeb_1.0.2g-1ubuntu4_amd64.udeb" && \
    echo "83633d8dc2c32363914282ac32087d7e *libssl1.0.0-udeb_1.0.2g-1ubuntu4_amd64.udeb" && \
    dpkg -i libc6-udeb_2.27-3ubuntu1_amd64.udeb && \
    dpkg -i libcrypto1.0.0-udeb_1.0.2g-1ubuntu4_amd64.udeb && \
    dpkg -i libssl1.0.0-udeb_1.0.2g-1ubuntu4_amd64.udeb

USER $NB_UID
WORKDIR $HOME
