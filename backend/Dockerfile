# Cần image "devel" (có nvcc) để biên dịch nvdiffrast cho Deep3D
FROM nvidia/cuda:12.1.1-devel-ubuntu22.04

ENV DEBIAN_FRONTEND=noninteractive \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1

# Ubuntu 22.04 đi kèm Python 3.10
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 python3-dev python3-pip python-is-python3 \
    build-essential git ninja-build libgl1 libglib2.0-0 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY requirements_cu12.txt .
RUN pip install --default-timeout=1000 -r requirements_cu12.txt

# nvdiffrast biên dịch CUDA plugin lúc chạy lần đầu (JIT), cache ở /root/.cache/torch_extensions
RUN pip install --default-timeout=1000 --no-build-isolation \
    git+https://github.com/NVlabs/nvdiffrast.git@v0.3.3

COPY . .

EXPOSE 5002
CMD ["python", "app.py"]
