# syntax=docker/dockerfile:1
# ==============================================================================
# RoboSplat Docker Image
# Base: NVIDIA CUDA 11.8 + cuDNN 8 + Ubuntu 22.04
# Python 3.10 | PyTorch 2.4.1 | PyTorch3D | diff-gaussian-rasterization
# ==============================================================================

FROM nvidia/cuda:11.8.0-devel-ubuntu22.04 AS base

# Prevent interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
        git \
        curl \
        wget \
        build-essential \
        ninja-build \
        cmake \
        libgl1-mesa-glx \
        libglib2.0-0 \
        libsm6 \
        libxrender1 \
        libxext6 \
        python3.10 \
        python3.10-dev \
        python3.10-venv \
        python3-pip \
    && ln -sf /usr/bin/python3.10 /usr/bin/python \
    && ln -sf /usr/bin/pip3 /usr/bin/pip \
    && rm -rf /var/lib/apt/lists/*

# Upgrade pip
RUN python -m pip install --no-cache-dir --upgrade pip setuptools wheel

# ---------- Stage: PyTorch ----------
RUN pip install --no-cache-dir \
    torch==2.4.1 \
    torchvision==0.19.1 \
    torchaudio==2.4.1 \
    --index-url https://download.pytorch.org/whl/cu118

# ---------- Stage: Python dependencies ----------
COPY requirements.txt /tmp/requirements.txt
RUN pip install --no-cache-dir -r /tmp/requirements.txt \
    && rm /tmp/requirements.txt

# ---------- Stage: diff-gaussian-rasterization ----------
COPY third_party/diff-gaussian-rasterization /tmp/diff-gaussian-rasterization
RUN pip install --no-cache-dir /tmp/diff-gaussian-rasterization \
    && rm -rf /tmp/diff-gaussian-rasterization

# ---------- Stage: PyTorch3D ----------
RUN git clone --depth 1 --branch v0.7.8 https://github.com/facebookresearch/pytorch3d.git /tmp/pytorch3d \
    && curl -L -o /tmp/cub.tar.gz https://github.com/NVIDIA/cub/archive/1.16.0.tar.gz \
    && tar xzf /tmp/cub.tar.gz -C /tmp \
    && CUB_HOME=/tmp/cub-1.16.0 pip install --no-cache-dir /tmp/pytorch3d \
    && rm -rf /tmp/pytorch3d /tmp/cub-1.16.0 /tmp/cub.tar.gz

# ---------- Application ----------
WORKDIR /workspace
COPY . /workspace

# Default command: show help
CMD ["python", "data_aug/generate_demo.py", "--help"]
