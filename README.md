![Analysis pipeline](https://github.com/comprna/METEORE/blob/master/figure/meteore_logo_white.png)
# METEORE: MEthylation deTEction with nanopORE sequencing                                         :stars:

**About METEORE**

METEORE provides snakemake pipelines for various tools to detect DNA methylation from Nanopore sequencing reads. Additionally, it provides
a new predictive model that combines the outputs from the tools to produce a consensus prediction with higher accuracy than the individual tools.

----------------------------
# Table of Contents
----------------------------

   * [Pipeline](#pipeline)
   * [Installation](#installation)
   * [Tutorial on an example dataset](#tutorial-on-an-example-dataset)
      * [Nanopolish snakemake pipeline](#nanopolish-snakemake-pipeline)
            * [Create and activate the Conda environment](#create-and-activate-the-conda-environment-1)
            * [Run the snakemake](#run-the-snakemake-1)
            * [Prepare the input file for combined model usage (optional)](#prepare-the-input-file-for-combined-model-usage-optional-1)
      * [DeepSignal snakemake pipeline](#deepsignal-snakemake-pipeline)
            * [Create and activate the Conda environment](#create-and-activate-the-conda-environment-2)
            * [Run the snakemake](#run-the-snakemake-2)
            * [Prepare the input file for combined model usage (optional)](#prepare-the-input-file-for-combined-model-usage-optional-2)
      * [Tombo snakemake pipeline](#tombo-snakemake-pipeline)
            * [Create and activate the Conda environment](#create-and-activate-the-conda-environment-3)
            * [Run the snakemake](#run-the-snakemake-3)
            * [Prepare the input file for combined model usage (optional)](#prepare-the-input-file-for-combined-model-usage-optional-3)
      * [Guppy snakemake pipeline](#guppy-snakemake-pipeline)
            * [Modified basecallling](#modified-basecalling)
            * [Create and activate the Conda environment](#create-and-activate-the-conda-environment-4)
            * [Run the snakemake](#run-the-snakemake-4)
            * [Prepare the input file for combined model usage (optional)](#prepare-the-input-file-for-combined-model-usage-optional-4)
      * [Megalodon](#megalodon)
      * [Tombo](#tombo)
   * [Combined model usage](#combined-model-usage)
      * [Input file](#input-file)
      * [Command](#command)
      * [Per site predictions](#per-site-predictions)
      * [train your own combination model](#train-your-own-combination-model)



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

Once you have installed Conda, you can download the Snakemake pipelines and example datasets.
```
git clone https://github.com/comprna/METEORE.git
```
------------------------------------------
# Tutorial on an example dataset
------------------------------------------

We provide an example dataset `data/example` for you to try the pipelines with. The example contains 50 single-read fast5 files from the positive control dataset for E.coli generated by [Simpson et al. (2017)](https://www.nature.com/articles/nmeth.4184).

You can run the pipeline with your own dataset by replacing `example` folder in the `data` directory with your folder containing the fast5 files. You will use the **fast5 folder name** to specify your target output file in the Snakemake pipeline. Simply replace ***example*** in the output file with ***your fast5 folder name*** in the command line below.


## Nanopolish snakemake pipeline

### Create and activate the Conda environment

The first step is to create the environment from the nanopolish.yml file:
```
conda env create -f nanopolish.yml
```

Then activate the Conda environment:
```
conda activate nanopolish_cpg_snakemake
```

### Run the snakemake

Before executing the workflow below, make sure you have the basecalled fastq file in the `METEORE` directory. Nanopolish needs to link the read ids from the fastq file with their signal-level data in the fast5 files. An example fastq file `example.fastq` is provided.

A Snakefile named `Nanopolish` contains all rules for the Snakemake workflow. Run the snakemake to create the output files:
```
snakemake -s Nanopolish nanopolish_results/example_nanopolish-freq-perCG.tsv
```
This will produce four index files `example.fastq.index`, `example.fastq.index.fai`, `example.fastq.index.gzi` and `example.fastq.index.readdb`, and the `nanopolish_results` output directory containing all output files.
* `example_nanopolish-log.tsv` is the raw output after running `nanopolish call-methylation`.
* `example_nanopolish-log-perCG.tsv` contains per-read per-site data, which splits up the CpG group containing multiple nearby sites into its constituent CpG sites.
```
Chr             Pos         Strand	  Log.like.ratio	  Read_ID
NC_000913.3	    3499494	    +	        -0.62	            094dfe6b-23ed-4195-8876-805a399fade5
NC_000913.3	    3499526	    +	        -0.33	            094dfe6b-23ed-4195-8876-805a399fade5
NC_000913.3	    3499546	    +	        -0.12	            094dfe6b-23ed-4195-8876-805a399fade5
NC_000913.3	    3499563	    +	        8.26              094dfe6b-23ed-4195-8876-805a399fade5
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
snakemake -s Deepsignal deepsignal_results/example_deepsignal-perRead-score.tsv
```
The output is in .tsv format and contains four columns: `ID`,	`Pos`,	`Strand` and `Score`, e.g.:
```
ID                                        Pos    Strand    Score
2f43696e-70f0-42dd-b23e-d9e0ea954d4f    2687804    -       29.64
...
```

## DeepSignal snakemake pipeline

### Create and activate the Conda environment

The first step is to create the environment from the nanopolish.yml file:
```
conda env create -f deepsignal.yml
```

Then activate the Conda environment:
```
conda activate deepsignal_cpg_snakemake
```

Please download DeepSignal's trained model `model.CpG.R9.4_1D.human_hx1.bn17.sn360.v0.1.7+.tar.gz` [here](https://drive.google.com/drive/folders/1zkK8Q1gyfviWWnXUBMcIwEDw3SocJg7P).
To extract a `model.CpG.R9.4_1D.human_hx1.bn17.sn360.v0.1.7+.tar.gz`to the `METEORE/data` directory:
```
tar xvzf model.CpG.R9.4_1D.human_hx1.bn17.sn360.v0.1.7+.tar.gz -C <path_to_METEORE/data_diectory>
```

### Run the snakemake

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

You can also generate the per-read prediction output in a format that can be used in METEORE to generate a consensus prediction.
```
snakemake -s Deepsignal deepsignal_results/example_deepsignal-perRead-score.tsv
```
The output is in tsv format and contains four columns: `ID`,	`Pos`,	`Strand` and `Score`.

## Tombo snakemake pipeline

### Create and activate the Conda environment

The first step is to create the environment from the tombo.yml file:
```
conda env create -f tombo.yml
```

Then activate the Conda environment:
```
conda activate tombo_cpg_snakemake
```

### Run the snakemake

A Snakefile named `Tombo` contains all rules in the Snakemake workflow. Run the snakemake to create the output files:
```
snakemake -s Tombo example_tombo-freq-perCG.tsv
```
This will produce the following original modified base prediction results generated by `Tombo`. You can find the detailed information for all Tombo commands and outputs on the [tombo documentation page](https://nanoporetech.github.io/tombo/).
* `example.CpG.tombo.stats`: a binary Tombo statistics file
* `example.CpG.tombo.per_read_stats`: a per-read statistics file
* `example.dampened_fraction_modified_reads.plus.wig`: a wiggle output file which stores the estimated fraction of significantly modified reads mapped on +'ve strand
* `example.dampened_fraction_modified_reads.minus.wig`: a wiggle output file which stores the estimated fraction of significantly modified reads mapped on -'ve strand
* `example.coverage.plus.bedgraph`: coverage data for +'ve strand in bedGraph format
* `example.coverage.minus.bedgraph`: coverage data for -'ve strand in bedGraph format
There are rules in the Snakemake workflow for downstream processing of the output files generated by Tombo. The above command also 
generates a `tombo_results` output directory which contains the following output files:
* `example_tombo-freq-only.tsv`: contains methylation frequency results from the wiggle files
* `example_tombo-cov-only.tsv`: contains coverage data from the bedGraph files
* `example_tombo-freq-perCG.tsv`: the final per-site results combining the methylation frequency and coverage results.
```
Chr             Pos         Methyl_freq     Cov
NC_000913.3     3501925     0.7889          16
NC_000913.3     3501929     0.7889          16
NC_000913.3     3501956     0.51665         16
NC_000913.3     3501977     0.68335         16
```

### Prepare the input file for combined model usage (optional)

You can also generate the per-read prediction output in a format that can be used in METEORE to generate a consensus prediction.
```
Tombo tombo_results/example_tombo-perRead-score.tsv
```
The output is in tsv format and contains four columns: `ID`,	`Pos`,	`Strand` and `Score`.

**Note:** This command can only extract per-read data for a region of interest. You may need to go to ***script_in_snakemake/extract_tombo_per_read_results.py*** to specify your region of interest (chromosome, start position and end position) in the Python script and rerun the results.
Open the Python script ***extract_tombo_per_read_results.py*** with an editor of your choice and go to line 18 to 24 of the file:
```
# specify region of interest (plus strand) below:
reg_data_plus = tombo_helper.intervalData(
    chrm='NC_000913.3', start=412305, end=4584088, strand="+")

# specify region of interest (minus strand) below:
reg_data_minus = tombo_helper.intervalData(
    chrm='NC_000913.3', start=412305, end=4584088, strand="-")
```

## Guppy Snakemake pipeline

### Modified basecallling

You need to basecall with the standalone Guppy basecaller before running the Snakemake pipeline. The pipeline was only designed to process and analyse the Guppy's fast5 output using the open-source custom scripts available from https://github.com/kpalin/gcf52ref
Guppy basecaller is only available to Oxford Nanopore Technologies' customers via the community site. For download and installation instructions, please check their [website](https://community.nanoporetech.com/downloads)

Once you have installed Guppy, you can perform modified basecalling from the signal data using the `dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg` guppy config.
```
./<path_to_ont-guppy-cpu>/bin/guppy_basecaller --config dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg --fast5_out --input_path data/example/ --save_path guppy_results/ --cpu_threads_per_caller 10
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
* `example_guppy-log-perCG.tsv` contains per-read per-site data
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

You can also generate the per-read prediction output in a format that can be used in METEORE to generate a consensus prediction.
```
snakemake -s Guppy guppy_results/example_guppy-perRead-score.tsv
```
The output is in tsv format and contains four columns: `ID`,	`Pos`,	`Strand` and `Score`.

## Megalodon

We did not develop a snakemake pipeline for Megalodon as it can be run with a single command.

Please check out [Megalodon GitHub Page](https://github.com/nanoporetech/megalodon) for more details. Note that the new release of Megalodon requires Guppy basecaller to be installed to run Megalodon.
```
megalodon data/example/ --outputs mods --reference data/ecoli_k12_mg1655.fasta --mod-motif Z CG 0 --write-mods-text --processes 10 --guppy-server-path ./<path_to_ont-guppy-cpu>/bin/guppy_basecall_server --guppy-params "--num_callers 10" --guppy-timeout 240 --overwrite
```
This will produce the `megalodon_results` directory which contains logs, per-read modified base output `per_read_modified_base_calls.txt`, per-site modified base output `modified_bases.5mC.bed`.

We provide a bash script to generate the methylation frequency file and the input file for combined model usage, which are in the same format used in every methods. Run the bash script with ***your preferred filename*** (eg. example) for the output files:
```
./script/megalodon.sh example
```
You will get the following files:
* `example_megaldon-freq-perCG.tsv` contains the per-site data including the genomic position of the CpG site, methylation frequency and coverage. The methylation calls from both strands are merged into a single strand.
* `example_megalodon-perRead-score.tsv` is a per-read prediction file containing four columns: `ID`,	`Pos`,	`Strand` and `Score`. This file can be used later in METEORE to generate a consensus prediction.


## DeepMod

We did not develop a snakemake pipeline for DeepMod as it can be run with a single command.

Please check out [DeepMod GitHub Page](https://github.com/WGLab/DeepMod) for installation instructions and usage.
```
# Set the working directory where you install DeepMod
python bin/DeepMod.py detect --wrkBase <path_to_data_folder>/example/ --Ref <path_to_data_folder>/ecoli_k12_mg1655.fasta --outFolder <your_save_path>/deepmod_results --Base C --modfile train_mod/rnn_conmodC_P100wd21_f7ne1u0_4/mod_train_conmodC_P100wd21_f3ne1u0 --FileID example_deepmod --threads 10
```
Note that DeepMod does not produce per-read predictions, so we could not produce an output file in the same format as those described above (.tsv) to be later combined to build a consensus.

--------------------------------------
# Combined model usage
--------------------------------------

We have trained Random Forest models that combine the outputs from two of the methods above
to produce consensus predictions with improved accuracy. We also provide the scripts to train new models with two
or more methods.

First, please make sure you install the required libraries in the conda env

```
pip install -r requirements.txt
```


## Input file
To make the predictions from combination model (e.g. deepsignal and nanopolish) the input (.tsv) file from each method must be formatted as below:
```
ID                                        Pos    Strand    Score
2f43696e-70f0-42dd-b23e-d9e0ea954d4f    2687804    -       29.64
7ee0a989-c750-4dde-9114-354b97996dae    3104781    -       -4.47
dc9dcb55-703c-4251-a916-4214abd67991    1173719    +        5.34
2bea7f2a-f76c-491a-b5ee-b22c6f4a8539    1864274    -        5.33
```
where the score is the significance score given by each method. In our snakemake pipelines, this score is generated as described above. Nanopolish
and Guppy provide a log-likelihood ratio value per site and per read. Similarly, Tombo produces a significance value per site and per read. On the other hand, for DeepSignal we give the log-ratio of the probabilities for a site to be methylated over the probablity of being unmethylated for each read. Similarly, for Megalodon we used the difference of the log-probabilities to obtain a log-ratio.

## Command

Given the already pre-trained model (models available in `./saved_models/`), to run the model you need the following command:
```
python combination_model_prediction.py  -i [path of tsv file containing methods name and path] -m [model_to_use (default or optimized)] -o [output_file]
```
where the file for option `-i` contains the information about the methods used (1st col) and the path to the respective input file (2nd col). 
For example, if we call this file `model_content.tsv`:
```
python combination_model_prediction.py  -i model_content.tsv -m default -o output
```
the file `model_content.tsv` could have the format
```
deepsignal    ./test_case/deepsignal_test.tsv
nanopolish    ./test_case/nanopolish_test.tsv
```
And these .tsv files are of the format indicated above. For instance, the file `deepsignal_test.tsv` has the format
```
ID	Pos	Strand	Score
b9fdd6aa-ba93-4424-8f4b-c632e4d16d2e	1817032	-	2.45
b6d8fb1d-36e6-4f87-9106-ca74cb66b604	1816321	+	2.63
a293bb13-f8d3-4ffa-80c5-6cfcc8bab58c	4549464	+	0.32
930f9738-859a-49fd-bcca-671aaf47417c	1151179	-	-2.96
...
```
and similar for Nanopolish. This command will use the model named `rf_model_default_deepsignal_nanopolish.model` to score the sites and reads that are common 
between the two files `./test_case/deepsignal_test.tsv` and `./test_case/nanopolish_test.tsv`. 
Default refers to the parameters used to build the Random Forest, which in this case are the defult parameters from the 
[sklern library](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html) (n_estimator = 100 and max_dep = None). 

If we use the command 
```
python combination_model_prediction.py  -i model_content.tsv -m optimized -o output
```
The model used will be `rf_model_max_depth_3_n_estimator_10_deepsignal_nanopolish.model`. If you want a different combination, e.g. deepsignal+guppy, 
you can replace the "nanopolish" line in the `model_content.tsv` file with guppy and its input file path. You can also add other methods paths as well in the same file.

**Note**: The order of method names in the samples.tsv file should be same as the order used to generate the combined model. For example, we provide the models with name *'rf_model_default_**deepsignal_nanopolish**.model'* so the order in the *samples.tsv* file should be **deepsignal and then nanopolish** and not the other way round.

These commands will write the output in a directory called `combined_model_results`. New results from subsequent runs will be saved into 
the same output directory. The ouput after running combination_model_prediction.py script will contain predictions for the 
reads common to both deepsignal and nanopolish method. The
format is as below:
```
ID                                        Pos       Prediction  Prob_methylation
2f43696e-70f0-42dd-b23e-d9e0ea954d4f    2687804           0      0.02
dc9dcb55-703c-4251-a916-4214abd67991    1173719           1      0.90
2bea7f2a-f76c-491a-b5ee-b22c6f4a8539    1864274           0      0.45
...
```
The column prediction (0 refers to unmethylated and 1 refers to methylated) is based on a threshold of 0.5 score. That is, if the P(methylation) is <= 0.5, 
it is predicted as unmethylated (0), otherwise as methylated (1). Predictions for a different thresholds can be obtained by parsing the column `Prob_methylation`. 


## Per-site predictions

We also provide a Python script to convert the per-site predictions for each individual read (methylated / unmethylated) into per-site predictions at genome level (% methylation) by summarising the predictions on individual reads from the model:

```
python prediction_to_mod_frequency.py combined_per_site_per_read_results combined_per_site_results
```
Where the file `combined_per_site_per_read_results` is the output file from the previous step, and `combined_per_site_results` is the new output with the percentage methylation per genomic position.


## train your own combination model

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
ID	Pos	Strand	Score
b9fdd6aa-ba93-4424-8f4b-c632e4d16d2e	1817032	-	2.45
...
```
The command to obtain the consensus predictions could be of the form
```
python combination_model_prediction.py  -i new_model_content.tsv -m optimized -o output2
```
This command will use the model `rf_model_max_depth_3_n_estimator_10_deepsignal_nanopolish_guppy.model` to predict on the sites and reads common in the input files 
`deepsignal_test2.tsv`, `nanopolish_test2.tsv`, and `guppy_test2.tsv`. 
