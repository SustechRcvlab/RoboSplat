# RoboSplat Docker Environment

This folder contains Docker and Docker Compose configuration for quickly building and running RoboSplat.

## Prerequisites

| Requirement | Version |
|---|---|
| Docker Engine | ≥ 20.10 |
| Docker Compose | ≥ 2.0 (the `docker compose` plugin) |
| NVIDIA Driver | ≥ 520 (for CUDA 11.8 support) |
| [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) | latest |

## Quick Start

### 1. Build the image

```bash
cd docker
docker compose build
```

> **Note:** The first build may take a long time because it compiles PyTorch3D and diff-gaussian-rasterization from source.

### 2. Download data

Before running, download `data.zip` from the [Google Drive folder](https://drive.google.com/drive/folders/1zUsHHKl21251-LdehpujsUFqt36vkqQX?usp=sharing), place it in the repository root, and unzip it:

```bash
cd ..          # back to repository root
unzip data.zip
```

The `data/` directory will be mounted into the container automatically.

### 3. Run demo generation

```bash
cd docker
docker compose run --rm robosplat python data_aug/generate_demo.py \
    --image_size 256 \
    --save True \
    --save_video True \
    --ref_demo_path data/source_demo/real_000000.h5 \
    --xy_step_str '[10, 10]' \
    --augment_lighting False \
    --augment_appearance False \
    --augment_camera_pose False \
    --output_path data/generated_demo/pick_100
```

Generated results will appear in the `output/` and `data/generated_demo/` directories on the host.

### 4. Interactive shell

```bash
cd docker
docker compose run --rm robosplat bash
```

This drops you into a shell inside the container with all dependencies ready.

## Customization

### Using a different CUDA version

Edit the `FROM` line in `Dockerfile` and the PyTorch install URL to match your CUDA version. For example, for CUDA 12.1:

```dockerfile
FROM nvidia/cuda:12.1.0-devel-ubuntu22.04 AS base
```
```dockerfile
RUN pip install --no-cache-dir \
    torch==2.4.1 torchvision==0.19.1 torchaudio==2.4.1 \
    --index-url https://download.pytorch.org/whl/cu121
```

### Mounting additional directories

Add more entries under `volumes:` in `docker-compose.yml`:

```yaml
volumes:
  - ../data:/workspace/data
  - ../output:/workspace/output
  - /path/on/host:/workspace/custom
```
