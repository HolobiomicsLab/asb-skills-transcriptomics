---
name: clustering-agreement-ari-nmi-evaluation
description: Use when you have a per-cell TE count matrix (locus-level or subfamily-level)
  from MATES and need to validate quantification quality via clustering agreement
  (ARI/NMI) with gene-based clustering or via ARI improvement tests.
when_to_use_negative:
- The MATES multi-mapping rate estimation is expected to be inaccurate for TEs longer
  than 2001 bp, so ARI gains involving such loci should not be over-interpreted
- The input is already a published benchmark table — reproduce rather than re-derive
  the reported ARI/NMI values
- The goal is marker TE identification rather than quantification-quality validation;
  use marker association testing instead
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MATES
  role: Produces the locus-level and subfamily-level TE count matrices that are clustered
    and compared via ARI/NMI
  repo: https://github.com/mcgilldinglab/MATES
- name: TE_quantifier
  role: Estimates locus-specific multi-mapping rate alpha_i and computes per-cell
    TE counts used as clustering features
  repo: https://github.com/mcgilldinglab/MATES
- name: data_processor
  role: Provides utilities for constructing matrices and comparing TE-based clustering
    with gene-peak-based clustering
  repo: https://github.com/mcgilldinglab/MATES
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:e841504a638041a4a8339fe346d5ed42485df42c929377276369d4ba450b72b9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mcgilldinglab/MATES
  repo_url: https://github.com/mcgilldinglab/MATES
  tools:
  - MATES
  - MATES_model
  - TE_quantifier
  - bam_processor
  - data_processor
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# clustering-agreement-ari-nmi-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate whether locus-level TE quantification from MATES produces biologically meaningful single-cell groupings by clustering cells on TE count matrices and measuring agreement (median ARI and NMI) against gene-expression- or gene-peak-based clustering, and by testing locus-level vs subfamily-level ARI improvements.

## When to use

Use when you have a per-cell TE count matrix (locus-level or subfamily-level) from MATES and need to validate quantification quality via clustering agreement (ARI/NMI) with gene-based clustering or via ARI improvement tests.

## When NOT to use

- The MATES multi-mapping rate estimation is expected to be inaccurate for TEs longer than 2001 bp, so ARI gains involving such loci should not be over-interpreted
- The input is already a published benchmark table — reproduce rather than re-derive the reported ARI/NMI values
- The goal is marker TE identification rather than quantification-quality validation; use marker association testing instead

## Inputs

- Per-cell locus-level TE count matrix from MATES TE_quantifier (counts = URi + ceil(alpha_i x MRi))
- Per-cell subfamily-level TE count matrix for comparison
- Gene expression matrix (scRNA-seq) or gene-peak matrix (scATAC-seq) with matching cell barcodes
- Cell cluster labels from gene-based clustering as the reference partition

## Outputs

- Median ARI and NMI between TE-based and gene-based cell clusterings
- Locus-level vs subfamily-level ARI comparison with significance P-value (e.g., P = 5.48e-7; P = 2.55e-34)
- Cell cluster assignments on TE-only and gene-based matrices

## How to apply

After MATES computes per-cell locus-level counts as URi + ceil(alpha_i x MRi), cluster cells on the TE-only count matrix and independently on the gene expression matrix (or gene-peak matrix for scATAC-seq), then compute agreement as median Adjusted Rand Index (ARI) and Normalized Mutual Information (NMI) across resamples. Benchmark against reported values: for Smart-Seq2 human glioblastoma scRNA-seq, TE-only vs gene-based clustering yields median ARI 0.105 and NMI 0.161; for 10x adult mouse brain scATAC-seq, locus-level TE clustering vs gene-peak clustering yields median ARI 0.309 and NMI 0.438. To demonstrate that locus-specific resolution (not just TE signal) drives the gain, recompute clustering on subfamily-level counts and test the ARI improvement (locus vs subfamily: P = 5.48e-7 in glioblastoma, Fig. 3j; P = 2.55e-34 in mouse brain scATAC, Fig. 4i). Accept the analysis when the reproduced medians and significance levels match the published benchmarks.

## Related tools

- **MATES** (Produces the locus-level and subfamily-level TE count matrices that are clustered and compared via ARI/NMI) — https://github.com/mcgilldinglab/MATES
- **TE_quantifier** (Estimates locus-specific multi-mapping rate alpha_i and computes per-cell TE counts used as clustering features) — https://github.com/mcgilldinglab/MATES
- **data_processor** (Provides utilities for constructing matrices and comparing TE-based clustering with gene-peak-based clustering) — https://github.com/mcgilldinglab/MATES

## Examples

```
from MATES import MATES_pipeline; mates = MATES_pipeline(TE_mode, data_mode, sample_list_file, bam_path_file); mates.run()
```

## Evaluation signals

- Median ARI and NMI for TE-only vs gene-based clustering on Smart-Seq2 glioblastoma reproduce ~0.105 and ~0.161
- Median ARI and NMI for locus-level TE clustering vs gene-peak clustering on 10x mouse brain scATAC reproduce ~0.309 and ~0.438
- Locus-level clustering ARI is significantly higher than subfamily-level ARI (P = 5.48e-7 glioblastoma; P = 2.55e-34 mouse brain scATAC)
- Cell barcodes in TE-based and gene-based clusterings are the same set, ensuring a valid pairwise ARI/NMI comparison
- Marker TE associations (e.g., HUERS-P1-int/HERVK-int with neoplastic, LTR2B with immune populations) are recovered from the same clusters

## Limitations

- ARI/NMI agreement between TE-only and gene-based clustering is modest even for correct quantification (median ARI 0.105, NMI 0.161 in glioblastoma), so low absolute values are expected
- MATES cannot reliably distinguish identical TE copies such as HERVK isoforms due to lack of uniquely mappable flanking regions, which caps achievable clustering gains
- Long autonomously expressed TEs such as L1HS spanning several thousand bases are poorly quantified, biasing locus-level clustering features

## Evidence

- [other] Cluster cells on the TE-only count matrix and on the gene expression matrix, and compute the clustering agreement (median ARI 0.105, median NMI 0.161).: [evidence span withheld — anchor status: not_found]
- [other] computing median ARI and NMI against the published benchmarks (ARI 0.309, NMI 0.438): [evidence span withheld — anchor status: not_found]
- [readme] mates.run() #train model and quantify both subfamily and locus-level TE expression: [evidence span withheld — anchor status: external_doc]
