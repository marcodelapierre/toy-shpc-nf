## Demo RNAseq pipeline, to test interplay with SHPC container modules

The pipeline requires [Nextflow](https://github.com/nextflow-io/nextflow) to run, and has some additional tool requirements:

* [Singularity-HPC](http://github.com/singularityhub/singularity-hpc), or SHPC, container modules utility
* [Singularity](http://singularity.hpcng.org) container runtime
* Software modules, i.e. [Lmod](https://lmod.readthedocs.io) or [Environment Modules](https://modules.readthedocs.io)

*Credits*: the backbone of the pipeline (`main.nf`) and sample input data come from [nextflow-io/rnaseq-nf](https://github.com/nextflow-io/rnaseq-nf).  





The pipeline uses the following bioinformatics software: 

* [Salmon](https://combine-lab.github.io/salmon/) 0.8.2
* [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) 0.11.5
* [Multiqc](https://multiqc.info) 1.0

