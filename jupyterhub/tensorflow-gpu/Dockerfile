# Base image: https://github.com/jupyter/docker-stacks/blob/74bbd0bffc3b444e2d65279739bc2d681b6199e2/images/docker-stacks-foundation/Dockerfile

ARG ROOT_CONTAINER=tensorflow/tensorflow:2.14.0-gpu

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

# Configure environment
ENV CONDA_DIR=/opt/conda \
    SHELL=/bin/bash \
    NB_USER="${NB_USER}" \
    NB_UID=${NB_UID} \
    NB_GID=${NB_GID} \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8
ENV PATH="${CONDA_DIR}/bin:${PATH}" \
    HOME="/home/${NB_USER}"


SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root

COPY --from=foundation /usr/local/bin/fix-permissions /usr/local/bin/
COPY --from=foundation --chown="${NB_UID}:${NB_GID}" "${CONDA_DIR}/.condarc" "${CONDA_DIR}/"
COPY --from=foundation /usr/local/bin/run-hooks.sh /usr/local/bin/
COPY --from=foundation /usr/local/bin/start.sh /usr/local/bin/

COPY --from=base_notebook /usr/local/bin/start-notebook.sh /usr/local/bin/
COPY --from=base_notebook /usr/local/bin/start-singleuser.sh /usr/local/bin/
COPY --from=base_notebook /etc/jupyter/jupyter_server_config.py /etc/jupyter/
COPY --from=base_notebook /etc/jupyter/docker_healthcheck.py /etc/jupyter/

ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update --yes && \
    apt-get upgrade --yes && \
    apt-get install --yes --no-install-recommends \
    bzip2 \
    curl \
    git \
    tzdata \
    openssh-client \
    less \
    texlive-xetex \
    texlive-fonts-recommended \
    texlive-plain-generic \
    xclip \
    git \
    nano \
    bat \    
    fonts-liberation \
    pandoc \
    run-one && \
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
    apt install -y eza

# Copy a script that we will use to correct permissions after running certain commands
RUN chmod a+rx /usr/local/bin/fix-permissions

# Enable prompt color in the skeleton .bashrc before creating the default NB_USER
# hadolint ignore=SC2016
RUN sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc && \
    # Add call to conda init script see https://stackoverflow.com/a/58081608/4413446
    echo 'eval "$(command conda shell.bash hook 2> /dev/null)"' >> /etc/skel/.bashrc

# Create NB_USER with name jovyan user with UID=1000 and in the 'users' group
# and make sure these dirs are writable by the `users` group.
RUN echo "auth requisite pam_deny.so" >> /etc/pam.d/su && \
    sed -i.bak -e 's/^%admin/#%admin/' /etc/sudoers && \
    sed -i.bak -e 's/^%sudo/#%sudo/' /etc/sudoers && \
    useradd --no-log-init --create-home --shell /bin/bash --uid "${NB_UID}" --no-user-group "${NB_USER}" && \
    mkdir -p "${CONDA_DIR}" && \
    chown "${NB_USER}:${NB_GID}" "${CONDA_DIR}" && \
    chmod g+w /etc/passwd && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

USER ${NB_UID}

# Pin python version here, or set it to "default"
ARG PYTHON_VERSION=3.11

# Setup work directory for backward-compatibility
RUN mkdir "/home/${NB_USER}/work" && \
    fix-permissions "/home/${NB_USER}"

# Download and install Micromamba, and initialize Conda prefix.
#   <https://github.com/mamba-org/mamba#micromamba>
#   Similar projects using Micromamba:
#     - Micromamba-Docker: <https://github.com/mamba-org/micromamba-docker>
#     - repo2docker: <https://github.com/jupyterhub/repo2docker>
# Install Python, Mamba and jupyter_core
# Cleanup temporary files and remove Micromamba
# Correct permissions
# Do all this in a single RUN command to avoid duplicating all of the
# files across image layers when the permissions change
WORKDIR /tmp
RUN set -x && \
    arch=$(uname -m) && \
    if [ "${arch}" = "x86_64" ]; then \
        # Should be simpler, see <https://github.com/mamba-org/mamba/issues/1437>
        arch="64"; \
    fi && \
    wget --progress=dot:giga -O /tmp/micromamba.tar.bz2 \
        "https://micromamba.snakepit.net/api/micromamba/linux-${arch}/latest" && \
    tar -xvjf /tmp/micromamba.tar.bz2 --strip-components=1 bin/micromamba && \
    rm /tmp/micromamba.tar.bz2 && \
    PYTHON_SPECIFIER="python=${PYTHON_VERSION}" && \
    if [[ "${PYTHON_VERSION}" == "default" ]]; then PYTHON_SPECIFIER="python"; fi && \
    # Install the packages
    ./micromamba install \
        --root-prefix="${CONDA_DIR}" \
        --prefix="${CONDA_DIR}" \
        --yes \
        "${PYTHON_SPECIFIER}" \
        'mamba' \
        'jupyter_core' && \
    rm micromamba && \
    # Pin major.minor version of python
    mamba list python | grep '^python ' | tr -s ' ' | cut -d ' ' -f 1,2 >> "${CONDA_DIR}/conda-meta/pinned" && \
    mamba clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

# Configure container startup
ENTRYPOINT ["tini", "-g", "--"]
# CMD ["start.sh"]

USER root

# Create dirs for startup hooks
RUN mkdir /usr/local/bin/start-notebook.d && \
    mkdir /usr/local/bin/before-notebook.d

USER ${NB_UID}

# Install JupyterLab, Jupyter Notebook, JupyterHub and NBClassic
# Generate a Jupyter Server config
# Cleanup temporary files
# Correct permissions
# Do all this in a single RUN command to avoid duplicating all of the
# files across image layers when the permissions change
WORKDIR /tmp
RUN mamba install --yes \
    'jupyterlab' \
    'notebook' \
    'jupyterhub' \
    'nbclassic' && \
    jupyter server --generate-config && \
    mamba clean --all -f -y && \
    npm cache clean --force && \
    jupyter lab clean && \
    rm -rf "/home/${NB_USER}/.cache/yarn" && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

ENV JUPYTER_PORT=8888
EXPOSE $JUPYTER_PORT

# Configure container startup
CMD ["start-notebook.sh"]

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