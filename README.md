# SomaticShortV-nf

<p align="center">
:wrench: This pipeline is currently under development :wrench:
</p>

  - [Description](#description)
  - [Diagram](#diagram)
  - [User guide](#user-guide)
      - [Infrastructure usage and
        recommendations](#infrastructure-usage-and-recommendations)
  - [Benchmarking](#benchmarking)
  - [Workflow summaries](#workflow-summaries)
      - [Metadata](#metadata)
      - [Component tools](#component-tools)
  - [Additional notes](#additional-notes)
  - [Help/FAQ/Troubleshooting](#helpfaqtroubleshooting)
  - [Acknowledgements/citations/credits](#acknowledgementscitationscredits)

## Description 

SomaticShortV-nf is a Nextflow pipeline for identifying somatic short variant events in Illumina short read whole genome sequence data. 
We have followed the [GATK Best practices](https://gatk.broadinstitute.org/hc/en-us/articles/360035894731-Somatic-short-variant-discovery-SNVs-Indels-) . The input to this pipeline are the per 
sample "g.vcf" files which are generated using [nf-core/sarek](https://nf-co.re/sarek). 


There are three main steps in the process of calling Somatic Short Variants:

(1) **Creation of Somatic short variants Panel of Normals (PON)** : 
<br>This involves converting the Normal BAMs to PON. The PON's are -
  * Made from normal samples i.e. the samples derived from healthy tissue and 
  * Their main purpose is to capture recurrent technical artifacts in order to improve the results of the variant calling analysis.
<br>

(2) **Call the somatic short variants for a tumor-normal pair** :
<br>The two main steps involved in calling  Somatic Short Variants are - 
  * Generate a large set of candidate somatic variants,
  * Filter the candidate somatic variants into a more confident set of somatic variant calls.

- Calling candidate somatic short variants involved the use of the tool [Mutect2](https://gatk.broadinstitute.org/hc/en-us/articles/360035531132) which calls both SNVs and indels simultaneously by generating a local assembly of haplotypes in an active region, de-novo. When Mutect2 sees a region with somatic variations, it dis-regards the existing mapping information completely and re-assembles the reads in that region in order to generate candidate variant haplotypes.

* This was followed by calculating contamination using the tools [GetPileupSummaries](https://gatk.broadinstitute.org/hc/en-us/articles/9570416554907-GetPileupSummaries) and [CalculateContamination](https://gatk.broadinstitute.org/hc/en-us/articles/9570322332315-CalculateContamination)
* The next step was to learn the parameters of a model for orientation bias using the tool [LearnReadOrientationModel](https://gatk.broadinstitute.org/hc/en-us/articles/360051305331-LearnReadOrientationModel)
* The [FilterMutectCalls](https://gatk.broadinstitute.org/hc/en-us/articles/360036856831-FilterMutectCalls) tool then applies filters to the raw output of Mutect2. 


(3) **Annotating the variants**
<br> An external genomic variant annotations and functional effect prediction tool [SnpEff](http://pcingola.github.io/SnpEff/) was used for annotating the filtered variants (such as amino-acid changes etc). Please refer to the above link for SnpEff details.

## Diagram

<p align="center"> 
<img src="./images/Somatic_variant_calling.png" width="60%">
</p> 

## User guide 

To run this pipeline, you will need to prepare your input files, reference data, and clone this repository. Before proceeding, ensure Nextflow is installed on the system you're working on. To install Nextflow, see these [instructions](https://www.nextflow.io/docs/latest/getstarted.html#installation). 

### 1. Prepare inputs

To run this pipeline you will need the following inputs: 

* Paired-end BAM files
* Corresponding BAM index files  
* Input sample sheet 

This pipeline processes paired-end BAM files and is capable of processing multiple samples in parallel. BAM files are expected to be coordinate sorted and indexed (see [Fastq-to-BAM](https://github.com/Sydney-Informatics-Hub/Fastq-to-BAM) for an example of a best practice workflow that can generate these files).  

You will need to create a sample sheet with information about the samples you are processing, before running the pipeline. This file must be **tab-separated** and contain a header and one row per sample. Columns should correspond to sampleID, BAM file, BAI file: 

|sampleID|bam                   |bai                       |
|--------|----------------------|--------------------------|
|SAMPLE1 |/data/Bams/sample1.bam|/data/Bams/sample1.bam.bai|
|SAMPLE2 |/data/Bams/sample2.bam|/data/Bams/sample2.bam.bai|

When you run the pipeline, you will use the mandatory `--input` parameter to specify the location and name of the input file: 

```
--input /path/to/samples.tsv
```

### 2. Prepare the reference materials 

To run this pipeline you will need the following reference files:

* Indexed reference genome in FASTA format 

You will need to download and index a copy of the reference genome you would like to use. Reference FASTA files must be accompanied by a .fai index file. If you are working with a species that has a public reference genome, you can download FASTA files from the [Ensembl](https://asia.ensembl.org/info/data/ftp/index.html), [UCSC](https://genome.ucsc.edu/goldenPath/help/ftp.html), or [NCBI](https://www.ncbi.nlm.nih.gov/genome/doc/ftpfaq/) ftp sites. You can use our [IndexReferenceFasta-nf pipeline](https://github.com/Sydney-Informatics-Hub/IndexReferenceFasta-nf) to generate indexes. 

When you run the pipeline, you will use the mandatory `--ref` parameter to specify the location and name of the reference.fasta file: 

```
--ref /path/to/reference.fasta
```


### 3. Clone this repository 

Download the code contained in this repository with: 

```
git clone https://github.com/Sydney-Informatics-Hub/GermlineShortV-nf
```

This will create a directory with the following structure: 
```
SomaticShortV-nf/
├── LICENSE
├── README.md
├── config/
├── main.nf
├── modules/
└── nextflow.config
```
The important features are: 

* **main.nf** contains the main nextflow script that calls all the processes in the workflow.
* **nextflow.config** contains default parameters to use in the pipeline.
* **modules** contains individual process files for each step in the workflow. 
* **config** contains infrastructure-specific config files (this is currently under development)

### 4. Run the pipeline 

The most basic run command for this pipeline is: 

```
nextflow run main.nf --ref reference.fasta

```

By default, this will generate `work` directory, `results` output directory and a `runInfo` run metrics directory in the same location you ran the pipeline from. 

To specify additional optional tool-specific parameters, see what flags are supported by running:

```
nextflow run main.nf --help 
```

If for any reason your workflow fails, you are able to resume the workflow from the last successful process with `-resume`. 

## Infrastructure useage and recommendations 

Coming soon! 

## Benchmarking 

Coming soon!

## Workflow summaries
### Metadata 

|metadata field     | GermlineStructuralV-nf / v1.0     |
|-------------------|:--------------------------------- |
|Version            | 1.0                               |
|Maturity           | stable                            |
|Creators           | Nandan Deshpande, Georgie Samaha                    |
|Source             | NA                                |
|License            | GNU General Public License v3.0   |
|Workflow manager   | NextFlow                          |
|Container          | See Component tools               |
|Install method     | NA                                |
|GitHub             | https://github.com/Sydney-Informatics-Hub/SomaticShortV-nf|
|bio.tools 	        | NA                                |
|BioContainers      | NA                                | 
|bioconda           | NA                                |

### Component tools 

To run this pipeline you must have Nextflow and Singularity installed on your machine. All other tools are run using containers. 

|Tool         | Version  |
|-------------|:---------|
|Nextflow     |>=20.07.1 |
|Singularity  |          |
|SnpEff       |     |
|VEP          |108       |
|R            |          |


## Additional notes 
### Resources 

* [Nextflow documentation](https://www.nextflow.io/docs/latest/index.html) 

### Help/FAQ/Troubleshooting

* It is essential that the reference genome you're using contains the same chromosomes, contigs, and scaffolds as the BAM files. To confirm what contigs are included in your indexed BAM file, you can use Samtools idxstats: 
```
samtools idxstats input.bam | cut -f 1
```

## Acknowledgements/citations/credits
### Authors 
- Nandan Deshpande (Sydney Informatics Hub, University of Sydney)   
- Georgie Samaha (Sydney Informatics Hub, University of Sydney)   

### Acknowledgements 

- This pipeline was built using the [Nextflow DSL2 template](https://github.com/Sydney-Informatics-Hub/Nextflow_DSL2_template).  
- Documentation was created following the [Australian BioCommons documentation guidelines](https://github.com/AustralianBioCommons/doc_guidelines).  

### Cite us to support us! 
Acknowledgements (and co-authorship, where appropriate) are an important way for us to demonstrate the value we bring to your research. Your research outcomes are vital for ongoing funding of the Sydney Informatics Hub and national compute facilities. We suggest including the following acknowledgement in any publications that follow from this work:  

The authors acknowledge the technical assistance provided by the Sydney Informatics Hub, a Core Research Facility of the University of Sydney and the Australian BioCommons which is enabled by NCRIS via Bioplatforms Australia. 
