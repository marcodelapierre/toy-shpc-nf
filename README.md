## Demo RNAseq pipeline, to test interplay with SHPC container modules

The pipeline requires [Nextflow](https://github.com/nextflow-io/nextflow) to run, plus some additional tools:

* [Singularity-HPC](http://github.com/singularityhub/singularity-hpc), or SHPC, container modules utility
* [Singularity](http://singularity.hpcng.org) container runtime
* Software modules, i.e. [Lmod](https://lmod.readthedocs.io) or [Environment Modules](https://modules.readthedocs.io)

The Conda example requires only [Miniconda](https://docs.conda.io).  

**Credits**: the backbone of the pipeline (`main.nf`) and sample input data come from [nextflow-io/rnaseq-nf](https://github.com/nextflow-io/rnaseq-nf).  

## Usage

### 1. Setup

#### Install Modules

Most clusters will already have a module system installed, but if needed, here are basic steps to install environment modules. For this tutorial
it was installed it on a development server using these steps.

```bash
curl -LJO https://github.com/cea-hpc/modules/releases/download/v4.7.1/modules-4.7.1.tar.gz
tar xfz modules-4.7.1.tar.gz
cd modules-4.7.1
./configure
make
sudo make install
```

To ensure they are available at shell startup:

```bash
sudo ln -s /usr/local/Modules/init/profile.sh /etc/profile.d/modules.sh
```

The [singularity-hpc](https://github.com/singularityhub/singularity-hpc) repository has Dockerfile's and GitHub workflows that show additional examples.

#### Install shpc

Singularity HPC can be easily installed with a clone:

```bash
$ git clone https://github.com/singularityhub/singularity-hpc
$ cd singularity-hpc
$ pip install -e .
```

Ensure to configure it for your module software (default is lmod, here is changing to environment modules):

```bash
$ shpc config set module_sys:tcl
```

#### Install Singularity

You can [install singularity](https://sylabs.io/guides/3.9/user-guide/quick_start.html)
in many different ways! I usually choose from source.

```bash
$ git clone https://github.com/sylabs/singularity
$ cd singularity
$ ./mconfig
```

#### Install nextflow

Next (har har), you'll want to [install nextflow](https://www.nextflow.io/docs/latest/getstarted.html)

#### Clone workflow

And finally, you need the workflow!

```
$ git clone https://github.com/researchapps/demo-shpc-nf
$ cd demo-shpc-nf
```

### 2. Run with Singularity

First let's test just running the workflow with Singularity, which is probably our best bet since
it only requires the two dependencies of nextflow and singularity. Make sure you have `singularity` 
in the `PATH`, with all required variables, such as `SINGULARITY_BINDPATH`.
Then run:

```bash
$ nextflow run main.nf -profile singularity
```
```bash
Done! Open the following report in your browser --> results/multiqc_report.html

Completed at: 05-Feb-2022 15:13:48
Duration    : 8m 51s
CPU hours   : (a few seconds)
Succeeded   : 4
```

### 2. Run with Singularity-HPC (SHPC)

So why would you want to use shpc if it's one more dependency? Since we can install the containers as modules,
given a shared HPC module system, users can easily share them across workflows. Or if it's just you, you can do the same.
First, (one-off) install the container modules you need for shpc:

```bash
shpc install quay.io/biocontainers/salmon:1.6.0--h84f40af_0
shpc install quay.io/biocontainers/fastqc:0.11.9--0
shpc install quay.io/biocontainers/multiqc:1.11--pyhdfd78af_0
```

Since you just pulled these containers with singularity, they should be cached and the install quick!
You should still have `singularity` in the `PATH`, with all required variables, such as `SINGULARITY_BINDPATH`
Finally, tell the environment modules about your module directory.

```bash
$ module use ~/Documents/Code/singularity-hpc/modules
```
You should be able to see the modules available!

```bash
$ module avail
----------------------------------- /home/vanessa/Documents/Code/singularity-hpc/modules -----------------------------------
quay.io/biocontainers/fastqc/0.11.9--0/module.tcl            quay.io/biocontainers/salmon/1.6.0--h84f40af_0/module.tcl  
quay.io/biocontainers/multiqc/1.11--pyhdfd78af_0/module.tcl  

---------------------------------------------- /usr/local/Modules/modulefiles ----------------------------------------------
dot  module-git  module-info  modules  null  use.own  
```

Finally, let's run the workflow!

```bash
$ nextflow run main.nf -profile shpc
```


### 3. Run with Conda

Finally, here is how to run the same workflow with conda! You'll need to have the `conda` executable on your path,
and to add the correct channels:

```bash
conda config --add channels cctbx202112
conda config --add channels conda-forge
```

Then:

```bash
nextflow run main.nf -profile conda
```

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
