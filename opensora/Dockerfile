#FROM nvidia/cuda:12.1.0-devel-ubuntu20.04
FROM public.ecr.aws/sagemaker/sagemaker-distribution:latest-gpu
ARG NB_USER="sagemaker-user"
ARG NB_UID=1000
ARG NB_GID=100

ENV MAMBA_USER=$NB_USER

USER root




# Add micromamba to PATH
ENV PATH="/usr/local/bin:$PATH"
# Nice to have for easier command writing (less -y's)
ENV DEBIAN_FRONTEND=noninteractive
# install environment and difficult dependency tensornvme
ENV CUDA_HOME=/opt/conda/targets/x86_64-linux/
ENV PATH=$PATH:$CUDA_HOME/bin
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CUDA_HOME/lib64



RUN micromamba install sagemaker-inference --yes --channel conda-forge --name base
RUN apt-get update && apt-get install -y make
RUN git clone https://github.com/axboe/liburing.git && \
    cd liburing && \
    ./configure && \
    make && \
    make install
    # cd .. && \
RUN micromamba install conda-forge::pyproject2conda conda-forge::cmake conda-forge::make  conda-forge::git
COPY ./pyproject.toml /config/workspace/opensora/pyproject.toml
RUN micromamba run -n base pyproject2conda yaml -f /config/workspace/opensora/pyproject.toml --python 3.12 -o environment.yaml && \
    micromamba install -y -n base -f environment.yaml
RUN conda clean --all --yes
COPY . /home/$MAMBA_USER/
#  && \
RUN micromamba run  pip3 install --upgrade pip && \
    micromamba install  -y && \
    micromamba run  git clone https://github.com/hpcaitech/TensorNVMe.git && \
    cd TensorNVMe && \
    micromamba run  pip install -r requirements.txt && \
    micromamba run  pip install -v --no-cache-dir . && \
#     micromamba run  pip3 install --no-cache-dir \
#     jupyterlab \
#     notebook \
#     sagemaker \
#     boto3 \
#     pandas \
#     numpy \
#     matplotlib \
#     scikit-learn && \
    cd /home/$MAMBA_USER && micromamba run  pip3 install .

# Set default command for JupyterLab
ENV SHELL=/bin/bash
ENV NB_USER=sagemaker-user
ENV NB_UID=1000
ENV UID=1000
ENV NB_GID=100
ENV GID=100
USER $MAMBA_USER
ENTRYPOINT ["jupyter-lab"]
CMD ["--ServerApp.ip=0.0.0.0","--ServerApp.port=8888","--ServerApp.allow_origin=*","--ServerApp.token=","--ServerApp.base_url=/jupyterlab/default"]
