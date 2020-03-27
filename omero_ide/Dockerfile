# omero_ide

ARG PYOMERO_UPLOAD_VERSION=v5.6.2_2.0.0
ARG BASE_CONTAINER=biordm/omero-connect:omero_jupyter-$PYOMERO_UPLOAD_VERSION
FROM $BASE_CONTAINER
# FROM openjdk:11

LABEL maintainer="SBS Research Data Management <bio_rdm@ed.ac.uk>"
ARG NB_USER="jovyan"
ARG NB_UID="1000"
ARG NB_GID="100"

USER root

# ttf-dejavu is required to render GUI under X11: https://github.com/docker-library/openjdk/issues/73
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update && apt-get -yq dist-upgrade \
 && apt-get install -yq --no-install-recommends \
    libx11-6 \
    x11-apps \
    libxext-dev \
    libxtst-dev \
    libxrender-dev \
    ttf-dejavu \
    openssh-server \
    vim \
    xorg \
    openbox \
 && rm -rf /var/lib/apt/lists/*

# RUN echo "jovyan" | passwd --stdin jovyan
RUN echo "jovyan:jovyan" | chpasswd
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -i 's/#AddressFamily any/AddressFamily inet/' /etc/ssh/sshd_config
# RUN service ssh restart

# Configure container startup
# ENTRYPOINT ["tini", "-g", "--"]
# CMD ["service ssh restart"]
# CMD ["start-notebook.sh"]

EXPOSE 22
# CMD ["/etc/init.d/ssh", "start"]
COPY start.sh /usr/bin/start.sh
ENTRYPOINT ["tini", "-g", "--"]
CMD ["sh", "/usr/bin/start.sh"]

RUN ln -s /opt/pycharm-2019.2.4/bin/pycharm.sh /usr/bin/pycharm

# install VS Code and dependencies
RUN wget https://github.com/VSCodium/vscodium/releases/download/1.41.1/codium_1.41.1-1576787344_amd64.deb
RUN apt-get update && apt-get install -yq --no-install-recommends \
    libnotify4 \
    libnss3 \
    libxkbfile1 \
    libgconf-2-4 \
    libsecret-1-0 \
    libgtk-3-0 \
    libxss1
RUN dpkg -i codium_1.41.1-1576787344_amd64.deb
# RUN apt-get update && apt-get install -yq --no-install-recommends \
    # gdebi-core \
    # apt-utils
# RUN gdebi codium_1.41.1-1576787344_amd64.deb
RUN apt-get -yq update && apt-get -yq install -f && apt-get -yq install libasound2 \
    && rm -rf /var/lib/apt/lists/*

USER $NB_UID
WORKDIR $HOME

# Install code extensions
RUN codium --install-extension ms-python.python \                                                                                                           && codium --install-extension tht13.python \
    && codium --install-extension donjayamanne.python-extension-pack \
    && codium --install-extension himanoa.python-autopep8

# install Pycharm
RUN wget -O /tmp/pycharm-2019.2.4.tar.gz https://download.jetbrains.com/python/pycharm-community-2019.2.4.tar.gz \
    && mkdir -p /opt/pycharm-2019.2.4 \
    && tar -xf /tmp/pycharm-2019.2.4.tar.gz --strip-components=1 -C /opt/pycharm-2019.2.4 \
    && rm /tmp/pycharm-2019.2.4.tar.gz

# Run Pycharm once to initialise the files
RUN pycharm &
RUN sleep 30

# Update Pycharm config to fix startup errors and allocate enough memory
RUN sed -i 's/-XX:+UseConcMarkSweepGC/#-XX:+UseConcMarkSweepGC/' /opt/pycharm-2019.2.4/bin/pycharm.vmoptions
RUN sed -i 's/-XX:+UseConcMarkSweepGC/#-XX:+UseConcMarkSweepGC/' /opt/pycharm-2019.2.4/bin/pycharm64.vmoptions
RUN sed -i 's/-Xmx[0-9]*m/-Xmx2048m/' /opt/pycharm-2019.2.4/bin/pycharm.vmoptions
RUN sed -i 's/-Xmx[0-9]*m/-Xmx2048m/' /opt/pycharm-2019.2.4/bin/pycharm64.vmoptions

# USER root

# clone the repo with the PyUploader code
RUN mkdir -p /home/$NB_USER/work
RUN git clone --single-branch --branch master https://github.com/SynthSys/pyOmeroUpload /home/$NB_USER/work/pyOmeroUpload

# Use pip to install the extra dependencies, since Conda updates GCC and breaks OMERO
# RUN pip install -r //pyOmeroUpload/requirements.txt && \
RUN conda install --quiet --yes --file /home/$NB_USER/work/pyOmeroUpload/requirements.txt && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Switch back to jovyan to avoid accidental container runs as root
USER $NB_UID
