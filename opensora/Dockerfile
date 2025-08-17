#FROM nvidia/cuda:12.1.0-devel-ubuntu20.04
FROM public.ecr.aws/sagemaker/sagemaker-distribution:latest-gpu

USER root
# ARGS for sagemaker 
ARG NB_USER="sagemaker-user"
ARG NB_UID="1000"
ARG NB_GID="100"

# Add micromamba to PATH
ENV PATH="/usr/local/bin:$PATH"
# Specify Cuda
ENV CUDA_HOME=/usr/local/cuda
# Nice to have for easier command writing (less -y's)
ENV DEBIAN_FRONTEND=noninteractive

# Install necessary system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    wget  \
    curl \
    build-essential \
    git \
    # python3-pip \
    # python3-dev \
    # nvidia-cuda-toolkit \
    ca-certificates && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# # Create SageMaker user
# RUN useradd -m -s /bin/bash -N -u $NB_UID $NB_USER && \
#     chmod g+w /etc/passwd && \
#     echo "${NB_USER} ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# Download and install micromamba
# RUN wget -qO- https://micro.mamba.pm/api/micromamba/linux-64/latest | tar xvj -C /usr/local bin/micromamba

# install environment and difficult dependency tensornvme
COPY ./pyproject.toml /config/workspace/opensora/pyproject.toml


# liburing
RUN git clone https://github.com/axboe/liburing.git && \
  cd liburing && \
  ./configure && \
  make && \
  make install && \
  rm -rf build
ENV CUDA_HOME=/opt/conda/targets/x86_64-linux/
ENV PATH=$PATH:$CUDA_HOME/bin
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CUDA_HOME/lib64
USER $MAMBA_USER

RUN  micromamba install -c conda-forge pyproject2conda && \
# Create SORA env
    micromamba run pyproject2conda yaml -f /config/workspace/opensora/pyproject.toml --python 3.10 -o environment.yaml -e dev --name opensora && \
    micromamba env create --file environment.yaml && \
    micromamba run -n opensora pip3 install --upgrade pip && \
    micromamba install -n opensora -y conda-forge::cmake conda-forge::make  conda-forge::git && \
# install tensornvme
    micromamba run -n opensora git clone https://github.com/hpcaitech/TensorNVMe.git && \
    cd TensorNVMe && \
    micromamba run -n opensora pip install -r requirements.txt && \
    micromamba run -n opensora pip install -v --no-cache-dir . && \
#  # Ensure CUDA-enabled PyTorch
    micromamba run -n opensora pip install --upgrade pip && \
    micromamba run -n opensora pip3 install --no-cache-dir \
    torch \
    torchvision \
    torchaudio \
    --index-url https://download.pytorch.org/whl/cu121 && \
# nice to have libraries
    micromamba run -n opensora pip3 install --no-cache-dir \
    jupyterlab \
    notebook \
    sagemaker \
    boto3 \
    pandas \
    numpy \
    matplotlib \
    scikit-learn 

# # Commands ran each merge
# # Copy local content and reinstall opensora package
RUN mkdir /home/$MAMBA_USER -p
COPY . /home/$MAMBA_USER/
RUN  cd /home/$MAMBA_USER && micromamba run -n opensora pip3 install -e .

# Set default command for JupyterLab
ENV SHELL=/bin/bash
ENV NB_USER=sagemaker-user
ENV NB_UID=1000
ENV UID=1000
ENV NB_GID=100
ENV GID=100
WORKDIR /home/sagemaker-user
USER $MAMBA_USER
ENTRYPOINT ["jupyter-lab"]
CMD ["--ServerApp.ip=0.0.0.0","--ServerApp.port=8888","--ServerApp.allow_origin=*","--ServerApp.token=","--ServerApp.base_url=/jupyterlab/default"]
