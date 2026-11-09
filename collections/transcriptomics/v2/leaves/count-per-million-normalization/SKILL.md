---
name: count-per-million-normalization
description: Use when per-sample RNA read counts from small RNA-seq annotation (COMPSRA
  Annotation Module tab-delimited count files) must be library-size normalized before
  differential expression testing between case and control samples.
when_to_use_negative:
- You only want to merge count files into a matrix without any case/control testing
  — use the merge-only path (-fm -fms, e.g., -pro COMPSRA_MERGE) instead of the differential-expression
  path
- The count data are already normalized to a common library size (CpM would re-scale
  an already-comparable matrix)
- There is no case/control design to compare — normalization here is tied to the -fd
  differential expression workflow of the COMPSRA Function module
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: COMPSRA
  role: Function module (-fun with -fdnorm cpm) normalizes each sample's small RNA
    read-count library to Count-per-Million before Mann-Whitney differential expression
    testing
  repo: https://github.com/cougarlj/COMPSRA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:28e7a9e3ca8716022d61d4cd0136b8ec7e8485049601e270cd1e17a41847615a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - COMPSRA
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# count-per-million-normalization

## Summary

COMPSRA's Function module default normalization scales each sample's small RNA-seq read-count library to Count-per-Million (CpM) so RNA molecules are comparable across samples before case-versus-control Mann-Whitney differential expression testing.

## When to use

Use when per-sample RNA read counts from small RNA-seq annotation (COMPSRA Annotation Module tab-delimited count files) must be library-size normalized before differential expression testing between case and control samples.

## When NOT to use

- You only want to merge count files into a matrix without any case/control testing — use the merge-only path (-fm -fms, e.g., -pro COMPSRA_MERGE) instead of the differential-expression path
- The count data are already normalized to a common library size (CpM would re-scale an already-comparable matrix)
- There is no case/control design to compare — normalization here is tied to the -fd differential expression workflow of the COMPSRA Function module

## Inputs

- Tab-delimited per-sample read-count files from the COMPSRA Annotation Module (one per FASTQ sample)
- A sample list file (e.g., ./example/sample.list) passed via -inf
- Case and control sample column designations (-fdcase/-fdctrl)
- Selected small RNA classes (-fdclass, e.g., 1,2,3,4,5,6)

## Outputs

- A merged count matrix with RNA molecules as rows and samples as columns
- CpM-normalized expression values (each sample library scaled to one million reads)
- Tab-delimited differential expression results file (e.g., COMPSRA_DEG.tsv) with Mann-Whitney U test p-values per RNA molecule

## How to apply

Load the per-sample read-count files output by the Annotation Module and merge them into a data matrix with RNA molecules as rows and samples as columns (COMPSRA Function module via -fm/-fms when only merging). Mark each sample column as case or control (e.g., -fdcase 1-6 -fdctrl 7-12) and restrict to the requested small RNA classes with -fdclass (e.g., 1,2,3,4,5,6 covering miRNA, piRNA, snRNA, snoRNA, tRNA, circRNA). Apply CpM normalization with -fdnorm cpm, which normalizes each sample library size into one million reads. The normalized values then feed the default Mann-Whitney rank sum test (Wilcoxon Rank Sum Test) selected with -fdtest mwu, and results are written to a tab-delimited output file named via -pro (e.g., COMPSRA_DEG).

## Related tools

- **COMPSRA** (Function module (-fun with -fdnorm cpm) normalizes each sample's small RNA read-count library to Count-per-Million before Mann-Whitney differential expression testing) — https://github.com/cougarlj/COMPSRA

## Examples

```
java -jar COMPSRA.jar -ref hg38 -fun -fd -fdclass 1,2,3,4,5,6 -fdcase 1-6 -fdctrl 7-12 -fdnorm cpm -fdtest mwu -fdann -pro COMPSRA_DEG -inf ./example/sample.list -out ./example_out/
```

## Evaluation signals

- After CpM normalization, each sample's normalized counts sum to one million reads (CpM definition: 'normalizes each sample library size into one million reads')
- The output table exists, is tab-delimited, and contains the annotated RNA molecules across the selected RNA classes (-fdclass 1,2,3,4,5,6)
- The output includes case-versus-control Mann-Whitney U test p-values for each RNA molecule, consistent with the paper's stated default statistical test
- The command explicitly sets -fdnorm cpm and -fdtest mwu, matching the documented defaults rather than relying on undocumented behavior
- When only merging is intended, the run uses -fm -fms (e.g., -pro COMPSRA_MERGE) and produces a matrix without differential expression columns

## Limitations

- CpM is the only normalization method documented as default for this module in the paper and README excerpt; no alternative normalization strategies are described
- CpM does not account for compositional bias beyond library size, which is inherent to the counts-per-million approach
- The paper's results section does not state the COMPSRA software version used (the README tracks releases up to v1.0.3, 2020-11-11)

## Evidence

- [methods] CpM default normalization method: "The default normalization method is Count-per-Million (CpM), which normalizes each sample library size into one million reads"
- [readme] Mann-Whitney default test after normalization: "perform a case versus control differential expression analysis for each RNA molecule using the Mann-Whitney rank sum test (Wilcoxon Rank Sum Test) as the default statistical test"
- [readme] Merge-only alternative path: "If you only want to merge the count files, you can use -fm -fms."
