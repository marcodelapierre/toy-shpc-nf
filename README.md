## Demo RNAseq pipeline, to test interplay with SHPC container modules


The pipeline requires [Nextflow](https://github.com/nextflow-io/nextflow) to run, plus some additional tools:

* [Singularity-HPC](http://github.com/singularityhub/singularity-hpc), or SHPC, container modules utility
* [Singularity](http://singularity.hpcng.org) container runtime
* Software modules, i.e. [Lmod](https://lmod.readthedocs.io) or [Environment Modules](https://modules.readthedocs.io)

The Conda example requires only [Miniconda](https://docs.conda.io).  

**Credits**: the backbone of the pipeline (`main.nf`) and sample input data come from [nextflow-io/rnaseq-nf](https://github.com/nextflow-io/rnaseq-nf).  


### 1. Run with Singularity

Setup: have `singularity` in the `PATH`, with all required variables, such as `SINGULARITY_BINDPATH`  

Run: `nextflow run main.nf -profile singularity`  


### 2. Run with Singularity-HPC (SHPC)

Setup:
1. (one-off) Pre-install the relevant container modules:
    ```
    shpc install quay.io/biocontainers/salmon:1.6.0--h84f40af_0
    shpc install quay.io/biocontainers/fastqc:0.11.9--0
    shpc install quay.io/biocontainers/multiqc:1.11--pyhdfd78af_0
    ```
1. Have `singularity` in the `PATH`, with all required variables, such as `SINGULARITY_BINDPATH`
2. Have the container modules tree in `MODULEPATH`, e.g. using `module use`

Run: `nextflow run main.nf -profile shpc`  


### 3. Run with Miniconda

Setup:
1. Have `conda` available
2. (one-off) Add a couple of Conda channels, as required by some package dependencies:
    ```
    conda config --add channels cctbx202112
    conda config --add channels conda-forge
    ```

Run: nextflow run main.nf -profile conda`  


### Expected output

The run takes under a minute to complete (excluding container downlaod times, one-off).  
On successful completion of the four tasks, the following message is displayed:  
```
Done! Open the following report in your browser --> results/multiqc_report.html
```


### Implementation note

The key configuration difference between using Singularity, SHPC modules, or Conda are the keywords `process.container`/`process.module`/`process.conda`; the package names are almost a copy/paste, with only small caveats:
* between containers and modules, replace the colon `:` between container name and tag with a slash `/` in the module;
* between containers and conda, replace the colon `:` with an equal sign `=`, and the double dash `--` in the tag (between version and build) with another equal sign `=`.

Here is the relevant snippet from the `profiles` section of the configuration file `nextflow.config`:

```
  singularity {
    process {
      withName: 'index|quant' { container = 'quay.io/biocontainers/salmon:1.6.0--h84f40af_0' }
      withName: 'fastqc'      { container = 'quay.io/biocontainers/fastqc:0.11.9--0' }
      withName: 'multiqc'     { container = 'quay.io/biocontainers/multiqc:1.11--pyhdfd78af_0' }
    }
    singularity.enabled = true
    singularity.autoMounts = true
  }

  shpc {
    process {
      withName: 'index|quant' { module = 'quay.io/biocontainers/salmon/1.6.0--h84f40af_0' }
      withName: 'fastqc'      { module = 'quay.io/biocontainers/fastqc/0.11.9--0' }
      withName: 'multiqc'     { module = 'quay.io/biocontainers/multiqc/1.11--pyhdfd78af_0' }
    }
  }

  conda {
    process {
      withName: 'index|quant' { conda = 'bioconda::salmon=1.6.0=h84f40af_0' }
      withName: 'fastqc'      { conda = 'bioconda::fastqc=0.11.9=0' }
      withName: 'multiqc'     { conda = 'bioconda::multiqc=1.11=pyhdfd78af_0' }
    }
  }
```



