# https://docs.docker.com/engine/reference/builder/
# /home/jovyan/work

FROM quay.io/jupyter/minimal-notebook:latest
USER root

# Install all OS dependencies for fully functional notebook server
RUN pip install bash_kernel && python -m bash_kernel.install

# Allow using 'su -' to make sandbox changes
RUN chpasswd <<<"root:root" && \
    sed -Ei 's/(.*pam_deny.so)/# \1/' /etc/pam.d/su
RUN /opt/conda/bin/conda install -c conda-forge jupyter-collaboration

# Switch back to jovyan to avoid accidental container runs as root
USER $NB_UID
EXPOSE 8888