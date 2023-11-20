# syntax=docker/dockerfile:1.3-labs

FROM ghcr.io/ertis-research/jupyterhub-base-cuda12.2-cudnn8-py3.11:main

ARG DEBIAN_FRONTEND=noninteractive

USER root

COPY requirements.txt /tmp/requirements.txt

RUN pip install --no-cache-dir -r /tmp/requirements.txt

ARG NB_USER="ertis"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV HOME="/home/${NB_USER}"

RUN chsh -s /bin/bash

USER ${NB_UID}

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN <<EOF
    echo "alias ls='eza'" >> ~/.bashrc
    echo "alias cat='batcat'" >> ~/.bashrc
EOF

WORKDIR "${HOME}"