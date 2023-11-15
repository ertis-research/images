# Base image: https://github.com/jupyter/docker-stacks/blob/74bbd0bffc3b444e2d65279739bc2d681b6199e2/images/docker-stacks-foundation/Dockerfile

ARG ROOT_CONTAINER=tensorflow/tensorflow:2.14.0-gpu
# ARG ROOT_CONTAINER=ubuntu:22.04

ARG FOUNDATION_CONTAINER=quay.io/jupyter/docker-stacks-foundation:4d70cf8da953
ARG BASE_NOTEBOOK=quay.io/jupyter/base-notebook:b86753318aa1
ARG MIN_NOTEBOOK


FROM ${FOUNDATION_CONTAINER} as foundation
FROM ${BASE_NOTEBOOK} as base_notebook

FROM $ROOT_CONTAINER

LABEL maintainer="Antonio J. Chaves <chaves@uma.es>"

ARG NB_USER="ertis"
ARG NB_UID="1000"
ARG NB_GID="100"

# Pin python version here, or set it to "default"
ARG PYTHON_VERSION=3.11


ENV DEBIAN_FRONTEND noninteractive

# Configure environment
ENV SHELL=/bin/bash \
    NB_USER="${NB_USER}" \
    NB_UID=${NB_UID} \
    NB_GID=${NB_GID} \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8 \
    PATH="${PATH}:/home/${NB_USER}/.local/bin"


ENV HOME="/home/${NB_USER}"

ENV JUPYTER_PORT=8888


SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root

COPY --from=foundation /usr/local/bin/fix-permissions /usr/local/bin/
COPY --from=foundation /usr/local/bin/start.sh /usr/local/bin/
COPY run-hooks.sh /usr/local/bin/run-hooks.sh

COPY --from=base_notebook /usr/local/bin/start-notebook.sh /usr/local/bin/
COPY --from=base_notebook /usr/local/bin/start-singleuser.sh /usr/local/bin/
COPY start-notebook.py start-singleuser.py /usr/local/bin/
# COPY --from=base_notebook /etc/jupyter/jupyter_server_config.py /etc/jupyter/
# COPY --from=base_notebook /etc/jupyter/docker_healthcheck.py /etc/jupyter/
COPY jupyter_server_config.py /etc/jupyter/jupyter_server_config.py
COPY docker_healthcheck.py /etc/jupyter/docker_healthcheck.py


RUN apt-get update --yes && \
    apt-get upgrade --yes && \
    apt-get install --yes --no-install-recommends \
    bzip2 \
    curl \
    git \
    tzdata \
    openssh-client \
    less \
    xclip \
    git \
    nano \
    bat \    
    run-one \
    iputils-ping \
    ca-certificates \
    locales \
    sudo \
    gpg \
    tini \
    wget && \
    apt-get clean && rm -rf /var/lib/apt/lists/* && \
    echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen


RUN mkdir -p /etc/apt/keyrings && \
    wget -qO- https://raw.githubusercontent.com/eza-community/eza/main/deb.asc | sudo gpg --dearmor -o /etc/apt/keyrings/gierens.gpg && \
    echo "deb [signed-by=/etc/apt/keyrings/gierens.gpg] http://deb.gierens.de stable main" | sudo tee /etc/apt/sources.list.d/gierens.list && \
    chmod 644 /etc/apt/keyrings/gierens.gpg /etc/apt/sources.list.d/gierens.list && \
    apt update -y && \
    apt install -y eza \
    texlive-xetex \
    texlive-fonts-recommended \
    texlive-plain-generic \
    fonts-liberation \
    pandoc \
    python3-pip && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Copy a script that we will use to correct permissions after running certain commands
RUN chmod a+rx /usr/local/bin/fix-permissions

RUN echo "auth requisite pam_deny.so" >> /etc/pam.d/su && \
    sed -i.bak -e 's/^%admin/#%admin/' /etc/sudoers && \
    sed -i.bak -e 's/^%sudo/#%sudo/' /etc/sudoers && \
    useradd --no-log-init --create-home --shell /bin/bash --uid "${NB_UID}" --no-user-group "${NB_USER}" && \
    chmod g+w /etc/passwd && \
    fix-permissions "/home/${NB_USER}"

USER ${NB_UID}

# Setup work directory for backward-compatibility
RUN mkdir "/home/${NB_USER}/work" && \
    fix-permissions "/home/${NB_USER}"

WORKDIR /tmp

RUN pip install jupyter-core \
    jupyterlab \ 
    notebook \
    jupyterhub \
    nbclassic && \
    jupyter server --generate-config && \
    jupyter lab clean && \
    rm -rf "/home/${NB_USER}/.cache/yarn" && \
    fix-permissions "/home/${NB_USER}"

# Configure container startup
ENTRYPOINT ["tini", "-g", "--"]

USER root

# Create dirs for startup hooks
RUN mkdir /usr/local/bin/start-notebook.d && \
    mkdir /usr/local/bin/before-notebook.d

USER ${NB_UID}


EXPOSE $JUPYTER_PORT

# Configure container startup
CMD ["start-notebook.py"]

# Fix permissions on /etc/jupyter as root
USER root
RUN fix-permissions /etc/jupyter/

# HEALTHCHECK documentation: https://docs.docker.com/engine/reference/builder/#healthcheck
# This healtcheck works well for `lab`, `notebook`, `nbclassic`, `server` and `retro` jupyter commands
# https://github.com/jupyter/docker-stacks/issues/915#issuecomment-1068528799
HEALTHCHECK --interval=5s --timeout=3s --start-period=5s --retries=3 \
    CMD /etc/jupyter/docker_healthcheck.py || exit 1

# Switch back to jovyan to avoid accidental container runs as root
USER ${NB_UID}

WORKDIR "${HOME}"