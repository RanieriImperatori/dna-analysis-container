# dna-analysis-container

A container image and tooling for local testing and Kubernetes deployment of common DNA analysis tasks: QC, trimming, alignment and variant annotation. The image focuses on reproducible binaries (no Conda) and follows Docker best practices for production readiness.

## Features
- FastQC, fastp (QC and trimming)
- bwa / bwa-mem2 (alignment)
- samtools, bcftools, bedtools, vcftools (SAM/BAM/VCF manipulations)
- Picard (jar) and snpEff (annotation jar)
- Small, production-oriented runtime image (multi-stage) and optional development image with build tools

## Quick start (local)
Build the image locally:
```bash
docker build -t dna-analysis-container:latest -f docker/Dockerfile .
```

Run the image interactively (example):
```bash
docker run --rm -it \
  -v /path/to/data:/data \
  -w /data \
  dna-analysis-container:latest /bin/bash
```