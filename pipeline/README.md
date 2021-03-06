# BigSataScript RNAseq Pipelines: Automated and scalable RNAseq analysis in the Cloud

The pipelines are designed to automate good-practice RNAseq analysis protocol including QC, read preprocessing, alignment and transcript quantification. Afterwards, DEG detection and GO/pathway enrichment analysis is performed using R & Bioconductor following the [tutorials](https://github.com/cribioinfo/CRI-Workshop-AMIA-2016-RNAseq/blob/master/Run_RNAseq.tutorial.rendered.ipynb).

![GitHub Logo](https://github.com/cribioinfo/CRI-Workshop-AMIA-2016-RNAseq/blob/master/notebook_ext/ipynb_data/assets/Figure29.png)
*DEG analysis will be performed using R & Bioconductor packages. It is not included in the automated pipeline.*

The pipeline is designed to be

* **Automate** *Full analysis pipeline from QC to read counts with submission of one master script*
* **Scalable** *Manages analysis of tens to hundreds of samples*
* **Flexible** *Fine control of analysis modules, parameters, and environmental settings*
* **Robust** *Extensive checkpoints and error detection features*
* **Reproducible** *Quick restart of analysis and sharing of results*
* **Real-time** *Easy access and monitor of the pipeline progress*
* **Transferable** *Runs on a desktop, work station, HPC and cloud with one single switch*
* **Adaptable** *Easily adopted to various research projects with minimal human intervention*

The pipeline is implemented in 

* [BigDataScript](http://pcingola.github.io/BigDataScript/) *Scripting language for data pipelines*
* [Perl](https://www.perl.org/) *Utilities*

# Quick Demo

### Quick Start

To test the pipeline, log into the AWS EC2 machine assigned to you. 
Go to directory `/home/ubuntu/CRI-Workshop-AMIA-2016-RNAseq/pipeline/test` and run 
 
```
#!bash
./Build_RNAseq.DLBC.sh
```

The test run takes about 4 minutes. An output directory `myProject` will be generated in the current directory. Result files will contain QC reports, preprocessed reads, alignment BAMs, alignment metrics and read count quantification of gene expression.

```
myProject/
├── configs
├── logs
└── results
    └── DLBC_samples
        ├── KO01
        │   ├── alignment
        │   ├── alignment_metrics
        │   ├── clean_reads
        │   ├── qc_reports
        │   └── read_counts
        ├── KO02
        ├── KO03
        ├── WT01
        ├── WT02
        └── WT03
```

### Description

To gain the first look into the pipelines, we provide sample data and scripts to build a small project with two conditions (KO vs WT), six samples.

| Sample | Group | Sequencing File | Sequencing Data |
|------|------|------|------|------|   
| KO01 | KO | KO01.fastq.gz | 74,126,025 reads |   
| KO02 | KO | KO02.fastq.gz | 64,695,948 reads |   
| KO03 | KO | KO03.fastq.gz | 52,972,573 reads |   
| WT01 | WT | WT01.fastq.gz | 55,005,729 reads |   
| WT01 | WT | WT02.fastq.gz | 61,079,377 reads |   
| WT01 | WT | WT03.fastq.gz | 66,517,156 reads | 

**To run the pipelines for your own projects, only two input files are required**. We have prepared examples of such files in directory `pipeline/test/`

* **DLBC.metadata.txt** *Sample metadata table*
* **DLBC.pipeline.yaml** *Pipeline configuration file*

Examples of output directories and files are provided in `pipeline/test/archive/KO01_WT01/`. 

This is a quick demo of the testing of a project. To run real analysis, please follow the instructions on [wiki](https://github.com/cribioinfo/CRI-Workshop-AMIA-2016-RNAseq/wiki).

### Build_RNAseq.DLBC.sh

This build script builds your project file structure, config files, the submission script, and launch the pipeline.

```
#!bash
now=$(date +"%m-%d-%Y_%H:%M:%S")
export PATH=/home/ubuntu/software/tree-1.7.0:$PATH
export PATH=/home/ubuntu/.bds:$PATH

## script
# BuildRNAseqExe=../Build_RNAseq.pl
BuildRNAseqExe=../Build_RNAseq.pl
SubmitRNAseqExe=Submit_RNAseq.DLBC.sh
RunRNAseqExe=../Run_RNAseq.bds

## project info
project=DLBC
metadata=DLBC.metadata.txt
config=DLBC.pipeline.yaml
projdir=$PWD/myProject

## options
threads=2
mapq=0
force="--force"
tree="--tree tree"

## bds
platform="--local"
scheduler=""
bdscfg=""
retry=0
# log="--log" ## Log all tasks (do not delete tmp files).
log=''

echo "START" `date`

## build pipeline scripts
echo "START" `date` " Running $BuildRNAseqExe "
perl $BuildRNAseqExe \
        --project $project \
        --metadata $metadata \
        --config $config \
        --projdir $projdir \
        --threads $threads \
        --mapq $mapq \
        --retry $retry \
        --pipeline $RunRNAseqExe \
        $force \
        $tree \
        $platform \
        $scheduler \
        $bdscfg \
        $log \
        >& Build_RNAseq.$project.$now.log

## submit pipeline master script
echo "START" `date` " Running $SubmitRNAseqExe "
sh $SubmitRNAseqExe

echo "END" `date`
```

### Submit_RNAseq.DLBC.sh

This submission script that was generated by `Build_RNAseq.DLBC.sh`, it calls the master BDS script `Run_RNAseq.bds` to launch the pipeline. This script can be submitted separately, or as part of the build script `Build_RNAseq.DLBC.sh` (default).

```
#!bash

## set up environment
now=$(date +"%m-%d-%Y_%H:%M:%S")

## submit BDS job (launch the pipeline)
bds -c /home/ubuntu/dev/rnaseq/CRI-Workshop-Nov2016-RNAseq/pipeline/test/myProject/DLBC.bds.cfg  -y 0    ../Run_RNAseq.bds -aligners star -callers featurecounts -projdir /home/ubuntu/dev/rnaseq/CRI-Workshop-Nov2016-RNAseq/pipeline/test/myProject -project DLBC -samples KO01 WT01 > Run_RNAseq.DLBC.$now.log.out 2> Run_RNAseq.DLBC.$now.log.err
```

# How to run pipelines on your own computers

The AWS machine has the working environment and tools pre-installed and is ready for analysis. If you want to run the pipelines on your own computational infrastructure (e.g. laptop, workstation, HPC), please follow the three steps as below.

General guidelines (Linux/Unix system)
* Install prerequisites
 * `Perl 5`, `Java 1.8`, `BigDataScript 0.99`, `GCC 4.8`, `R 3.3` or above
 * `bedtools`, `fastqc`, `featurecounts`, `picard`, `pigz`, `rseqc`, `sambamba`, `samtools`, `star`, `trimmomatic`, `UCSCtools` (`bedGraphToBigWig`)
* Install pipelines
 `git clone https://github.com/cribioinfo/CRI-Workshop-AMIA-2016-RNAseq.git`
* Replace software path in the example popeline YAML config file `pipeline/test/DLBC.pipeline.yaml` with the installation path on your computer
 * e.g. in the `fastqc` block, replace `/home/ubuntu/software/fastqc-0.11.5/fastqc` with the path to `fastqc` executable on your computer     
```
 fastqc:
      exe: /home/ubuntu/software/fastqc-0.11.5/fastqc
      mem: 4
      module: ~
      threads: 4
```
  *A `module` system will be in future development plan to implement, allowing easy management of software versions.*

# Documentation

Please see the [Wiki](https://github.com/riyuebao/CRI-Workshop-Nov2016-RNAseq/wiki) for full documentation, including installation, pipeline architecture and help menus.

# Communication

Questions and comments? Please post on [Issues](https://github.com/cribioinfo/CRI-Workshop-AMIA-2016-RNAseq/issues) or contact Riyue Bao at rbao AT bsd DOT uchicago DOT edu.

# More

* [RNAseq Workshop on AWS EC2](https://github.com/riyuebao/CRI-Workshop-Nov2016-RNAseq/blob/master/Run_RNAseq.tutorial.rendered.ipynb) *Full introduction and hands-on practice of RNAseq analysis & clinical applications*
