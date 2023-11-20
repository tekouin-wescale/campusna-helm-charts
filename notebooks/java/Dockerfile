FROM jupyter/minimal-notebook:latest


USER root
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

# Code Server Installation
ENV CODE_VERSION=4.19.0
RUN curl -fOL https://github.com/coder/code-server/releases/download/v$CODE_VERSION/code-server_${CODE_VERSION}_amd64.deb \
    && dpkg -i code-server_${CODE_VERSION}_amd64.deb \
    && rm -f code-server_${CODE_VERSION}_amd64.deb



# Extentions

RUN pip install jupyterlab-night --no-cache-dir   &&  fix-permissions "${CONDA_DIR}" && fix-permissions "/home/${NB_USER}"
RUN pip install jupyterlab-language-pack-fr-FR &&  fix-permissions "${CONDA_DIR}" && fix-permissions "/home/${NB_USER}"
RUN pip install jupyter-lsp &&  fix-permissions "${CONDA_DIR}" && fix-permissions "/home/${NB_USER}"
RUN pip install zsh-jupyter-kernel &&  fix-permissions "${CONDA_DIR}" && fix-permissions "/home/${NB_USER}"
RUN pip install bash_kernel  &&  fix-permissions "${CONDA_DIR}" && fix-permissions "/home/${NB_USER}"
RUN code-server --install-extension ritwickdey.LiveServer 
RUN code-server --install-extension  ms-python.python 
RUN code-server --install-extension  ms-vscode.live-server ms-toolsai.jupyter esbenp.prettier-vscode
RUN /opt/conda/bin/conda install -c conda-forge jupyter-server-proxy
RUN /opt/conda/bin/conda install -c conda-forge jupyter-vscode-proxy
RUN /opt/conda/bin/conda install -c conda-forge jupyter-collaboration


# Install dependencies
RUN apt-get update && apt-get install -y \
    software-properties-common \
    curl

# Install Zulu OpenJdk 11 (LTS)
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 0xB1998361219BD9C9 \
    && apt-add-repository 'deb http://repos.azulsystems.com/ubuntu stable main' \
    && apt install -y zulu-11

# Unpack and install the kernel
RUN curl -L https://github.com/SpencerPark/IJava/releases/download/v1.3.0/ijava-1.3.0.zip > ijava-kernel.zip
RUN unzip ijava-kernel.zip -d ijava-kernel \
    && cd ijava-kernel \
    && python3 install.py --sys-prefix

# Install jupyter RISE extension.
RUN pip install jupyter_contrib-nbextensions RISE \
    && jupyter-nbextension install rise --py --system \
    && jupyter-nbextension enable rise --py --system \
    && jupyter contrib nbextension install --system \
    && jupyter nbextension enable hide_input/main

# Cleanup
RUN rm ijava-kernel.zip


# Set user back to priviledged user.
USER $NB_USER