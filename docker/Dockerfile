FROM nfcore/base

LABEL authors="phil@lifebit.ai" \
      description="Docker image containing fastqc & multiqc for the JAX workshop"

RUN conda install -c bioconda fastqc=0.11.8 && \
    conda install -c bioconda multiqc=1.7