---
name: riboseq-sample-naming-convention-parsing
description: Use when organizing gzip-compressed FastQ inputs for RiboDoc-style Ribo-seq
  runs and sample filenames must be validated, parsed, or mapped to condition/replicate
  groupings before pipeline execution.
when_to_use_negative:
- Input files are single-end reads outside a RiboDoc-style project layout with no
  condition/replicate structure encoded in filenames
- Filenames are already structured by a metadata sheet rather than the [CONDITION]_[NAME].[REPLICATE].fastq.gz
  convention
- The analysis does not involve grouping samples by condition and replicate (e.g.
  single-sample exploratory QC)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: RiboDoc
  role: Pipeline consuming the parsed sample names to group samples for QC (PCA, Spearman
    correlation) and DESeq2 differential expression; naming is mandatory for its project
    folder structure
  repo: https://github.com/equipeGST/RiboDoc
- name: FastQC
  role: Receives the correctly named FastQ inputs in the 'fastq' subfolder for raw-read
    quality control before trimming
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:5683c7768aaceac7f0cb5f0841362c3c72e2153380842251650538838f5c2582
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/equipeGST/RiboDoc
  repo_url: https://github.com/equipeGST/RiboDoc
  tools:
  - Cutadapt
  - FastQC
  - RiboDoc
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# riboseq-sample-naming-convention-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parses Ribo-seq FastQ sample filenames into condition, sample name, and replicate fields according to RiboDoc's required naming convention ([CONDITION]_[NAME].[REPLICATE].fastq.gz), enabling the pipeline to group samples for differential expression and QC.

## When to use

Use when organizing gzip-compressed FastQ inputs for RiboDoc-style Ribo-seq runs and sample filenames must be validated, parsed, or mapped to condition/replicate groupings before pipeline execution.

## When NOT to use

- Input files are single-end reads outside a RiboDoc-style project layout with no condition/replicate structure encoded in filenames
- Filenames are already structured by a metadata sheet rather than the [CONDITION]_[NAME].[REPLICATE].fastq.gz convention
- The analysis does not involve grouping samples by condition and replicate (e.g. single-sample exploratory QC)

## Inputs

- gzip-compressed FastQ files (*.fastq.gz) named [CONDITION]_[NAME].[REPLICATE].fastq.gz
- Project folder structure with a lowercase 'fastq' subfolder

## Outputs

- Parsed sample metadata table mapping each FastQ file to condition, sample name, and replicate
- List of files failing the naming convention for user correction

## How to apply

RiboDoc requires FastQ files to be placed in a lowercase-named 'fastq' subfolder of the project volume and named exactly as [CONDITION]_[NAME].[REPLICATE].fastq.gz (e.g. Wild_Type.56.fastq.gz or Mutant.42.fastq.gz), where the segment before the final dot encodes condition_name and the last segment encodes the replicate. Parse by splitting on the final '.fastq.gz' suffix, then splitting the remaining stem on the first underscore (condition+name) and the last dot (replicate). Enforce exclusions: no dashes, parentheses, blank spaces, or special characters other than underscores; on Windows beware hidden extensions. Reject files whose names violate the pattern or whose parent folders contain uppercase letters, since these are 'majors for the good course of the analysis' and will break downstream sample grouping in the DESeq2 differential expression step.

## Related tools

- **RiboDoc** (Pipeline consuming the parsed sample names to group samples for QC (PCA, Spearman correlation) and DESeq2 differential expression; naming is mandatory for its project folder structure) — https://github.com/equipeGST/RiboDoc
- **FastQC** (Receives the correctly named FastQ inputs in the 'fastq' subfolder for raw-read quality control before trimming)

## Examples

```
python -c "import re,sys;[print(m.group(1),m.group(2),m.group(3)) for f in sys.argv[1:] if (m:=re.match(r'(\w+)_(.+)\.(\w+)\.fastq\.gz$',f))]" Wild_Type.56.fastq.gz Mutant.42.fastq.gz
```

## Evaluation signals

- Every parsed file matches the regex pattern ^[^_]+_.+\.[^.]+\.fastq\.gz$ (condition/name before the dot, replicate after)
- No filename contains dashes, parentheses, blank spaces, or special characters other than underscores
- Project subfolder names are all lowercase, with FastQ inputs inside the 'fastq' subfolder
- Each parsed replicate is unique within its condition and maps 1:1 to one *.fastq.gz file in the input folder
- Downstream RiboDoc run completes without sample-grouping errors (e.g. DESeq2 design resolves expected condition/replicate levels)

## Limitations

- On Windows, file extensions can be hidden, so files may appear to follow the convention while lacking the .fastq.gz suffix
- Underscores are permitted within condition or sample names, so parsing must split on the first underscore and the last dot rather than assuming single-underscore names
- The example GEO dataset (GSE173856) does not align enough material for the riboWaltz pipeline (TRiP must be selected), which affects later steps but not naming itself

## Evidence
