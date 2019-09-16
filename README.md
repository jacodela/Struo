Struo
=====

**Struo:** a pipeline for building custom databases for common metagenome profilers

> "Struo" --> from the Latin: “I build” or “I gather”


* Version: 0.1.4
* Authors:
  * Nick Youngblut <nyoungb2@gmail.com>
  * Jacobo de la Cuesta <jacobo.delacuesta@tuebingen.mpg.de>
* Maintainers:
  * Nick Youngblut <nyoungb2@gmail.com>
  * Jacobo de la Cuesta <jacobo.delacuesta@tuebingen.mpg.de>
* Previous name
  * Ley Lab MetaGenome Profiler DataBase generator (LLMGP-DB)


# Custom databases

Custom GTDB databases available at the [struo data ftp server](http://ftp.tue.mpg.de/ebio/projects/struo/)


# Description

## Setup

### conda env setup

* r-argparse
* r-curl
* r-data.table
* r-dplyr
* ncbi-genome-download
* snakemake

## Getting database genomes

### Downloading genomes

* If using [GTDB](https://gtdb.ecogenomic.org/) genomes, run `GTDB_metadata_filter.R` to select genomes
* If downloading genomes from genbank/refseq, you can use `genome_download.R`

## Input data (`samples.txt` file)

* The pipeline requires a tab-delimited table that includes the following columns (column names specified in the `config.yaml` file):
  * Sample ID
    * This will usually just be the species/strain names
  * Path to the genome assembly fasta file
    * NOTE: these must be gzip'ed
  * taxonomy ID
    * This should be the NCBI taxonomy ID at the species/strain level
      * Needed for Kraken
  * taxonomy
    * This should at least include `g__<genus>;s__<species>`
    * The taxonomy can include higher levels, as long as levels 6 & 7 are genus and species
    * Any taxonomy lacking genus and/or species levels will be labeled:
      * `g__unclassified`  (if no genus)
      * `s__unclassified`  (if no species)
    * This is needed for humann2
    
## Running the pipeline

### Edit the `config.yaml`

* Specify the input/output paths
* Modify parameters as needed

### Running locally

`snakemake --use-conda`

### Running on a cluster

If SGE, then you can use the `snakemake_sge.sh` script. You can create a similar bash script
for other cluster architectures. See the following resources for help:

* [Snakemake docs on cluster config](https://snakemake.readthedocs.io/en/stable/snakefiles/configuration.html)
* [Snakemake profiles](https://github.com/Snakemake-Profiles)

### General info on using `snakemake`

Snakemake allows for easy re-running of the pipeline on just genomes that have not yet been processed.
You can just add more genomes to the input table and re-run the pipeline (test first with `--dryrun`).
Snakemake should just process the new genomes and then re-create the combined dataset files (this must be done each time).
Make sure to not mess with the files in the `nuc_filtered` and `prot_filtered` directories! Otherwise,
snakemake may try to run all genomes again through the computationally expensive gene annotation process.


## Using the resulting databases

Set the database paths in humann2, kraken2, etc. to the new, custom database files.

* humann2
  * nucleotide
    * `all_genes_annot.fna.gz`
  * amino acid
    * `all_genes.dmnd`
* kraken2
  * `database*mers.kraken`
  

### Adding more samples (genomes) to an existing custom DB

Add new genomes to the input table and delete the following files (if they exist):

* humann2 database
  * all_genes_annot.dmnd
* kraken database
  * hash.k2d
  * taxo.k2d
* bracken database
  * database100mers.kraken
  * database150mers.kraken

### Adding existing gene sequences to humann2 databases

If you have gene sequences already formatted for creating a humann2 custom DB,
and you'd like to include them with the gene sequences generated from the
input genomes, then just provide the file paths to the nuc/prot fasta files
(`humann2_nuc_seqs` and `humann2_prot_seqs` in the `config.yaml` file).

All genes (from genomes & user-provided) will be clustered altogether with `vsearch`.
See the `config.yaml` for the default clustering parameters used.


