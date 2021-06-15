# GPSeq pre-processing example

<!-- MarkdownTOC -->

- [Introduction](#introduction)
- [Requirements](#requirements)
- [Tutorial](#tutorial)

<!-- /MarkdownTOC -->

## Introduction

The pre-processing of GPSeq's sequencing output generates a BED file per restriction condition with the count of de-duplicated reads per restriction site. The pipeline achieves this with the following workflow:

* Quality control.
* Flag extraction and frequency calculation.
* Manual check of barcode flag frequency.
* Filtering by prefix.
* Mapping to reference genome.
* Filtering out bad alignments.
* Correcting mapping on negative strand.
* Grouping reads and assigning them to restriction sites.
* De-duplicating.

More details on each step can be found in the tutorial below. The current pipeline is *under development*. The old pipeline (`gpseq-seq-gg`) is available [here](https://github.com/ggirelli/gpseq-seq-gg).

## Requirements

The tutorial requires a few programs and R/Python packages.

Before proceeding, please consult the requirements page [here](pages/requirements.md).

## Tutorial

Follow the step-by-step tutorial [here](pages/tutorial.md).
