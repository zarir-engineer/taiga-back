# Use a lightweight Python base image
FROM python:3.8

# Install system dependencies
RUN apt-get update && apt-get install -y \
    libxml2 \
    libxslt1.1 \
    libxml2-dev \
    libxslt-dev \
    gcc \
    python3-venv \
    python3-dev \
    libpq-dev \
    libssl-dev \
    libffi-dev \
    build-essential \
    zlib1g-dev \
    gettext \
    git \
    net-tools \
    procps \
    wget \
    libgomp1 \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Set up virtual environment
RUN python -m venv /taiga/venv
ENV PATH="/taiga/venv/bin:$PATH"

# Set working directory
WORKDIR /taiga-back
# Upgrade pip, setuptools, and wheel before installing any Python packages
RUN /taiga/venv/bin/pip install --no-cache-dir --upgrade pip setuptools wheel

RUN /taiga/venv/bin/pip install --no-cache-dir --upgrade cython
RUN /taiga/venv/bin/pip install --no-cache-dir --no-binary lxml lxml==4.9.2


# **Manually set environment variables for compilation**
ENV CFLAGS="-fPIC"
ENV LDFLAGS="-L/usr/lib/x86_64-linux-gnu"

# **Explicitly install cffi separately before other dependencies**
RUN /taiga/venv/bin/pip install --no-cache-dir --force-reinstall cffi

# Copy only requirements first (to optimize Docker caching)
COPY requirements.txt requirements-contribs.txt ./

# Install NumPy first (ensures prebuilt wheel is used)
RUN /taiga/venv/bin/pip install --no-cache-dir "numpy>=1.22.4"

# Install Python dependencies
RUN /taiga/venv/bin/pip install --no-cache-dir --use-deprecated=legacy-resolver -r requirements.txt \
    && /taiga/venv/bin/pip install --no-cache-dir -r requirements-contribs.txt

# Copy the rest of the app
COPY . .

# Ensure entrypoint scripts are executable
RUN chmod +x docker/entrypoint.sh docker/async_entrypoint.sh

# Set Django settings module
ENV DJANGO_SETTINGS_MODULE=settings.config

# Expose API port
EXPOSE 8000

# Run entrypoint script
ENTRYPOINT ["./docker/entrypoint.sh"]
