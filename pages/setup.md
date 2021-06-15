# Setup

Before proceeding, please consult the requirements page [here](../pages/requirements.md).

<!-- MarkdownTOC -->

- [Input](#input)
- [Reference genome](#reference-genome)
- [Cutsite list](#cutsite-list)
- [Parameters](#parameters)

<!-- /MarkdownTOC -->

## Input

Create a folder where to perform the analysis:

```bash
mkdir $HOME/gpseq-tutorial
cd $HOME/gpseq-tutorial
```

Check if `sratools` are available by running:

```bash
fasterq-dump -h
```

If the output is `command not found: fastq-dump`, install them (only for the current tutorial) by running:

```
mkdir -p $HOME/gpseq-tutorial/tools/sra-tools/
cd $HOME/gpseq-tutorial/tools/sra-tools/
curl https://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/2.9.6/sratoolkit.2.9.6-ubuntu64.tar.gz -o sratoolkit.2.9.6-ubuntu64.tar.gz
tar -xvzf sratoolkit.2.9.6-ubuntu64.tar.gz
export PATH=$HOME/tools/sra-tools/sratoolkit.2.9.6-ubuntu64/bin:$PATH
fasterq-dump -h # check installation
```

Then, download the input data with the following (*NOTE. As the file is ~2 GB it might take a few minutes to complete the download.*):

```bash
cd $HOME/gpseq-tutorial
mkdir fastq
fasterq-dump SRR9974287 -O fastq -p
mv fastq/SRR9974287.fastq fastq/TUTORIAL01_S1_LALL_R1_001.fastq
gzip fastq/TUTORIAL01_S1_LALL_R1_001.fastq
```

## Reference genome

Download the `GRCh37.p13` reference genome with the following:

```bash
mkdir reference
curl http://ftp.ensembl.org/pub/release-104/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz --output reference/Homo_sapiens.GRCh38.dna.primary_assembly.noChr.fa.gz
zcat reference/Homo_sapiens.GRCh38.dna.primary_assembly.noChr.fa.gz | sed 's/>/>chr/' | gzip > reference/Homo_sapiens.GRCh38.dna.primary_assembly.fastq.gz
rm reference/Homo_sapiens.GRCh38.dna.primary_assembly.noChr.fa.gz
```

Then, build the bowtie2 index with:

```bash
```

## Cutsite list

```bash
```

## Parameters

Execute the following to set the parameter values.

```bash
# Parameters
input="$HOME/fastq/TUTORIAL01_S1_LALL_R1_001.fastq.gz"
libid="TUTORIAL01"
bowtie2_ref="$HOME/reference/"
cutsite_path="$HOME/reference/"
threads=10
```

Five parameters are required to run the pipeline:

* `input` is the path to the input fastq file (generally gzipped and merge by all lanes).
* `libid` is the library ID (for Illumina filenames, the first bit of the fastq name).
* `bowtie2_ref` is the path to the bowtie2 index.
* `cutsite_ref` is the path to a (gzipped) BED file with the location of known restriction sites.
* `threads` is the number of threads used for parallelization.
