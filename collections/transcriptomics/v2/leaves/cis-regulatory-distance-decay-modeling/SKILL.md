---
name: cis-regulatory-distance-decay-modeling
description: Use when preprocessing paired scRNA-seq/scATAC-seq for cnmf joint clustering
  and you need a peak-gene coupling matrix that weights open chromatin peaks by distance
  to genes (d0 = 40 kb decay).
when_to_use_negative:
- You already have consistent cluster assignments for the scRNA-seq and scATAC-seq
  samples obtained elsewhere — the cnmf/coupling-matrix route is explicitly optional.
- The comparison condition has single-donor data processed directly through sc_compreg
  with clusters supplied by another method that already links subpopulations.
- The scATAC-seq peak coordinates do not map to one of the supported genome versions
  (hg19, hg38, mm9, mm10), since cnmf_process_data.sh cannot build the coupling prior.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: sc-compReg
  role: Provides cnmf_process_data.sh, cnmf_load_coupling_matrix, and the CoupledNMF
    joint clustering workflow that consumes the distance-decay coupling matrix
  repo: https://github.com/SUwonglab/sc-compReg
- name: scCompReg
  role: R package implementing cnmf_tsne visualization and sc_compreg, which takes
    the cnmf cluster labels as cluster-assignment inputs
  repo: https://github.com/SUwonglab/sc-compReg
- name: Bedtools
  role: Linux dependency used by the preprocess_data scripts to intersect peaks with
    genes when building the coupling matrix
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25089-2
    title: Sc-compReg enables the comparison of gene regulatory networks between conditions
      using single-cell data
  source_content_hash: sha256:b2049b9a386541c1f43489cf25f0797230fba2f7c13ccf9da737eb710bf5bc9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/sc-compReg
  tools:
  - Bedtools
  - R (>=)
  - sc-compReg
  - scCompReg
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cis-regulatory distance-decay modeling

## Summary

Links scATAC-seq peaks to putatively regulated co-located genes via a coupling term A_ij = R_ij+ * exp(-d_ij/d0) with distance-decay scale d0 = 40 kb, which feeds the peak-gene coupling matrix D used by the CoupledNMF (cnmf) joint clustering step of sc-compReg.

## When to use

Use when preprocessing paired scRNA-seq/scATAC-seq for cnmf joint clustering and you need a peak-gene coupling matrix that weights open chromatin peaks by distance to genes (d0 = 40 kb decay).

## When NOT to use

- You already have consistent cluster assignments for the scRNA-seq and scATAC-seq samples obtained elsewhere — the cnmf/coupling-matrix route is explicitly optional.
- The comparison condition has single-donor data processed directly through sc_compreg with clusters supplied by another method that already links subpopulations.
- The scATAC-seq peak coordinates do not map to one of the supported genome versions (hg19, hg38, mm9, mm10), since cnmf_process_data.sh cannot build the coupling prior.

## Inputs

- peak.bed file of scATAC-seq peak coordinates (chr, start, end)
- genome_version identifier (hg19, hg38, mm9, or mm10)
- prior_data directory from the sc-compReg repository
- peak.name vector and gene symbol vector in R
- peak_gene_coupling_matrix.txt produced by cnmf_process_data.sh

## Outputs

- peak_gene_coupling_matrix.txt (peak-gene coupling file)
- Coupling matrix D in R (A_ij = R_ij+ * exp(-d_ij/d0), d0 = 40 kb)
- Consistent cnmf cluster labels for sample 1 and sample 2 (H1, H2 factor matrices) passed as O1.idx, E1.idx, O2.idx, E2.idx

## How to apply

Preprocess the peak annotation with cnmf_process_data.sh (bash cnmf_process_data.sh path/to/peak.bed genome_version path/to/prior_data, genome_version one of hg19/hg38/mm9/mm10) to produce peak_gene_coupling_matrix.txt. Load it in R with cnmf_load_coupling_matrix('peak_gene_coupling_matrix.txt', peak.name, symbol) to obtain the coupling matrix D used in the coupled multiplicative updates for W1, H1, W2, H2. Each peak-gene coupling A_ij is defined as R_ij+ * exp(-d_ij/d0), where d_ij is the peak-to-gene distance and d0 = 40 kb is the distance-decay scale, so cis-regulatory influence decays smoothly with distance. The resulting coupling lets CoupledNMF jointly factorize the scRNA-seq and scATAC-seq matrices, yielding consistent cluster labels for both modalities that are passed to sc_compreg as O1.idx, E1.idx, O2.idx, E2.idx. This distance-decay modeling is optional: sc_compreg only requires some consistent cluster assignments across RNA and ATAC cells, which can be obtained elsewhere.

## Related tools

- **sc-compReg** (Provides cnmf_process_data.sh, cnmf_load_coupling_matrix, and the CoupledNMF joint clustering workflow that consumes the distance-decay coupling matrix) — https://github.com/SUwonglab/sc-compReg
- **scCompReg** (R package implementing cnmf_tsne visualization and sc_compreg, which takes the cnmf cluster labels as cluster-assignment inputs) — https://github.com/SUwonglab/sc-compReg
- **Bedtools** (Linux dependency used by the preprocess_data scripts to intersect peaks with genes when building the coupling matrix)

## Examples

```
bash cnmf_process_data.sh path/to/peak.bed hg38 path/to/prior_data; D <- cnmf_load_coupling_matrix('peak_gene_coupling_matrix.txt', peak.name, symbol)
```

## Evaluation signals

- peak_gene_coupling_matrix.txt is generated without error for the chosen genome_version and loads into D via cnmf_load_coupling_matrix using matching peak.name and symbol vectors.
- The cnmf run converges and yields cluster labels present for both sample 1 and sample 2 with consistent labels across the RNA and ATAC modalities.
- The stored cluster assignments can be loaded (e.g., saved as cnmf_output.rds) and passed directly to sc_compreg as O1.idx, E1.idx, O2.idx, E2.idx.
- cnmf_tsne(cnmf.output$H1, cnmf.output$H2, path=path, save.plot=F, perplexity=100) produces joint cluster and data-type plots in which clusters co-localize across modalities.
- Couplings A_ij decay monotonically with peak-to-gene distance d_ij, bounded by the exp(-d_ij/40kb) decay scale.

## Limitations

- The coupling prior depends on precomputed ENCODE-style prior data for hg19, hg38, mm9, or mm10 only; other genome builds are unsupported by cnmf_process_data.sh.
- This workflow only produces cluster assignments; differential regulatory analysis requires the downstream sc_compreg step and log2-transformed expression/accessibility matrices.
- sc-compReg currently assumes one scRNA-seq + one scATAC-seq sample per condition and does not yet accommodate multiple donors per condition.

## Evidence

- [methods] Preprocess data for cnmf: "Preproces data for `cnmf`"
- [methods] Run cnmf to get cluster labels: "Run `cnmf` to get the cluster labels for sample 1 and sample 2."
- [readme] cnmf preprocessing command: "bash cnmf_process_data.sh path/to/peak.bed genome_version path/to/prior_data"
- [readme] cnmf_tsne visualization: "`cnmf_tsne` allows users to visualize clustering outputs."
- [readme] Cluster assignments are optional for scCompReg: "It is not required to obtain cluster assignments using the coupled nonnegative matrix factorization workflow."
