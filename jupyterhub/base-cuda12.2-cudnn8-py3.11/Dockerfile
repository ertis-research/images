# syntax=docker/dockerfile:1.3-labs

FROM nvidia/cuda:12.2.2-cudnn8-devel-ubuntu22.04

ARG DEBIAN_FRONTEND=noninteractive

USER root

# Install some basic utilities and python3.11
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    ca-certificates \
    sudo \
    git \
    bzip2 \
    tzdata \
    less \
    xclip \
    libx11-6 \
    nano \
    bat \    
    locales \
    sudo \
    iputils-ping \
    gpg \
    tini \
    wget \
    dnsutils \
    zip \
    unzip \
    htop \
    atop \
    jq \
    zsh \
    sed \
    graphviz \
    && \    
    apt-get clean && rm -rf /var/lib/apt/lists/* && \
    echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

# Install EZA
RUN mkdir -p /etc/apt/keyrings && \
    wget -qO- https://raw.githubusercontent.com/eza-community/eza/main/deb.asc | sudo gpg --dearmor -o /etc/apt/keyrings/gierens.gpg && \
    echo "deb [signed-by=/etc/apt/keyrings/gierens.gpg] http://deb.gierens.de stable main" | sudo tee /etc/apt/sources.list.d/gierens.list && \
    chmod 644 /etc/apt/keyrings/gierens.gpg /etc/apt/sources.list.d/gierens.list && \
    apt update -y && \
    apt install -y eza \
    && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Install Python 3.11
RUN apt-get update --yes && \
    apt-get install --yes --no-install-recommends \    
    python3.11 \
    python3-pip \
    python3.11-dev \
    python3.11-distutils \
    python3.11-venv \
    && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

RUN curl -LsSf https://astral.sh/uv/install.sh | sh

COPY jupyter_requirements.txt /tmp/jupyter_requirements.txt

RUN pip install --no-cache-dir -r /tmp/jupyter_requirements.txt

# Initialization of user copied from:
# https://github.com/jupyter/docker-stacks/blob/main/docker-stacks-foundation/Dockerfile

ARG NB_USER="ertis"
ARG NB_UID="1000"
ARG NB_GID="100"

# Fix: https://github.com/hadolint/hadolint/wiki/DL4006
# Fix: https://github.com/koalaman/shellcheck/wiki/SC3014
SHELL ["/bin/zsh", "-o", "pipefail", "-c"]

USER root

COPY fix-permissions /usr/local/bin/fix-permissions
RUN chmod a+rx /usr/local/bin/fix-permissions

ENV HOME="/home/${NB_USER}"

# Enable prompt color in the skeleton .bashrc before creating the default NB_USER
# hadolint ignore=SC2016
RUN sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc

# Create NB_USER with name jovyan user with UID=1000 and in the 'users' group
# and make sure these dirs are writable by the `users` group.
RUN echo "auth requisite pam_deny.so" >> /etc/pam.d/su && \
    useradd -l -m -s /bin/bash -N -u "${NB_UID}" "${NB_USER}" && \
    chmod g+w /etc/passwd && \
    fix-permissions "${HOME}"

# RUN chsh -s /bin/zsh

RUN sed -i 's/auth\s*sufficient\s*pam_shells.so/auth\s*required\s*pam_shells.so/g' /etc/pam.d/chsh
# Additional line to modify pam_rootok.so (usually not needed)
# RUN sed -i 's/auth\s*sufficient\s*pam_rootok.so/auth\s*required\s*pam_rootok.so/g' /etc/pam.d/chsh

USER ${NB_UID}

# RUN chsh -s /bin/zsh

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN <<EOF
    echo "alias ls='eza'" >> ~/.bashrc
    echo "alias cat='batcat'" >> ~/.bashrc
EOF


RUN <<EOF
    echo "alias ls='eza'" >> ~/.zshrc
    echo "alias cat='batcat'" >> ~/.zshrc
EOF

WORKDIR "${HOME}"
