# omero_ide

ARG BASE_CONTAINER=omero_jupyter
FROM $BASE_CONTAINER

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
RUN service ssh restart

# Configure container startup
# ENTRYPOINT ["tini", "-g", "--"]
# CMD ["service ssh restart"]
# CMD ["start-notebook.sh"]

USER $NB_UID
WORKDIR $HOME

# install Pycharm
RUN wget -O /tmp/pycharm-2019.2.4.tar.gz https://download.jetbrains.com/python/pycharm-community-2019.2.4.tar.gz \
    && mkdir -p /opt/pycharm-2019.2.4 \
    && tar -xf /tmp/pycharm-2019.2.4.tar.gz --strip-components=1 -C /opt/pycharm-2019.2.4 \
    && rm /tmp/pycharm-2019.2.4.tar.gz

EXPOSE 22

# Switch back to jovyan to avoid accidental container runs as root
USER $NB_UID

# ENV DISPLAY :0
# CMD xeyes
ENTRYPOINT ["/bin/bash"]
