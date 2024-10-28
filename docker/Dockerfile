# Use ARM64 compatible base image
FROM ubuntu:22.04

# Set label for the docker image
LABEL description="Docker image for xtts-api-server (ARM64 CPU Version)" \
      maintainer="Your Name <your.email@example.com>" \
      version="1.0"

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1

# Install required packages and clean up in a single layer
RUN apt-get update && \
    apt-get install --no-install-recommends -y \
        python3-dev \
        portaudio19-dev \
        libportaudio2 \
        libasound2-dev \
        libportaudiocpp0 \
        git \
        python3 \
        python3-pip \
        make \
        g++ \
        ffmpeg \
        curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    python3 -m pip install --no-cache-dir --upgrade pip setuptools wheel ninja virtualenv

# Create necessary directories
RUN mkdir -p /app/xtts-server/speakers \
    /app/xtts-server/output \
    /app/xtts-server/models

WORKDIR /app

# Install CPU version of PyTorch for ARM64
RUN pip install --no-cache-dir torch torchaudio --index-url https://download.pytorch.org/whl/cpu

# Install xtts-api-server without deepspeed (since it's CPU-only)
RUN pip install --no-cache-dir xtts-api-server

# Expose the container port
EXPOSE 8020

# Set default environment variables
ENV XTTS_PORT=8020 \
    XTTS_HOST="0.0.0.0" \
    XTTS_URL="http://localhost:8020" \
    SPEAKERS_FOLDER="/app/xtts-server/speakers" \
    OUTPUT_FOLDER="/app/xtts-server/output" \
    MODELS_FOLDER="/app/xtts-server/models"

# Set healthcheck
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:${XTTS_PORT}/health || exit 1

# Run xtts_api_server when the container starts (without deepspeed)
CMD ["sh", "-c", "python3 -m xtts_api_server \
    --listen \
    -p ${XTTS_PORT} \
    -h ${XTTS_HOST} \
    -t ${XTTS_URL} \
    -sf ${SPEAKERS_FOLDER} \
    -o ${OUTPUT_FOLDER} \
    -mf ${MODELS_FOLDER}"]
