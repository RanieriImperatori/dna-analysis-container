# Multi-stage Dockerfile for DNA analysis (QC, trimming, alignment, variant annotation)

# Stage 1: builder
ARG UBUNTU_VERSION=22.04
FROM ubuntu:${UBUNTU_VERSION} AS builder

LABEL stage="builder"
ARG DEBIAN_FRONTEND=noninteractive

# Versions for compiled tools
ARG HTSLIB_VERSION=1.17
ARG SAMTOOLS_VERSION=1.17
ARG BCFTOOLS_VERSION=1.17
ARG BWA_MEM2_VERSION=2.2.1

# Install build dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      ca-certificates \
      curl \
      wget \
      git \
      build-essential \
      autoconf \
      automake \
      pkg-config \
      libncurses-dev \
      libbz2-dev \
      liblzma-dev \
      zlib1g-dev \
      libcurl4-openssl-dev \
      libssl-dev \
      bzip2 \
      xz-utils \
      make \
      gcc \
      g++ \
      unzip \
      tar \
      gzip \
      python3 \
      python3-pip && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /build

# Build and install HTSlib
RUN set -eux; \
    curl -L -o htslib.tar.bz2 \
      https://github.com/samtools/htslib/releases/download/${HTSLIB_VERSION}/htslib-${HTSLIB_VERSION}.tar.bz2; \
    tar -xjf htslib.tar.bz2; \
    cd htslib-${HTSLIB_VERSION}; \
    autoheader && autoconf || true; \
    ./configure --prefix=/usr/local; \
    make -j"$(nproc)"; \
    make install

# --------------------------
# Build and install Samtools
# --------------------------
RUN set -eux; \
    curl -L -o samtools.tar.bz2 \
      https://github.com/samtools/samtools/releases/download/${SAMTOOLS_VERSION}/samtools-${SAMTOOLS_VERSION}.tar.bz2; \
    tar -xjf samtools.tar.bz2; \
    cd samtools-${SAMTOOLS_VERSION}; \
    ./configure --prefix=/usr/local; \
    make -j"$(nproc)"; \
    make install

# Build and install BCFtools
RUN set -eux; \
    curl -L -o bcftools.tar.bz2 \
      https://github.com/samtools/bcftools/releases/download/${BCFTOOLS_VERSION}/bcftools-${BCFTOOLS_VERSION}.tar.bz2; \
    tar -xjf bcftools.tar.bz2; \
    cd bcftools-${BCFTOOLS_VERSION}; \
    ./configure --prefix=/usr/local; \
    make -j"$(nproc)"; \
    make install

# --------------------------
# Build bwa-mem2
# --------------------------
RUN set -eux; \
    curl -L -o bwa-mem2.tar.bz2 \
      https://github.com/bwa-mem2/bwa-mem2/releases/download/v${BWA_MEM2_VERSION}/bwa-mem2-${BWA_MEM2_VERSION}.tar.bz2; \
    tar -xjf bwa-mem2.tar.bz2; \
    cd bwa-mem2-${BWA_MEM2_VERSION}; \
    make -j"$(nproc)"; \
    cp bwa-mem2 /usr/local/bin/bwa-mem2

# Stage 2: runtime
FROM ubuntu:${UBUNTU_VERSION} AS runtime

LABEL maintainer="your.name@example.com"
ARG DEBIAN_FRONTEND=noninteractive

# Versions for JAR-based tools
ARG PICARD_VERSION=2.41.0
ARG SNPEFF_VERSION=5.1d

# Common env vars
ENV PATH=/opt/bin:/usr/local/bin:$PATH
ENV SNPEFF_HOME=/opt/snpeff
ENV PICARD_HOME=/opt/picard
ENV JAVA_TOOL_OPTIONS="-Xms256m -Xmx4g"

