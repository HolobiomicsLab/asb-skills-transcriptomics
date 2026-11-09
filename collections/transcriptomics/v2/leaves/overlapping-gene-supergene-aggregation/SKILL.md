---
name: overlapping-gene-supergene-aggregation
description: Use when preprocessing a real scRNA-seq BAM file for count-matrix training
  and reads overlapping annotated genes must be assigned to mutually exclusive genomic
  features.
when_to_use_negative:
- Input is an scATAC-seq BAM for peak-based segmentation — use Utility.scATAC_CreateFeatureSets
  and Utility.CallPeak (MACS3) instead.
- The goal is isoform-level ground truths (isoform abundance or alternative splicing
  events), which scReadSim explicitly does not generate.
- Input already consists of non-overlapping feature regions and no supergene merging
  is needed.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: scReadSim
  role: Provides Utility.scRNA_CreateFeatureSets to build foreground/background BED
    feature sets and Utility.scRNA_bam2countmat to build the UMI count matrices
  repo: https://github.com/JSB-UCLA/scReadSim
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:dad337f8b5d900eac2e88e7fa43bcad9c2279d4c38026135eefafa06da04b3a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - scReadSim
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# overlapping-gene-supergene-aggregation

## Summary

scReadSim's scRNA-seq genome-segmentation step partitions the reference genome into gene (foreground) and inter-gene (background) BED regions, aggregating overlapping genes into non-overlapping supergenes so counts can be unambiguously assigned.

## When to use

Use when preprocessing a real scRNA-seq BAM file for count-matrix training and reads overlapping annotated genes must be assigned to mutually exclusive genomic features.

## When NOT to use

- Input is an scATAC-seq BAM for peak-based segmentation — use Utility.scATAC_CreateFeatureSets and Utility.CallPeak (MACS3) instead.
- The goal is isoform-level ground truths (isoform abundance or alternative splicing events), which scReadSim explicitly does not generate.
- Input already consists of non-overlapping feature regions and no supergene merging is needed.

## Inputs

- BAM file of real scRNA-seq reads (e.g., 10x single-cell RNA-seq fresh embryonic E18 mouse brain, mm10 reference genome)
- Gene annotation GTF (gencode.vM10.annotation.gtf)
- Chromosome sizes file (mm10.chrom.sizes, embedded in the scReadSim package)

## Outputs

- Gene-region foreground BED file with overlapping genes aggregated into non-overlapping supergenes
- Complementary inter-gene background BED file
- Gene-by-cell UMI count matrix ('<filename>.countmatrix.txt' / '<filename>.UMI.countmatrix.txt')
- Inter-gene-by-cell UMI count matrix with matching cell columns

## How to apply

Run Utility.scRNA_CreateFeatureSets on the input BAM file using the gencode.vM10.annotation.gtf annotation together with the mm10.chrom.sizes chromosome-size file embedded in the package. This produces the gene-region foreground BED file and its complementary inter-gene background BED file; where gene annotations overlap, they are merged into non-overlapping supergenes so each read position maps to exactly one feature. Next run Utility.scRNA_bam2countmat on the foreground BED with UMI_modeling=True and UMI_count_mat_filename set, counting per cell barcode the UMIs whose reads overlap each gene/supergene; repeat on the background inter-gene BED with the same cells in the same column order. The gene-by-cell UMI count matrix becomes the ground truth used downstream by scDesign2 for synthetic count simulation.

## Related tools

- **scReadSim** (Provides Utility.scRNA_CreateFeatureSets to build foreground/background BED feature sets and Utility.scRNA_bam2countmat to build the UMI count matrices) — https://github.com/JSB-UCLA/scReadSim

## Examples

```
from scReadSim.Utility import scRNA_CreateFeatureSets, scRNA_bam2countmat; scRNA_CreateFeatureSets(bamfilename='10x.bam', gtf_filename='gencode.vM10.annotation.gtf', chrom_sizes_filename='mm10.chrom.sizes', output_dir='out/')
```

## Evaluation signals

- Both output files '<filename>.countmatrix.txt' and '<filename>.UMI.countmatrix.txt' are present in the output directory.
- The gene-by-cell and inter-gene-by-cell matrices contain the same cells in the same column order.
- Foreground BED regions are non-overlapping (overlapping genes were merged into supergenes).
- The inter-gene background BED is exactly complementary to the foreground gene BED (Utility.ComplementFeature output).
- Count matrix rows correspond to genes/supergenes from the GTF-derived feature set, not raw overlapping GTF gene IDs.

## Limitations

- The default implementation may generate intronic synthetic reads when extracting sequences from the reference genome, which can affect benchmarks such as cellranger default mode.
- scReadSim generates gene-level ground truths (UMI counts) only, not isoform-level ground truths.
- Multi-sample or multi-condition simulation requires users to supply one BAM file per sample or condition.

## Evidence

- [methods] Workflow step 1: [evidence span withheld — anchor status: not_found]
- [methods] Feature construction function: "Use function `scRNA_CreateFeatureSets` to generate features."
- [methods] Count matrix function: "scReasSim constructs the count matrices for both foreground feautures and background features through function `Utility.scRNA_bam2countmat`"
- [readme] Readme quick install: [evidence span withheld — anchor status: external_doc]
- [methods] Reference genome context: "This BAM file uses mm10 as reference genome, the required chromosome size file is also embedded within the package."
