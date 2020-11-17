![Analysis pipeline](https://github.com/comprna/METEORE/blob/master/figure/meteore_logo_white.png)
# METEORE: MEthylation deTEction with nanopORE sequencing                                         :stars:

**About METEORE**

METEORE provides snakemake pipelines for various tools to detect DNA methylation from Nanopore sequencing reads. Additionally, it provides
new predictive models (random forest and multiple linear regression) that combine the outputs from the tools to produce a consensus prediction with higher accuracy than the individual tools.

----------------------------
# Table of Contents
----------------------------

   * [Pipeline](#pipeline)
   * [Installation](#installation)
   * [Tutorial on an example dataset](#tutorial-on-an-example-dataset)
      * [Nanopolish snakemake pipeline](#nanopolish-snakemake-pipeline)
         * [Create and activate the Conda environment](#create-and-activate-the-conda-environment)
         * [Run the snakemake](#run-the-snakemake)
         * [Prepare the input file for combined model usage (optional)](#prepare-the-input-file-for-combined-model-usage-optional)
      * [DeepSignal snakemake pipeline](#deepsignal-snakemake-pipeline)
         * [Create and activate the Conda environment](#create-and-activate-the-conda-environment-1)
         * [Run the snakemake](#run-the-snakemake-1)
         * [Prepare the input file for combined model usage (optional)](#prepare-the-input-file-for-combined-model-usage-optional-1)
      * [Tombo snakemake pipeline](#tombo-snakemake-pipeline)
         * [Create and activate the Conda environment](#create-and-activate-the-conda-environment-2)
         * [Run the snakemake](#run-the-snakemake-2)
         * [Prepare the input file for combined model usage (optional)](#prepare-the-input-file-for-combined-model-usage-optional-2)
      * [Guppy snakemake pipeline](#guppy-snakemake-pipeline)
         * [Modified basecallling](#modified-basecalling)
         * [Create and activate the Conda environment](#create-and-activate-the-conda-environment-3)
         * [Run the snakemake](#run-the-snakemake-3)
         * [Prepare the input file for combined model usage (optional)](#prepare-the-input-file-for-combined-model-usage-optional-3)
      * [Megalodon](#megalodon)
      * [DeepMod](#deepmod)
   * [Combined model (random forest) usage](#combined-model-random-forest-usage)
      * [Input file](#input-file)
      * [Command](#command)
      * [Per site predictions](#per-site-predictions)
      * [Train your own combination model](#train-your-own-combination-model)
   * [Combined model (multiple linear regression) usage](#combined-model-multiple-linear-regression-usage)
      * [Command and options](#command-and-options)
      * [Regression specific options](#regression-specific-options)
      * [Example run report](#example-run-report)



----------------------------
# Pipeline
----------------------------

![Analysis pipeline](https://github.com/comprna/METEORE/blob/master/figure/pipeline.png)
**Fig 1. Pipeline for CpG methylation detection form nanopore sequencing data**

----------------------------
# Installation
----------------------------
We recommend to install software dependencies via `Conda` on Linux. You can find Miniconda installation instructions for Linux [here](https://docs.conda.io/projects/conda/en/latest/user-guide/install/linux.html).
Make sure you install the [Miniconda Python3 distribution](https://docs.conda.io/en/latest/miniconda.html#linux-installers).
```
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```
Accept the license terms during installation.

Install `Mamba` via conda to install Snakemake for each pipeline later. See [Snakemake documentation](https://snakemake.readthedocs.io/en/stable/getting_started/installation.html) for more details.
```
conda install -c conda-forge mamba
```

Once you have installed Conda and Mamba, you can download the Snakemake pipelines and example datasets.
```
git clone https://github.com/comprna/METEORE.git
cd METEORE/
```
------------------------------------------
# Tutorial on an example dataset
------------------------------------------

We provide an example dataset `data/example` along with a genome reference `data/ecoli_k12_mg1655.fasta` for you to try the pipelines with. The example contains 50 single-read fast5 files from the positive control dataset for E.coli generated by [Simpson et al. (2017)](https://www.nature.com/articles/nmeth.4184).

**Run the pipelines with your own data:**
- You can run the pipeline with your own dataset by replacing `example` folder in the `data` directory with your folder containing the fast5 files. You will use the **fast5 folder name** to specify your target output file in the Snakemake pipeline. Simply replace ***example*** in the output file with ***your fast5 folder name*** in the command line below.
- You should place the reference genome file in *.fasta* format in a folder named `data`, and re-define the reference genome file within the Snakefile (`Nanopolish`, `Deepsignal`, `Tombo`, `Guppy`) by replacing `ecoli_k12_mg1655.fasta` with your specified reference genome.


## Nanopolish snakemake pipeline

### Create and activate the Conda environment

```bash
# Create an environment with Snakemake installed
mamba create -c conda-forge -c bioconda -n meteore_nanopolish_env snakemake
# Activate
conda activate meteore_nanopolish_env
# Install all required packages using conda
conda install -c bioconda nanopolish samtools r-data.table r-dplyr r-plyr
```

### Run the snakemake

Before executing the workflow below, make sure you have the basecalled fastq file in the `METEORE` directory. Nanopolish needs to link the read ids from the fastq file with their signal-level data in the fast5 files. An example fastq file `example.fastq` is provided.

A Snakefile named `Nanopolish` contains all rules for the Snakemake workflow. Run the snakemake to create the output files:
```
snakemake -s Nanopolish nanopolish_results/example_nanopolish-freq-perCG.tsv --cores all
```
This will produce four index files `example.fastq.index`, `example.fastq.index.fai`, `example.fastq.index.gzi` and `example.fastq.index.readdb`, and the `nanopolish_results` output directory containing all output files.
* `example_nanopolish-log.tsv` is the raw output after running `nanopolish call-methylation`.
* `example_nanopolish-log-perCG.tsv` contains per-read per-site data, which splits up the CpG group containing multiple nearby sites into its constituent CpG sites.
```
Chr           Pos         Strand    Log.like.ratio  Read_ID
NC_000913.3   3499494     +         -0.62           094dfe6b-23ed-4195-8876-805a399fade5
NC_000913.3   3499526     +         -0.33           094dfe6b-23ed-4195-8876-805a399fade5
NC_000913.3   3499546     +         -0.12           094dfe6b-23ed-4195-8876-805a399fade5
NC_000913.3   3499563     +         8.26            094dfe6b-23ed-4195-8876-805a399fade5
```
* `example_nanopolish-freq-perCG.tsv` stores the per-site data and provides the genomic position of the CpG site, the methylation frequency and the read coverage. The methylation calls from both strands are merged into a single strand.
```
Chr             Pos       Methyl_freq     Cov
NC_000913.3     3504395   0.95            19
NC_000913.3     3504402   0.95            19
NC_000913.3     3504420   0.875           8
NC_000913.3     3504429   0.875           8
```

### Prepare the input file for combined model usage (optional)

You can also generate the per-read prediction output in a format that can be used in METEORE to generate a consensus prediction (see further below).
```
snakemake -s Nanopolish nanopolish_results/example_nanopolish-perRead-score.tsv --cores all
```
The output is in .tsv format and contains four columns: `ID`,	`Pos`,	`Strand` and `Score`, e.g.:
```
ID                                      Pos       Strand    Score
2f43696e-70f0-42dd-b23e-d9e0ea954d4f    2687804   -         29.64
...
```

## DeepSignal snakemake pipeline

### Create and activate the Conda environment

```bash
# Create an environment with Snakemake installed
# Here we install Python 3.6 with older version of Snakemake so we can install tensorflow==1.13.1 later
conda create -n meteore_deepsignal_env python=3.6 snakemake=5.3.0
# Activate
conda activate meteore_deepsignal_env
# Install all required packages using pip
# We will run DeepSignal on CPUs so we install the CPU version of tensorflow
pip install deepsignal 'tensorflow==1.13.1' ont-tombo ont-fast5-api
```

Alternatively, you can run DeepSignal on a GPU-enabled machine. Then you can install the GPU version of tensorflow. Please check out [DeepSignal Github Page](https://github.com/bioinfomaticsCSU/deepsignal#Installation) for more information.

Please download DeepSignal's trained model `model.CpG.R9.4_1D.human_hx1.bn17.sn360.v0.1.7+.tar.gz` [here](https://drive.google.com/drive/folders/1zkK8Q1gyfviWWnXUBMcIwEDw3SocJg7P).
To extract the `model.CpG.R9.4_1D.human_hx1.bn17.sn360.v0.1.7+.tar.gz` to the `METEORE/data` directory:
```
tar xvzf model.CpG.R9.4_1D.human_hx1.bn17.sn360.v0.1.7+.tar.gz -C <path_to_METEORE/data_directory>
```

### Run the snakemake

Note:
- DeepSignal does not support multi-read fast5 files. Please use the `multi_to_single_fast5` command from the [ont_fast5_api package](https://github.com/nanoporetech/ont_fast5_api) to convert the fast5 files to single-read fast5 format before running the snakemake.
- Raw read fast5 files must contain basecall information from the fastq files. If not, please use the `tombo preprocess annotate_raw_with_fastqs` command from [tombo preprocess](https://nanoporetech.github.io/tombo/resquiggle.html)to add basecalls to the fast5 files

A Snakefile named `Deepsignal` contains all rules in the Snakemake workflow. Run the snakemake to create the output files:
```
snakemake -s Deepsignal deepsignal_results/example_deepsignal-freq-perCG.tsv
```
This will produce the `deepsignal_results` output directory containing all output files.
* `example_deepsignal-prob.tsv` contains the per-read results including the position of the CpG site, read ID, strand, methylated probability, unmethylated probability etc.
* `example_deepsignal-freq-perCG-raw.tsv` contains the default per-site results generated by a Python script *call_modification_frequency.py* provided by `DeepSignal`. The file contains 11 columns:
```
#chr          pos       strand    0-based_pos   prob_unmethy_sum    prob_methyl_sum   count_modified  count_unmodified  coverage  mod_freq  k_mer
NC_000913.3   3501290   +         3501290       1.947               5.053             7                0                 7        1.000     AAAAGCACCGTGGACTT
NC_000913.3   3501291   -         1140360       3.103               1.897             1                4                 5        0.200     AAAGTCCACGGTGCTTT
NC_000913.3   3501308   +         3501308       1.032               5.968             7                0                 7        1.000     CTGGTCACCGAAAATAT
NC_000913.3   3501309   -         1140342       1.493               3.507             3                2                 5        0.600     CATATTTTCGGTGACCA
```
* `example_deepsignal-freq-perCG.tsv` is the final per-site results containing the genomic position of the CpG site, methylation frequency and coverage.The methylation calls from both strands are merged into a single strand.
```
Chr             Pos       Methyl_freq     Cov
NC_000913.3     3501291   0.6             12
NC_000913.3     3501309   0.8             12
NC_000913.3     3501351   1               12
NC_000913.3     3501356   1               12
```

### Prepare the input file for combined model usage (optional)

You can also generate the per-read prediction output in a format that can be used in METEORE to generate a consensus prediction (see further below).
```
snakemake -s Deepsignal deepsignal_results/example_deepsignal-perRead-score.tsv
```
The output is in tsv format and contains four columns: `ID`,	`Pos`,	`Strand` and `Score`.

## Tombo snakemake pipeline

### Create and activate the Conda environment
```bash
# Create an environment with Snakemake installed
mamba create -c conda-forge -c bioconda -n meteore_tombo_env snakemake
# Activate
conda activate meteore_tombo_env
# Install all required packages using pip
pip install ont-tombo wiggelen ont-fast5-api
```

### Run the snakemake

Note:
- Tombo does not support multi-read fast5 files. Please use the `multi_to_single_fast5` command from the [ont_fast5_api package](https://github.com/nanoporetech/ont_fast5_api) to convert the fast5 files to single-read fast5 format before running the snakemake.
- Raw read fast5 files must contain basecall information from the fastq files. If not, please use the `tombo preprocess annotate_raw_with_fastqs` command from [tombo preprocess](https://nanoporetech.github.io/tombo/resquiggle.html)to add basecalls to the fast5 files

A Snakefile named `Tombo` contains all rules in the Snakemake workflow. Run the snakemake to create the output files:
```
snakemake -s Tombo tombo_results/example_tombo-freq-perCG.tsv --cores all
```
This will produce the following original modified base prediction results generated by `Tombo`. You can find the detailed information for all Tombo commands and outputs on the [tombo documentation page](https://nanoporetech.github.io/tombo/).
* `example.CpG.tombo.stats`: a binary Tombo statistics file
* `example.CpG.tombo.per_read_stats`: a per-read statistics file
* `example.fraction_modified_reads.plus.wig`: a wiggle output file which stores the raw fraction of significantly modified reads mapped on +'ve strand
* `example.fraction_modified_reads.minus.wig`: a wiggle output file which stores the raw fraction of significantly modified reads mapped on -'ve strand
* `example.valid_coverage.plus.wig`: a wiggle output file which stores the coverage data for reads on +'ve strand that are mapped, re-squiggled and outside the interval of the default thresholds
* `example.valid_coverage.minus.wig`: a wiggle output file which stores the coverage data for reads on -'ve strand that are mapped, re-squiggled and outside the interval of the default thresholds

There are rules in the Snakemake workflow for downstream processing of the output files generated by Tombo. The above command also generates a `tombo_results` output directory which contains the following output files:
* `example_tombo-freq-only.tsv`: contains methylation frequency results from the wiggle files
* `example_tombo-cov-only.tsv`: contains coverage data from the wiggle files
* `example_tombo-freq-perCG.tsv`: the final per-site results combining the methylation frequency and coverage results.
```
Chr             Pos         Methyl_freq     Cov
NC_000913.3     3501925     1               15
NC_000913.3     3501929     1               15
NC_000913.3     3501956     0.9375          9
NC_000913.3     3501977     0.86605         15

```

### Prepare the input file for combined model usage (optional)

You can also generate the per-read prediction output in a format that can be used in METEORE to generate a consensus prediction (see further below).
```
snakemake -s Tombo tombo_results/example_tombo-perRead-score.tsv --cores all
```
The output is in tsv format and contains four columns: `ID`,	`Pos`,	`Strand` and `Score`.

**Note:** This command can only extract per-read data for a region of interest. You may need to go to ***script_in_snakemake/extract_tombo_per_read_results.py*** to specify your region of interest (chromosome, start position and end position) in the Python script and rerun the results.
Open the Python script ***extract_tombo_per_read_results.py*** with an editor of your choice and go to line 18 to 24 of the file:
```bash
# specify region of interest (plus strand) below:
reg_data_plus = tombo_helper.intervalData(
    chrm='NC_000913.3', start=412305, end=4584088, strand="+")

# specify region of interest (minus strand) below:
reg_data_minus = tombo_helper.intervalData(
    chrm='NC_000913.3', start=412305, end=4584088, strand="-")
```

## Guppy Snakemake pipeline

### Modified basecallling

You need to basecall with the standalone Guppy basecaller before running the Snakemake pipeline. The pipeline was only designed to process and analyse the Guppy's fast5 output using the open-source custom scripts available from https://github.com/kpalin/gcf52ref.

Guppy basecaller is only available to Oxford Nanopore Technologies' customers via the community site. Please check the [community page](https://community.nanoporetech.com/downloads) for download and installation instructions.

Once you have installed Guppy, you can perform modified basecalling from the signal data using the `dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg` guppy config.
```bash
# For Guppy (CPU on Linux)
./<path_to_ont-guppy-cpu>/bin/guppy_basecaller --config dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg --fast5_out --input_path data/example/ --save_path guppy_results/ --cpu_threads_per_caller 10
# For Guppy (GPU on Linux)
./<path_to_ont-guppy-gpu>/bin/guppy_basecaller --config dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg --fast5_out --input_path data/example/ --save_path guppy_results/ --device auto
```
Once you have done modified basecalling with Guppy, the output files will be saved to the `guppy_results` directory where you will find the logs, basecalled fastq file(s) and a folder named `workspace` which contains basecalled fast5 files with modified base information.

### Create and activate the Conda environment

Before running the Snakmake pipeline, you need to prepare the following files:
* If you have more than 1 fastq files generated, please concatenate these files first
```
cat *.fastq > example.fq
```
* If there is only one fastq file, rename that file
```
mv [original_fastq_file_with_long_filename] example.fq
```
* Go to `guppy_results` directory and download the scripts that convert CpG methylation from fast5s to reference anchored calls
```
cd guppy_results/
git clone https://github.com/kpalin/gcf52ref.git
```

Then you can create the environment from the `guppy.yml` file:
```
conda env create -f guppy.yml
```
Then activate the Conda environment:
```
conda activate guppy_cpg_snakemake
```

### Run the snakemake

A Snakefile named Guppy contains all rules in the Snakemake workflow. Run the snakemake to create the output files:
```
snakemake -s Guppy guppy_results/example_guppy-freq-perCG.tsv
```
This will produce two output files in the `guppy_results` directory:
* `example_guppy-log-perCG.tsv` contains per-read per-site data.
```
chromosome    strand  start     end       read_name                               log_lik_ratio   log_lik_methylated    log_lik_unmethylated    num_calling_strands   num_motifs  sequence
NC_000913.3   +       3505711   3505712   8386596c-ff56-4032-b54e-8f062c194b16    -2.41           -2.41                 0.0                     1                     1           CG
NC_000913.3   +       3505714   3505715   8386596c-ff56-4032-b54e-8f062c194b16    -0.575          -0.676                -0.101                  1                     1           CG
NC_000913.3   +       3505726   3505727   8386596c-ff56-4032-b54e-8f062c194b16    1.49            -0.012                -1.51                   1                     1           CG
NC_000913.3   +       3505728   3505729   8386596c-ff56-4032-b54e-8f062c194b16    0.361           -0.155                -0.516                  1                     1           CG
NC_000913.3   +       3505745   3505746   8386596c-ff56-4032-b54e-8f062c194b16    -0.112          -0.359                -0.247                  1                     1           CG
```
* `example_guppy-freq-perCG.tsv` contains the per-site data including the genomic position of the CpG site, methylation frequency and coverage. The methylation calls from both strands are merged into a single strand.
```
Chr             Pos       Methyl_freq   Cov
NC_000913.3     3501544   0.875         14
NC_000913.3     3501549   0.83333       13
NC_000913.3     3501561   0.70833       14
NC_000913.3     3501573   0.5571        12
```

### Prepare the input file for combined model usage (optional)

You can also generate the per-read prediction output in a format that can be used in METEORE to generate a consensus prediction (see further below).
```
snakemake -s Guppy guppy_results/example_guppy-perRead-score.tsv
```
The output is in tsv format and contains four columns: `ID`,	`Pos`,	`Strand` and `Score`.

## Megalodon

We did not develop a snakemake pipeline for Megalodon as it can be run with a single command.

Please check out [Megalodon GitHub Page](https://github.com/nanoporetech/megalodon) for more details. Note that the new release of Megalodon requires Guppy basecaller to be installed to run Megalodon for the basecalling step. It uses Guppy (GPU on Linux) by default.
```
megalodon data/example/ --outputs mods --reference data/ecoli_k12_mg1655.fasta --mod-motif Z CG 0 --write-mods-text --processes 10 --guppy-server-path /<path_to_ont-guppy-gpu>/bin/guppy_basecall_server --guppy-params "--num_callers 5 --ipc_threads 6" --overwrite
```
However, you can use Megalodon with Guppy (CPU on Linux), but you need to specify the `--guppy-timeout` argument to allow sufficient time for calling a read during CPU basecalling. Here we use 200 seconds.
```
megalodon data/example/ --outputs mods --reference data/ecoli_k12_mg1655.fasta --mod-motif Z CG 0 --write-mods-text --processes 10 --guppy-server-path ./<path_to_ont-guppy-cpu>/bin/guppy_basecall_server --guppy-params "--num_callers 10" --guppy-timeout 200 --overwrite
```

This will produce the `megalodon_results` directory which contains logs, per-read modified base output `per_read_modified_base_calls.txt`, per-site modified base output `modified_bases.5mC.bed`.

We provide a bash script to generate the methylation frequency file and the input file for combined model usage, which are in the same format used in every methods. Run the bash script with ***your preferred filename*** (eg. example) for the output files:
```
./script/megalodon.sh example
```
You will get the following files:
* `example_megalodon-freq-perCG.tsv` contains the per-site data including the genomic position of the CpG site, methylation frequency and coverage. The methylation calls from both strands are merged into a single strand.
* `example_megalodon-perRead-score.tsv` is a per-read prediction file containing four columns: `ID`,	`Pos`,	`Strand` and `Score`. This file can be used later in METEORE to generate a consensus prediction.


## DeepMod

We did not develop a snakemake pipeline for DeepMod as it can be run with a single command.

Please check out [DeepMod GitHub Page](https://github.com/WGLab/DeepMod) for installation instructions and usage.
```bash
# Set the working directory where you install DeepMod
python bin/DeepMod.py detect --wrkBase <path_to_METEORE>/data/example/ --Ref <path_to_METEORE>/data/ecoli_k12_mg1655.fasta --outFolder <path_to_METEORE>/deepmod_results --Base C --modfile train_mod/rnn_conmodC_P100wd21_f7ne1u0_4/mod_train_conmodC_P100wd21_f3ne1u0 --FileID example_deepmod --threads 10
```
You will find a `deepmod_results` output folder which contains a *.done* file and a `example_deepmod` result folder. Inside the `example_deepmod` folder, you will find two output files in a bed format `mod_pos.NC_000913.3-.C.bed` and `mod_pos.NC_000913.3+.C.bed`, which per-site data including genomic position of the CpG site, strand, base, coverage, methylation percentage and methylation coverage. The format of the output of DeepMod is described [here](https://github.com/WGLab/DeepMod/blob/master/docs/Results_explanation.md).

We provide a R script to generate a methylation frequency file in the same format used in other methods. Run `Rscript script/run_deepmod.R deepmod_output_4_plus_strand.bed deepmod_output_4_minus_strand.bed output_file.tsv`
```bash
# Set the working directory where you install METEORE
Rscript script/run_deepmod.R deepmod_results/example_deepmod/mod_pos.NC_000913.3+.C.bed deepmod_results/example_deepmod/mod_pos.NC_000913.3-.C.bed example_deepmod-freq-perCG.tsv
```
The `example_deepmod-freq-perCG.tsv ` output file contains the per-site data including the genomic position of the CpG site, methylation frequency and coverage. The methylation calls from both strands are merged into a single strand.

Note that DeepMod does not produce per-read predictions, so we could not produce an output file in the same format as those described above (.tsv) to be later combined to build a consensus.

--------------------------------------
# Combined model (random forest) usage
--------------------------------------

We have trained Random Forest models that combine the outputs from two of the methods above
to produce consensus predictions with improved accuracy. We also provide the scripts to train new models with two
or more methods.

First, please make sure you install the required libraries in the conda env

```
pip install -r requirements.txt
```

## Input file
To make the predictions using the combination model (eg. ***deepSignal*** and ***nanopolish***), you can generate the per-read prediction input
file `example_deepsignal-perRead-score.tsv` and `example_nanopolish-perRead-score.tsv` as described before in our snakemake pipelines.
The input (.tsv) file from each method must be formatted as below:
```
ID                                      Pos       Strand    Score
2f43696e-70f0-42dd-b23e-d9e0ea954d4f    2687804   -         29.64
7ee0a989-c750-4dde-9114-354b97996dae    3104781   -         -4.47
dc9dcb55-703c-4251-a916-4214abd67991    1173719   +         5.34
2bea7f2a-f76c-491a-b5ee-b22c6f4a8539    1864274   -         5.33
```
where the score is the significance score given by each method.
Nanopolish and Guppy provide a log-likelihood ratio value per site and per read. Similarly, Tombo produces a significance value per site and per read.
On the other hand, for DeepSignal we give the log-ratio of the probabilities for a site to be methylated over the probablity of being unmethylated for each read.
Similarly, for Megalodon we used the difference of the log-probabilities to obtain a log-ratio.

## Command
Given the already pre-trained model (models available in `./saved_models/`), to run the model you need the following command:
```
python combination_model_prediction.py  -i [path of tsv file containing the methods' names and the paths to the input file] -m [model_to_use (default or optimized)] -o [output_file]
```
**Inputs (-i)**
The file for option `-i` contains the methods you choose to use (1st column) and the path to the respective per-read score files (2nd column):
```
deepsignal	./example_results/deepsignal_results/example_deepsignal-perRead-score.tsv
nanopolish	./example_results/nanopolish_results/example_nanopolish-perRead-score.tsv
```
Here we call this file `model_content.tsv` (provided as an example with the package) and use the results `example_results/` produced by the method from the example dataset.

**Models (-m)**
You can choose to use either `-m default` or `-m optimised`:
```bash
# Use the default model
python combination_model_prediction.py -i model_content.tsv -m default -o example_deepsignal_nanopolish-default-model-perRead.tsv
# Use the optimized model
python combination_model_prediction.py -i model_content.tsv -m optimized -o example_deepsignal_nanopolish-optimized-model-perRead.tsv
```
* `default` refers to the parameters used to build the Random Forest, which in this case are the default parameters from the [sklern library](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html) (n_estimator = 100 and max_dep = None). The above command will use the model named `rf_model_default_deepsignal_nanopolish.model` to score the sites and reads that are common
between the two files `example_deepsignal-perRead-score.tsv` and `example_nanopolish-perRead-score.tsv`.

* `optimised` refers to the adjusted parameters for n_estimator and max_dep used to build the Random Forest from sklern (n_estimator = 3 and max_dep = 10). The model used in the above command will be `rf_model_max_depth_3_n_estimator_10_deepsignal_nanopolish.model`.

**Important note:**
1. If you want a different combination, e.g. deepsignal+guppy, you can replace the "nanopolish" line in the `model_content.tsv` file with guppy and its input file path. You can also add other methods paths as well in the same file.
2. The order of method's names in the model_content.tsv file should be same as the order used to generate the combined model. For example, we provide the models with name *'rf_model_default_**deepsignal_nanopolish**.model'* so the order in the *model_content.tsv* file should be **deepsignal and then nanopolish** and not the other way round.

**Output (-o)**
Use **-o** option to write the per-read consensus prediction output to the specified file.
All outputs are written into the directory called `combined_model_results`. New results from subsequent runs will be saved into
the same output directory.

The output will contain per-read predictions for the reads common to both deepsignal and nanopolish method. For example, in the `example_deepsignal-nanopolish-optimized-model-perRead.tsv`
```
ID	                                   Pos	      Prediction	  Prob_methylation
c3e4c0c6-11f9-4ecb-858e-877e81e31a0c	 3502752	  0	            0.1621034590408923
c3e4c0c6-11f9-4ecb-858e-877e81e31a0c	 3502750	  0	            0.041931319736462455
c3e4c0c6-11f9-4ecb-858e-877e81e31a0c	 3502735	  0	            0.49939245262344834
c3e4c0c6-11f9-4ecb-858e-877e81e31a0c	 3502715	  0	            0.041931319736462455
...
```
The column `prediction` (0 refers to unmethylated and 1 refers to methylated) is based on a threshold of 0.5 score. That is, if the P(methylation) is <= 0.5,
it is predicted as unmethylated (0), otherwise as methylated (1). Predictions for a different thresholds can be obtained by parsing the column `Prob_methylation`.


## Per-site predictions

We also provide a Python script to convert the per-site predictions for each individual read (methylated / unmethylated) into per-site predictions at genome level (methylation frequency) by summarizing the predictions on individual reads from the model.
```
prediction_to_mod_frequency.py [-h] -i INPUT [-t THRESHOLD] -o OUTPUT
```
where the input is the file produced from the previous step, and the output is the per-site result at genome level. You can specify a threshold which applies on the *Prob_methylation* column in the per-read combined prediction file (see above) to define the site on the read as methylated or unmethylated.

Run `python prediction_to_mod_frequency.py -h` for help.

For example:
```bash
# You can specify a threshold when -t option is used
python prediction_to_mod_frequency.py -i combined_model_results/example_deepsignal_nanopolish_optimized_model_perRead.tsv -t 0.46 -o combined_model_results/example_deepsignal-nanopolish-optimized-model-freq-perCG.tsv
# Or you can use the default threshold of 0.5
python prediction_to_mod_frequency.py -i combined_model_results/example_deepsignal_nanopolish_default_model_perRead.tsv -o combined_model_results/example_deepsignal-nanopolish-default-model-freq-perCG.tsv
```
The per-site prediction output contains the following fields:
```
Pos	        Unmodified_read	  Modified_read	  Coverage      Methylation_frequency
3507262     3                 12              15            0.8
3507265     3                 12              15            0.8
3507269     2                 13              15            0.8666666666666667
3507274     4                 11              15            0.7333333333333333

```

## Train your own combination model

We provide the script to train a combined model from the per-read and per-site output from any number of methods (from 2 to 5). For instance, the command to train a model with 5 methods would be:
```
python combination_model_train.py  -d [path of deepsignal file] -n [path of nanopolish file] -g [path of guppy file] -m [path of megalodon file] -t [path of tombo file] -c [number of methods to combine together for training (range from 2-5)] -o [output_path_to_save_model]
```
The order of the methods in the command is not relevant, but it will determine how to specify the model when making predictions, as described above.
For example, the command to train a model with DeepSignal, Nanopolish, and guppy, could take the following form:
```
python combination_model_train.py  -d deepsignal.tsv -n nanopolish.tsv -g guppy.tsv -c 3 -o output_models
```
This command will create an output directory called `output_models`, and the models named `rf_model_default_deepsignal_nanopolish_guppy.model` and `rf_model_max_depth_3_n_estimator_10_deepsignal_nanopolish_guppy.model` will be saved inside this directory. To run this new model, you will need to create a new file
`new_model_content` of the form, e.g.:
```
deepsignal    deepsignal_test2.tsv
nanopolish    nanopolish_test2.tsv
guppy         guppy_test2.tsv
```
where the `_test2.tsv` files contain the reads and scores that you want to use for consensus prediction in the format
```
ID                                    Pos       Strand  Score
b9fdd6aa-ba93-4424-8f4b-c632e4d16d2e  1817032    -      2.45
...
```
The command to obtain the consensus predictions could be of the form
```
python combination_model_prediction.py  -i new_model_content.tsv -m optimized -o output2
```
This command will use the model `rf_model_max_depth_3_n_estimator_10_deepsignal_nanopolish_guppy.model` to predict on the sites and reads common in the input files
`deepsignal_test2.tsv`, `nanopolish_test2.tsv`, and `guppy_test2.tsv`.

---------------------------------------------------
# Combined model (multiple linear regression) usage
---------------------------------------------------

An alternative combination model based on multiple linear regression (REG) is available through the meteore_reg.py script. This is
a standalone script that first builds a model training data and then applies it to unknown test data generated from the same tools.
It a simple script to run which requires upwards of 12 GB of RAM, and approximately five minutes for the data sets used in our publication.

## Command and options

The example usage below would train the REG model on Nanopolish and DeepSignal outputs from the "mix1" data set, and apply 
this model to the data in "mix2" for the same tools. Inputs and outputs rely on the ordering of arguments to remain consistent. 

```
python meteore_reg.py --train set1_nanopolish.tsv set1_deepsignal.tsv \
      --tool_names nanopolish deepsignal --train_desc mix1 --test_desc mix2 \
      --test set2_nanopolish.tsv set2_deepsignal.tsv --testIsTraining \
      --trunc_min -50 -5 --trunc_max 50 5 --filter 0.1
```
### The following command-line options are available:
```
python meteore_reg.py -h
usage: meteore_reg.py [-h] --train TRAIN [TRAIN ...] --train_desc TRAIN_DESC [--test [TEST [TEST ...]]]
                      [--test_desc TEST_DESC] --tool_names TOOL_NAMES [TOOL_NAMES ...]
                      [--trunc_max [TRUNC_MAX [TRUNC_MAX ...]]] [--trunc_min [TRUNC_MIN [TRUNC_MIN ...]]] [--debug]
                      [--inc_pred] [--filter FILTER] [--testIsTraining] [--no_scaling]

optional arguments:
  -h, --help            show this help message and exit
  --train TRAIN [TRAIN ...]
                        Path to training data
  --train_desc TRAIN_DESC
                        Describe the training data set
  --test [TEST [TEST ...]]
                        Path to test data. Must be omitted or same number of tool files as as --train (and in the same order)
  --test_desc TEST_DESC
                        Describe the test data set
  --tool_names TOOL_NAMES [TOOL_NAMES ...]
                        Names of each tool in the same order as given for --train (and --test if provided)
  --trunc_max [TRUNC_MAX [TRUNC_MAX ...]]
                        Truncate maximum values. Must match order of training data
  --trunc_min [TRUNC_MIN [TRUNC_MIN ...]]
                        Truncate minimum values. Must match order of training data
  --debug               Turn on debugging output (STDERR)
  --inc_pred            Include tool prediction categories in regression. NOTE: must be present in all input files
  --filter FILTER       Percentage of total reads to remove around mean score, default -1 (disabled)
  --testIsTraining      Use functions for handling training data for the test parameters
  --no_scaling          Disable score scaling
```
# Regression specific options

### Several arguments are specific to the REG model and deserve further discussion.

#### --trunc_min INT ... --trunc_max INT ...
The trunc_min and trunc_max parameters can be used to restrict the useful range of log difference scores on a per-tool basis. 
Most of the existing methylation calling tools are well behaved but Nanopolish in particular can output extreme scores. By 
default REG scales each tool's scores linearly between 0 and 1. Extreme positive or negative values will reduce the 
contribution by scores that are less extreme but still meaningful indictors of methylation status. Values less than 
trunc_min INT for tool N will be replaced with the user provided score, and vice versa for the scores larger than trunc_max INT.

#### --inc_pred
Enables methylation labels to be considered as a contributor to the prediction model. This requires the test data to have the same format as the test data, with "group" and "label" columns. In some case it can improve prediction if the tool makes good classifications in spite of oddly distributed scores.

#### --filter FLOAT 0..1
Filters out a fixed proportion of reads, evenly distributed either side of the scaling mid-point. If scaling is disabled then the mean score is considered the mid-point.

#### --testIsTraining
Informs REG to use the input format corresponding to training data for test data as well.

#### --no_scaling
Disables MinMax (0..1) scaling. Not recommended in general but might be necessary for tools with wildly different scoring systems.

## Example run report
```
python meteore_reg.py --train set1_deepsignal.tsv.gz set1_nanopolish_updated.tsv.gz --tool_names deepsignal nanopolish --train_desc mix1 --test set2_deepsignal.tsv.gz set2_nanopolish.tsv.gz --test_desc mix2 --testIsTraining --trunc_min -5 -50 --trunc_max 5 50
Filter  Ignored deepsignal_pass deepsignal_prop nanopolish_pass nanopolish_prop Model_succ      Model_prop
-1      0       2981104 0.9105415715991803      2729348 0.8336457961081127      3029092 0.9251989163070138
Filter  Ignored deepsignal_pass deepsignal_prop nanopolish_pass nanopolish_prop Model_succ      Model_prop
-1      0       5728226 0.9151707857313122      5332267 0.8519103436420188      5778967 0.923277428318178
```

### REG reports a standard summary of the training and test accuracy.

Filter is the proportion of (presumably low-accuracy) reads to be removed, with -1 indicated it is disabled. Likewise the Ignored column is the corresponding number of reads removed by filtering.
Each tool is then reported with the number of site observations that match the expected methylation status (not accurate for hemi-methylated sites), and the overall proportion accuracy.
Finally the combined regression model numbers are reported for training data where labels are available to ascertain truth.
