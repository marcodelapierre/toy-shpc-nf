## Demo RNAseq pipeline, to test interplay with SHPC container modules

The pipeline requires [Nextflow](https://github.com/nextflow-io/nextflow) to run, plus some additional tools:

* [Singularity-HPC](http://github.com/singularityhub/singularity-hpc), or SHPC, container modules utility
* [Singularity](http://singularity.hpcng.org) container runtime
* Software modules, i.e. [Lmod](https://lmod.readthedocs.io) or [Environment Modules](https://modules.readthedocs.io)

**Credits**: the backbone of the pipeline (`main.nf`) and sample input data come from [nextflow-io/rnaseq-nf](https://github.com/nextflow-io/rnaseq-nf).  


### 1. Run with Singularity

Setup: have `singularity` in the `PATH`, with all required variables, such as `SINGULARITY_BINDPATH`  

Run: `nextflow run main.nf -profile singularity`  


### 2. Run with Singularity-HPC (SHPC)

Setup:
1. (one-off) pre-install the relevant container modules:
    ```
    shpc install quay.io/biocontainers/salmon:1.6.0--h84f40af_0
    shpc install quay.io/biocontainers/fastqc:0.11.9--0
    shpc install quay.io/biocontainers/multiqc:1.11--pyhdfd78af_0
    ```
1. have `singularity` in the `PATH`, with all required variables, such as `SINGULARITY_BINDPATH`
2. have the container modules tree in `MODULEPATH`, e.g. using `module use`

Run: `nextflow run main.nf -profile shpc`


The pipeline uses the following bioinformatics software: 

* [Salmon](https://combine-lab.github.io/salmon/) 0.8.2
* [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) 0.11.5
* [Multiqc](https://multiqc.info) 1.0