# Install runtime dependencies and tools from apt
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      ca-certificates \
      curl \
      wget \
      git \
      unzip \
      bzip2 \
      xz-utils \
      python3 \
      python3-pip \
      default-jre-headless \
      zlib1g \
      libbz2-1.0 \
      liblzma5 \
      libncursesw6 \
      libcurl4 \
      libssl3 \
      procps \
      gzip \
      tar \
      gnupg \
      # bioinformatics tools available via apt
      fastqc \
      fastp \
      bwa \
      bedtools \
      vcftools \
      && \
    rm -rf /var/lib/apt/lists/*

# Python packages (runtime only)
RUN python3 -m pip install --no-cache-dir \
      cutadapt \
      pysam \
      cyvcf2

# Create directories for custom tools
RUN mkdir -p /opt/bin /opt/tools /opt/snpeff /opt/picard && \
    chmod 755 /opt /opt/bin /opt/tools

# Copy compiled binaries and libraries from builder stage
COPY --from=builder /usr/local/bin/samtools /usr/local/bin/samtools
COPY --from=builder /usr/local/bin/bcftools /usr/local/bin/bcftools
COPY --from=builder /usr/local/bin/bwa-mem2 /usr/local/bin/bwa-mem2
COPY --from=builder /usr/local/lib /usr/local/lib

# Install Picard (jar)
RUN set -eux; \
    PICARD_URL="https://github.com/broadinstitute/picard/releases/download/${PICARD_VERSION}/picard.jar"; \
    echo "Downloading Picard from ${PICARD_URL}"; \
    curl -fsSL "${PICARD_URL}" -o /opt/picard/picard.jar; \
    chmod 644 /opt/picard/picard.jar

# Install snpEff
RUN set -eux; \
    SNPEFF_ZIP_URL="https://sourceforge.net/projects/snpeff/files/snpEff_v${SNPEFF_VERSION}_core.zip/download"; \
    echo "Downloading snpEff from ${SNPEFF_ZIP_URL}"; \
    cd /opt; \
    curl -L "${SNPEFF_ZIP_URL}" -o /opt/snpeff.zip; \
    unzip /opt/snpeff.zip -d /opt; \
    rm /opt/snpeff.zip; \
    if [ -d /opt/snpEff ]; then mv /opt/snpEff ${SNPEFF_HOME}; fi; \
    chmod -R 755 ${SNPEFF_HOME}

# Symlinks for convenience
RUN ln -s /usr/bin/fastqc /opt/bin/fastqc || true && \
    ln -s /usr/bin/fastp /opt/bin/fastp || true && \
    ln -s /usr/bin/bwa /opt/bin/bwa || true && \
    ln -s /usr/bin/bedtools /opt/bin/bedtools || true && \
    ln -s /usr/bin/vcftools /opt/bin/vcftools || true && \
    ln -s /usr/local/bin/samtools /opt/bin/samtools || true && \
    ln -s /usr/local/bin/bcftools /opt/bin/bcftools || true && \
    ln -s /usr/local/bin/bwa-mem2 /opt/bin/bwa-mem2 || true

# Create non-root user
RUN useradd -m -u 1000 -s /bin/bash bioinfo && \
    mkdir -p /home/bioinfo/work && \
    chown -R bioinfo:bioinfo /home/bioinfo /opt

USER bioinfo
WORKDIR /home/bioinfo/work

# Smoke tests at build-time (optional but useful)
RUN set -eux; \
    echo "Checking installed tool versions:"; \
    fastqc --version || true; \
    fastp --version || true; \
    bwa 2>&1 | head -n 1 || true; \
    bwa-mem2 2>&1 | head -n 1 || true; \
    samtools --version | head -n 1 || true; \
    bcftools --version | head -n 1 || true; \
    bedtools --version 2>&1 || true; \
    python3 -c "import cutadapt; print('cutadapt OK')" || true; \
    java -jar /opt/picard/picard.jar --version 2>&1 || true; \
    java -jar ${SNPEFF_HOME}/snpEff.jar 2>&1 || true

ENTRYPOINT ["/bin/bash"]
