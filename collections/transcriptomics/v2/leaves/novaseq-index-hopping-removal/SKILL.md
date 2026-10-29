---
name: novaseq-index-hopping-removal
description: Use when combined GEX and ADT data from a NovaSeq run show sample misassignment
  due to index hopping and a clean per-sample count matrix is needed before running
  gExcite.
when_to_use_negative:
- When sequencing was not performed as combined GEX and ADT NovaSeq runs — the article
  states the cleanup scripts are provided 'In case of combined GEX and ADT NovaSeq
  sequencing data'
- When the count data are already demultiplexed and free of sample-misassignment artifacts,
  since this is explicitly an optional preprocessing step ('optional preprocessing
  was performed if necessary')
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: gExcite
  role: Workflow providing the index_hopping_removal scripts as an optional read-mapping
    cleanup stage
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: Snakemake
  role: Workflow manager embedding the index-hopping removal scripts as a pre-processing
    rule ahead of the main gExcite run
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: cellranger
  role: Produces the GEX/ADT count matrices that feed the index-hopping removal step
- name: DropletUtils
  role: Package used by the gExcite index-hopping removal scripts to identify and
    remove misassigned reads
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad329
    title: 'gExcite: a start-to-end framework for single-cell gene expression, hashing,
      and antibody analysis'
  source_content_hash: sha256:b995d03191c9c1abdcf90f9ce92674219a9090896fa3a4ee878bb02d159deda4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ETH-NEXUS/gExcite_pipeline
  tools:
  - DropletUtils
  - Snakemake
  - cellranger
  - gExcite
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# novaseq-index-hopping-removal

## Summary

Optional gExcite pre-processing step that identifies and removes index-hopped reads from combined GEX and ADT NovaSeq sequencing data using DropletUtils, yielding a cleaned count matrix for downstream hashing deconvolution and gene/antibody analysis.

## When to use

Use when combined GEX and ADT data from a NovaSeq run show sample misassignment due to index hopping and a clean per-sample count matrix is needed before running gExcite.

## When NOT to use

- When sequencing was not performed as combined GEX and ADT NovaSeq runs — the article states the cleanup scripts are provided 'In case of combined GEX and ADT NovaSeq sequencing data'
- When the count data are already demultiplexed and free of sample-misassignment artifacts, since this is explicitly an optional preprocessing step ('optional preprocessing was performed if necessary')

## Inputs

- Unfiltered GEX count data from a combined NovaSeq sequencing run
- Unfiltered ADT (CITE-seq) count data from a combined NovaSeq sequencing run
- FASTQ files arranged per sample as SAMPLENAME_S[Number]_L00[Lane]_[ReadType]_001.fastq.gz

## Outputs

- index_hopping_cleaned_count_matrix.tsv (clean count matrix with index-hopped reads removed and remaining reads reassigned to correct samples)
- Pre-processed data accepted by the downstream gExcite Snakemake pipeline

## How to apply

Load the unfiltered GEX and ADT count data for the combined NovaSeq sequencing run, verifying that FASTQ/count files follow the required per-sample folder structure (SAMPLENAME_S[Number]_L00[Lane]_[ReadType]_001.fastq.gz). Run the gExcite index-hopping removal scripts (workflow/scripts/index_hopping_removal) on the GEX and ADT data to identify reads misassigned between hashed samples, using DropletUtils. Remove the index-hopped reads from the count matrices and reassign remaining reads to their correct samples. Write the cleaned count matrix to index_hopping_cleaned_count_matrix.tsv and wire it into the Snakemake workflow as a pre-processing stage ahead of the main gExcite run. The paper applies this only for combined GEX and ADT NovaSeq sequencing data, so treat it as an optional pre-processing step configured in config.yaml.

## Related tools

- **gExcite** (Workflow providing the index_hopping_removal scripts as an optional read-mapping cleanup stage) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **Snakemake** (Workflow manager embedding the index-hopping removal scripts as a pre-processing rule ahead of the main gExcite run) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **cellranger** (Produces the GEX/ADT count matrices that feed the index-hopping removal step)
- **DropletUtils** (Package used by the gExcite index-hopping removal scripts to identify and remove misassigned reads)

## Examples

```
snakemake --use-conda -n --configfile config/config.yaml  # dry run to validate index_hopping_cleaned_count_matrix.tsv as a pre-processing target
```

## Evaluation signals

- The cleaned count matrix file index_hopping_cleaned_count_matrix.tsv exists and is readable
- A dry run of the Snakemake workflow (snakemake -n) completes without errors with the cleaned matrix integrated as a pre-processing stage
- The downstream gExcite pipeline run accepts the pre-processed data without errors
- FASTQ/count inputs verified to follow the per-sample folder structure before cleanup is attempted
- No quantitative threshold is reported in the paper for this step, so validate structurally (file existence and pipeline acceptance) rather than numerically

## Limitations

- No quantitative threshold is reported in the paper for this step, so success is judged by structural validation rather than numeric cutoffs
- Cleanup applies only in the case of combined GEX and ADT NovaSeq sequencing data
- FASTQ inputs must be specified relative to the working directory (mandatory, not merely recommended), otherwise the pipeline will not pick them up

## Evidence

- [methods] Index hopping removal is an optional read-mapping cleanup step: "In case of combined GEX and ADT NovaSeq sequencing data, scripts are provided to clean up the data before a run"
- [results] DropletUtils-based cleanup yields a clean count matrix for downstream processing: [evidence span withheld — anchor status: not_found]
- [readme] Pipeline treats this as optional preprocessing: "optional preprocessing was performed if necessary"
