# ─────────────────────────────────────────────────────────────────────
# VoxCPM Training WebUI — Docker image
# ─────────────────────────────────────────────────────────────────────
# Base: PyTorch with CUDA for GPU-accelerated LoRA fine-tuning.
# Build context should be the project root:
#
#   docker build -f docker/Dockerfile -t voxcpm-training .
#
# ─────────────────────────────────────────────────────────────────────
FROM pytorch/pytorch:2.5.1-cuda12.4-cudnn9-devel

LABEL maintainer="OpenBMB <openbmb@gmail.com>"
LABEL description="VoxCPM LoRA Training WebUI with GPU support"

# Avoid interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# System deps required by Python packages:
#   git        — setuptools_scm needs it to resolve version in pyproject.toml
#   libsndfile1 — C library backing the 'soundfile' Python package
#   ffmpeg     — audio codec support for torchaudio/librosa
RUN apt-get update && apt-get install -y --no-install-recommends \
        git \
        libsndfile1 \
        ffmpeg \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Layer 1: Install dependencies only (cached unless pyproject.toml changes)
# Create a minimal package stub so pip can resolve deps without real source.
COPY pyproject.toml /app/
RUN mkdir -p /app/src/voxcpm && echo '__version__ = "0.0.0"' > /app/src/voxcpm/__init__.py
ENV SETUPTOOLS_SCM_PRETEND_VERSION=0.0.0
RUN pip install --no-cache-dir -e .

# Layer 2: Copy full project source (cheap rebuild on code changes)
COPY . /app/

# Create default directories and declare volumes
RUN mkdir -p /app/lora /app/models /app/output /app/data
VOLUME ["/app/models", "/app/lora", "/app/output", "/app/data"]

EXPOSE 7860

# Environment variables for configuration
ENV GRADIO_SERVER_PORT=7860
ENV GRADIO_ROOT_PATH=""
ENV HF_HOME=/app/models

# Default: launch training WebUI
CMD ["python", "lora_ft_webui.py"]
