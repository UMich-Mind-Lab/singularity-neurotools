# singularity-neurotools

## Quick Start

```
singularity build spm12.sif spm12.def
singularity run -B /path/to/data:/data spm12-mcr.sif <script> [script_inputs]
```

```
singularity build neurotools.sif neurotools.def
./neurotools.sif fslhd /path/to/bold.nii.gz
```

## Description

This repository hosts container definition files for containers used in MiND lab processing pipelines. Details on each container are available below.

### Neurotools.def

Because some neuroimaging tools contain wrappers around others (e.g., mrtrix3 and ICA_AROMA), I decided to include each of these tools in one singularity container. This container has only been used in the context of the neuroimaging pipelines used in the [MiND lab](https://sites.lsa.umich.edu/mindlab/) at the University of Michigan.

This container installs [FSL](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki), [AFNI](https://afni.nimh.nih.gov/), [ANTS](https://github.com/ANTsX/ANTs), [MRTRIX3](https://www.mrtrix.org/), and [ICA_AROMA](https://github.com/maartenmennes/ICA-AROMA)

I have not tested each package individually for full functionality and this container is a work in progress. When I have time I intend to work on decreasing the container size by removing unnecessary graphical functionality or template files, but as of now the container is largely each tool installed with minimal changes.

The container is set to run any command you offer it, whether it be accessing a single tool, such as

```
./neurotools.sif 3dclustsim [OPTIONS]
```

or to run a wrapper script:

```
./neurotools.sif my_fsl_preproc_wrapper.sh
```

With wrapper functions, be sure to pay attention to the paths needed by the scripts. The singularity container can only access relative paths available in your current `$PWD`. You'd need to specify binding points in both the definition file and in your call to the container created from it. There is a generic /data mount point available for this purpose:

```
singularity run -B /path/to/data:/data eddy -imain /data/path/to/data ...
```

##### Note on FSLOUTPUTTYPE

Users can specify their `FSLOUTPUTTYPE` in the definition file, but you will run into errors with ICA_AROMA if it is not set to `NIFTI_GZ`, as ICA_AROMA expects compressed output format in its various FSL wrapper functions. The mrtrix3 wrappers do not appear to have this same limitation. In the future it may be worth branching ICA_AROMA and having it read in `FSLOUTPUTTYPE`, but it's not an issue that's been too inconvenient up to this point.

### spm12.def

This container hosts a [standalone version of spm12](https://en.wikibooks.org/wiki/SPM/Standalone) along with the corresponding [matlab runtime libraries](https://www.mathworks.com/products/compiler/matlab-runtime.html).

The container entrypoint moves the user into the mounted `/data` directory, and then tells the SPM container to run the input script along with any options provided afterwards. In most cases, one can use the `scripts/spm_jobman_run.m` provided in this repository, but in cases where the user wants to access components outside of spm_jobman (such as using one of the many built-in toolboxes), they would need to write and specify another script. The file `scripts/spm_fm2vdm.m` provides an example for how one could use the built-in fieldmap toolbox to produce a voxel displacement map using this toolbox.

### conn.def

This container uses an [spm12 standalone container precompiled with CONN toolbox](https://www.nitrc.org/projects/conn) along with the corresponding [matlab runtime libraries](https://www.mathworks.com/products/compiler/matlab-runtime.html).

This container is incomplete, as I currently need to figure out how to use the conn_batch() tool in the standalone container. Hopefully this message won't be here long as I'll have the time to put this together and replace this with some better text :+1:

### neuropointillist.def

Container for [neuropointillist](https://ibic.github.io/neuropointillist/).

The container installs explicit dependencies of neuropointillist along with `lnme`. If one wishes to use other R packages, they would need to update the `neuropointillist.def` file to include those installations. There is a general use mount point `/data`, and the runscript for the package just takes one's command line arguments after calling singularity

## TO DO:

- [ ] compile standalone spm12 with CONN, [the Generalized PPI toolbox](https://www.nitrc.org/projects/gppi) myself so we ultimately use one spm based container.
- [ ] ICA_AROMA to handle the different options for FSLOUTPUTTYPE, suppress graphical elements from causing warnings/errors
- [ ] FSL/AFNI/ANTs cleaning to reduce size
- [ ] Look into AFNI's R scripts and make sure they work correctly (currently I just use tools like 3dclustsim, 3dUnifize, and 3dResample, and there's an error related to running `rPkgsInstall` when building the singularity container that I haven't gotten into yet)
