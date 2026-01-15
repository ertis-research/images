# syntax=docker/dockerfile:1.3-labs

FROM ghcr.io/ertis-research/jupyterhub-base-py3.11:main

ARG DEBIAN_FRONTEND=noninteractive
ARG NB_USER="ertis"

USER root

# COPY requirements.txt /tmp/requirements.txt

# RUN pip install --no-cache-dir -r /tmp/requirements.txt

#######################################################################################################
# Go and GoNB Libraries
#######################################################################################################
ARG GO_VERSION=1.25.5
ENV GOROOT=/usr/local/go
ENV GOPATH=$HOME/go
ENV PATH=$PATH:$GOROOT/bin:$GOPATH/bin

# Add exported variables to $NB_USER .profile: notice we start the docker as root, and it executes
# JupyterLab with a `su -l $NB_USER`, so the environment variables are lost. We could use --preserve_environment
# for GOROOT and GOPATH, but it feels safer to add these to .profile, so the autostart.sh script can also
# execute `su -l $NB_USER -c "<some command>"` if it wants.
USER ${NB_UID}
WORKDIR ${HOME}
RUN <<EOF
  echo "NB_UID=${NB_UID}" >> .profile
  echo "export PATH=${PATH}" >> .profile
  echo "export GOPATH=${GOPATH}" >> .profile
  echo "export GOROOT=${GOROOT}" >> .profile
EOF

USER root
WORKDIR /usr/local
RUN wget --quiet --output-document=- "https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz" | tar -xvz \
    && go version

# Install GoNB (https://github.com/janpfeifer/gonb) in the user account
USER ${NB_UID}
WORKDIR ${HOME}
RUN go install golang.org/x/tools/cmd/goimports@latest && \
    go install golang.org/x/tools/gopls@latest

# Clone from main, build&install gonb binary, and then install it as a kernel in Jupyter.
# - First introduce the cache-busting argument. This number can be bumped whenever we only want
#   to rebuild the gonb part.
ARG CACHEBUST=2
WORKDIR ${HOME}
RUN git clone 'https://github.com/janpfeifer/gonb.git'
WORKDIR ${HOME}/gonb
RUN go install . && \
    gonb --install

########################################################################################################

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV HOME="/home/${NB_USER}"

# RUN chsh -s /bin/bash

USER ${NB_UID}

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN <<EOF
    echo "alias ls='eza'" >> ~/.bashrc
    echo "alias cat='batcat'" >> ~/.bashrc
EOF

WORKDIR "${HOME}"