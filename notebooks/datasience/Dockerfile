FROM quay.io/jupyter/datascience-notebook:latest

# Fix: https://github.com/hadolint/hadolint/wiki/DL4006
# Fix: https://github.com/koalaman/shellcheck/wiki/SC3014
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root

# dev
RUN apt-get update --yes && \
    apt-get install --yes --quiet --no-install-recommends \
    curl \
    iputils-ping \
    build-essential \
    make \
    cmake \
    g++ \
    clang \
    htop \
    libopencv-dev \
    && \
    apt-get --quiet clean && rm -rf /var/lib/apt/lists/*

# Vs code
ENV CODE_VERSION=4.19.0+
RUN curl -fOL https://github.com/coder/code-server/releases/download/v$CODE_VERSION/code-server_${CODE_VERSION}_amd64.deb \
    && dpkg -i code-server_${CODE_VERSION}_amd64.deb \
    && rm -f code-server_${CODE_VERSION}_amd64.deb





# Extentions
RUN /opt/conda/bin/conda install -c conda-forge jupyter-server-proxy
RUN /opt/conda/bin/conda install -c conda-forge jupyter-vscode-proxy
RUN /opt/conda/bin/conda install -c conda-forge jupyter-collaboration
RUN pip install jupyterlab-night --no-cache-dir   &&  fix-permissions "${CONDA_DIR}" && fix-permissions "/home/${NB_USER}"
RUN pip install jupyterlab-language-pack-fr-FR --no-cache-dir   &&  fix-permissions "${CONDA_DIR}" && fix-permissions "/home/${NB_USER}"
RUN pip install jupyter-lsp --no-cache-dir   &&  fix-permissions "${CONDA_DIR}" && fix-permissions "/home/${NB_USER}"
RUN pip install zsh-jupyter-kernel --no-cache-dir   &&  fix-permissions "${CONDA_DIR}" && fix-permissions "/home/${NB_USER}"
RUN code-server --install-extension ritwickdey.LiveServer
RUN code-server --install-extension  ms-python.python 
RUN code-server --install-extension  ms-vscode.live-server ms-toolsai.jupyter esbenp.prettier-vscode
# Switch back to jovyan to avoid accidental container runs as root
USER ${NB_UID}
