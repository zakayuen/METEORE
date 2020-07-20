# METEORE
MEthylation deTEction with nanopORE sequencing                                         :stars:

----------------------------
# Table of Contents
----------------------------

   * [Pipeline](#pipeline)
   * [Installation](#installation)
   * [Tutorial on an example dataset](#tutorial-on-an-example-dataset)
      * [Nanopolish snakemake pipeline](#nanopolish-snakemake-pipeline)
      * [DeepSignal snakemake pipeline](#deespsignal-snakemake-pipeline)
      * [Tombo snakemake pipeline](#nanopolish-snakemake-pipeline)
   * [Combined model usage](#combined-model-usage)
      * [Input file](#input-file)
      * [Command](#command)
      * [Output](#output)


----------------------------
# Pipeline
----------------------------

![Analysis pipeline](https://github.com/comprna/METEORE/figure/pipeline.png)
**Fig 1. Pipeline for CpG methylation detection form nanopore sequencing data **


----------------------------
# Installation
----------------------------
We recommand to install software dependencies via `Conda` on Linux. You can find Miniconda installation instructions for Linux [here](https://docs.conda.io/projects/conda/en/latest/user-guide/install/linux.html).
Make sure you install the [Miniconda Python3 distribution](https://docs.conda.io/en/latest/miniconda.html#linux-installers).
```
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
``` 
Accept the license terms during installation.

Once you have installed Conda, you can download the Snakemake piplines and example datasets.
```
git clone https://github.com/comprna/METEORE.git
```
------------------------------------------
# Tutorial on an example dataset
------------------------------------------

We provide an example dataset `data/example` for you to try with. The example contains 50 single-read fast5 files which are from the positive control dataset for E.coli generated by Simpson et al. (2017).

You can run the pipeline with you own dataset by replacing `example` folder in the `data` directory with your folder containing the fast5 files. You will use the folder name to specify the input in the Snakemake pipeline. Simply replace *example* with your folder name in the command line below.  

## Nanopolish snakemake pipeline
The first step is to create the environment from the nanopolish.yml file:
```
conda env create -f nanopolish.yml
```

Then activate the Conda environment:
```
conda activate nanopolish_cpg_snakemake
```

A Snakefile named `Nanopolish` contains all rules in the Snakemake workflow. Run the snakemake:
```
snakemake -s Nanopolish nanopolish_results/example_nanopolish-freq-perCG.tsv
```
This will produce the `nanopolish_results` output directory containing all output files. 
* `example_nanopolish-log.tsv` is the raw output after running `nanopolish call-methylation`. 
* `example_nanopolish-log-perCG.tsv` contains per-read per-site data, which splits up the CpG group containing multiple nearby sites into its constituent CpG sites. 
* `example_nanopolish-freq-perCG.tsv` stores the per-site data including the position of the CpG site on the reference genome, methylation frequency and coverage.

For your convenience, there is a rule in Snakemake to create the input file for making the predictions from the model:
```
snakemake -s Nanopolish nanopolish_results/example_nanopolish-score.tsv
```

## DeepSignal snakemake pipeline
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
tar xvzf model.CpG.R9.4_1D.human_hx1.bn17.sn360.v0.1.7+.tar.gz -C /path/to/[METEORE/data]directory
```

A Snakefile named `Deepsignal` contains all rules in the Snakemake workflow. Run the snakemake:
```
snakemake -s Deepsignal deepsignal_results/example_deepsignal-freq-perCG-comb.tsv
```
This will produce the `deepsignal_results` output directory containing all output files. 
* `example_deepsignal-prob.tsv` contains the per-read results including the position of the CpG site, read ID, strand, methylated probability, unmethylated probability etc. 
* `example_deepsignal-freq-perCG.tsv` contains the per-site results using a Python script provided by `DeepSignal`. 
* `example_deepsignal-freq-perCG-comb.tsv` also contains the per-site results but we combine the methylation calls from both strands into a single strand. 

For your convenience, there is a rule in Snakemake to create the input file for making the predictions from the model:  
```
snakemake -s Deepsignal deepsignal_results/example_deepsignal-score.tsv
``` 

## Tombo snakemake pipeline

The first step is to create the environment from the nanopolish.yml file:
```
conda env create -f tombo.yml
```

Then activate the Conda environment:
```
conda activate tombo_cpg_snakemake
```

A Snakefile named `Tombo` contains all rules in the Snakemake workflow. Run the snakemake:
```
snakemake -s Tombo example_tombo-mods-freqNcov-perCG.tsv
```
`example_tombo-mods-freqNcov-perCG.tsv` contains the per-site results including the position of the CpG site, methylation frequency and coverage.

If you also want the per-read results from Tombo, you can run the following:
```
snakemake -s Tombo example_tombo_mods-scores-perRead.tsv
```
Note that this can only extract per-read data for a region of interest. You may need to go to `script/extract_tomo_per_read_results.py` to specify your region of interest (chromosome, start position and end position) in the script and rerun the results.

## Guppy

Guppy basecaller is available to Oxford Nanopore Technologies' customers via the community site. For download and installation instructions, please check out [here](https://community.nanoporetech.com/downloads)

Once you have installed Guppy, you can perform modified basecalling from the signal data using the `dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg` guppy config.
```
./<path_to_ont-guppy-cpu/bin/guppy_basecaller --config dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg --fast5_out --input_path data/example/ --save_path guppy_results/example_guppyHacModbase/ --cpu_threads_per_caller 10
```
This will create a folder named `workspace` which contains basecalled fast5 files with modified base information.

To process and analyse the Guppy's fast5 output, an open-source custom pipeline is used: https://github.com/kpalin/gcf52ref


## Megalodon

Please check out [Megalodon GitHub Page](https://github.com/nanoporetech/megalodon) for more details. Note that the new release of Megalodon requires Guppy basecaller to be installed to run Megalodon.
```
megalodon data/example/ --outputs mods --reference data/ecoli_k12_mg1655.fasta --mod-motif Z CG 0 --write-mods-text --processes 10 --guppy-server-path ./<path/to/ont-guppy-cpu>/bin/guppy_basecall_server --guppy-params "--num_callers 10" --guppy-timeout 240 --overwrite
```

--------------------------------------
# Combined model usage
--------------------------------------

## Input file
To make the predictions from combination model (deepsignal and nanopolish) format the input file (TSV) as below:
```
ID                                        Pos    Strand    Score
2f43696e-70f0-42dd-b23e-d9e0ea954d4f    2687804    -       29.64
7ee0a989-c750-4dde-9114-354b97996dae    3104781    -       -4.47
dc9dcb55-703c-4251-a916-4214abd67991    1173719    +        5.34
2bea7f2a-f76c-491a-b5ee-b22c6f4a8539    1864274    -        5.33

```

## Command

```
python combination_model_prediction.py  -a [fullpath_of_deepsignal_input_file] -b [fullpath_of_nanopolish_input_file] -m [model to use] -o [output_path]

```
Example for the testcase file provided in the package:

cd inside the directory downloaded package directory METEORE then run
```
python combination_model_prediction.py  -a test_case/deepsignal_test.tsv -b test_case/nanopolish_test.tsv -m deepsignal_nanopolish -o [output_path]

```

## Output

The ouput after running combination_model_prediction.py script will contain predictions for the reads common to both deepsignal and nanopolish method. The
format is as below:
```
ID                                        Pos       Prediction  Prob_methylation
2f43696e-70f0-42dd-b23e-d9e0ea954d4f    2687804           0      0.02
dc9dcb55-703c-4251-a916-4214abd67991    1173719           1      0.90
2bea7f2a-f76c-491a-b5ee-b22c6f4a8539    1864274           0      0.45

```

