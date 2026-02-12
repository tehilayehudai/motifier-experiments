This repository provides pipelines that implement the computational analysis for the following study:

Yehudai Tehila, Gaik Tamazian, Lakshminarasaiah Uppalapati, Sandra Völs, Saranya Sridhar, Guadalupe Cortés, Thorsten U. Vogel, Anna Roitburd-Berman, and Jonathan M. Gershoni.
Serum-antibody profiling of H3N2-infected ferrets using a combinatorial phage-display random peptide library.(in preparation)

The pipelines reimplement and extend the computational analysis described in the following paper:

Ashkenazy Haim, Oren Avram, Arie Ryvkin, Anna Roitburd-Berman, Yael Weiss-Ottolenghi, Smadar Hada-Neeman, Jonathan M. Gershoni, and Tal Pupko.
“Motifier: An IgOme Profiler Based on Peptide Motifs Using Machine Learning.”
[Journal of Molecular Biology 433, no. 15 (2021): 167071](https://doi.org/10.1016/j.jmb.2021.167071)

### Repository structure
```motifier-experiments/
├── input/
│   └── groups/        # Sample-to-barcode mapping files (sample2BC)
├── scripts/           # Jupyter notebooks for motif pruning and PCA analysis
└── Snakefiles/        # Snakemake workflows
```

The `Snakefiles/` directory contains three Snakemake workflows corresponding to:

Initial motif selection (400 motifs)
Manual motif pruning
Principal component analysis (PCA)

The `scripts/` directory contains Jupyter notebooks documenting the manual pruning process and PCA loadings analysis.

The `input/` directories contain files for demultiplexing sequenced reads, but do *not* contain the read FASTQ files. Users should download and copy the FASTQ files to the directories before running the pipelines (need to add a link to NCBI))

After successful execution of the pipelines, output files will be written to directories `data/` and `output/` in the pipeline directories.

# Installation

We recommend establishing a separate computational environment for running the pipelines. The pipelines are implemented in the [Snakemake](https://snakemake.readthedocs.io) workflow management system and require the [seaborn](https://seaborn.pydata.org) package for plotting heatmaps. The deep panning data analysis is implemented in the three following packages.

1. [motifier](https://github.com/gtamazian/motifier) provides routines for read translation into peptides and motif search
2. [pssm-tools](https://github.com/gtamazian/pssm-tools) implements computationally exhaustive routines for merging motifs, calculating motif cutoffs, and assigning peptides to motifs
3. [model-fitting](https://github.com/tehilayehudai/model-fitting) trains the random forest model, identifies biologically relevant motifs, and visualizes the motif-based prediction of a biological condition in question.

## Input FASTQ files

Input FASTQ files for the pipeline are available in the ???). The files should be downloaded to the `input/` directory in the pipeline directories. Note that the FASTQ files should be copied in their original (gzip-compressed) form and should *not* be uncompressed.

## Establishing environment

The computational environment can be conveniently established using the [Anaconda](https://anaconda.org) package manager.

```
conda create --yes --name motifier-legacy --channel conda-forge --channel bioconda python snakemake seaborn
conda activate motifier-legacy
pip install git+ssh://git@github.com/gtamazian/motifier.git
pip install git+https://github.com/tehilayehudai/model-fitting.git
```

## Compiling pssm-tools

Executable files from the [pssm-tools](https://github.com/gtamazian/pssm-tools) package should be compiled separately and copied into the `bin/` subdirectory in the pipeline directories. For the sake of simplicity, we assume that repositories of the pipeline and the executable files are cloned to the home directory.

```
cd
git clone git@github.com:gtamazian/motifier-pipeline.git
git clone git@github.com:gtamazian/pssm-tools.git
```

The executable files can be conveniently compiled using [CMake](https://cmake.org):

```
cd
cd pssm-tools
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --target package --parallel
```

Next, the compiled files are copied into the pipeline directories.

```
cd
cd motifier-pipeline/exp12
mkdir bin && cd bin && tar xvzf ~/pssm-tools/build/pssm-tools-0.1.2-Linux.tar.gz
cd
cd motifier-pipeline/exp22
mkdir bin && cd bin && tar xvzf ~/pssm-tools/build/pssm-tools-0.1.2-Linux.tar.gz
cd
```

## External programs

The pipeline requires two external programs: [CD-HIT](https://sites.google.com/view/cd-hit), which is the sequence clustering tool, and [MAFFT](https://mafft.cbrc.jp/), which is the multiple alignment program. Both tools are available in [Bioconda](https://bioconda.github.io/):

```
conda install --channel bioconda cd-hit mafft
```

Many Linux distributions also provide packages for both programs. For example, the programs can be installed in Ubuntu using its default package manager:

```
sudo apt install cd-hit mafft
```

To reproduce the published study, the specific version of MAFFT is required (version `v7.149b`). By default, the legacy sequence alignment tool from the motifier package (tool `motifier legacy align-sequences`) checks the MAFFT version and fails if the version differs from `v7.149b`. To disable the version check, a user can pass the command-line option `--ignore-version` to the tool.

## Checking environment

To check the computational environment installation, a user can run Snakemake in the dry mode:

```
conda activate motifier-legacy
cd
cd motifier-pipeline/exp12
snakemake --cores 1 --dry-run
```

# Running the pipeline

To run the pipeline, activate the computational environment, change the current directory to the pipeline directory, and launch Snakemake with the specified number of cores to be allocated to the pipeline routines.
Since multiple Snakefiles are provided (corresponding to motif selection, manual pruning, and PCA), the -s flag must be used to indicate which workflow to execute.
```
conda activate motifier-legacy
cd
cd motifier-pipeline/exp12
snakemake -s Snakefiles/FILE_NAME --cores all
```

Specifying `--cores all` will use all available cores of a machine. The exact number can be specified instead of `all`.
