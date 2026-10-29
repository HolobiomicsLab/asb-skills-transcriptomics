---
name: hrc-hrp-t3ss-gene-annotation
description: Use when mapping known virulence or secretion-system genes (e.g. T3SS
  hrc/hrp) across a multi-genome pan-genome to test whether structural components
  are conserved genus-wide while accessory genes are lifestyle-restricted.
when_to_use_negative:
- When the goal is de novo discovery of lifestyle-associated genes rather than annotation
  of known T3SS reference genes — use the bacLIFE pLAG pipeline (>70% group presence,
  >2 log2 fold change) instead
- When input genomes have not passed CheckM quality filtering (>90% completeness,
  <5% contamination) — low-quality assemblies would yield spurious hrc/hrp absences
- When the target genus lacks a precomputed bacLIFE gene-cluster absence/presence
  matrix, since hit-to-genome mapping depends on that matrix
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0622
tools:
- name: BLASTP
  role: Searches hrc/hrp reference proteins against gene-cluster representatives with
    minimum subject cover 70% and minimum sequence similarity 40%
  repo: https://github.com/Carrion-lab/bacLIFE
- name: Prokka 1.14.6
  role: Predicts genes in each input genome to produce the protein sequences clustered
    into gene clusters
  repo: https://github.com/Carrion-lab/bacLIFE
- name: linclust (MMseqs2)
  role: Clusters predicted genes at >90% identity and 80% minimum coverage (--min-seq-id
    0.9 --cov-mode 0 -c 0.8) to build the gene clusters being queried
  repo: https://github.com/Carrion-lab/bacLIFE
- name: MCL (Markov clustering algorithm)
  role: Performs the gene clustering combined with linclust that defines the cluster
    representatives searched with BLASTP
  repo: https://github.com/Carrion-lab/bacLIFE
- name: bacLIFE
  role: Snakemake-based workflow producing the absence/presence matrix and cluster
    representatives that hrc/hrp hits are mapped back to
  repo: https://github.com/Carrion-lab/bacLIFE
- name: CheckM
  role: Quality-filters input genomes (>90% completeness, <5% contamination) before
    clustering so presence/absence calls are reliable
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:df2ef2ce34732e469a67c682213287b267cc2950875792aa5b89d336b4de749c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Carrion-lab/bacLIFE
  tools:
  - BLASTP
  - CheckM
  - MCL (Markov clustering algorithm)
  - Prokka
  - Prokka 1.14.6
  - bacLIFE
  - linclust (MMseqs2)
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# hrc-hrp-t3ss-gene-annotation

## Summary

Reproduces the bacLIFE Pseudomonas pan-genome analysis mapping the 10 conserved hrc and 16 hrp Type III secretion system genes across genera by BLASTP-searching reference proteins against gene-cluster representatives and tracing hits to genomes via the absence/presence matrix.

## When to use

Use when mapping known virulence or secretion-system genes (e.g. T3SS hrc/hrp) across a multi-genome pan-genome to test whether structural components are conserved genus-wide while accessory genes are lifestyle-restricted.

## When NOT to use

- When the goal is de novo discovery of lifestyle-associated genes rather than annotation of known T3SS reference genes — use the bacLIFE pLAG pipeline (>70% group presence, >2 log2 fold change) instead
- When input genomes have not passed CheckM quality filtering (>90% completeness, <5% contamination) — low-quality assemblies would yield spurious hrc/hrp absences
- When the target genus lacks a precomputed bacLIFE gene-cluster absence/presence matrix, since hit-to-genome mapping depends on that matrix

## Inputs

- Non-redundant, quality-filtered Pseudomonas genome set (Supplementary Datasets 1)
- bacLIFE gene-cluster absence/presence matrix (rows = genomes, columns = cluster IDs)
- bacLIFE gene-cluster representative protein sequences (Prokka-predicted, linclust/MCL clusters)
- Reference protein sequences for the 10 hrc and 16 hrp T3SS genes

## Outputs

- BLASTP hit table linking hrc/hrp queries to gene-cluster representatives
- Binary per-genome hrc/hrp presence/absence table (hrc_hrp_presence_absence.tsv)
- Genus-wide distribution summary distinguishing conserved hrc genes from lifestyle-restricted hrp genes

## How to apply

Load the quality-filtered, non-redundant Pseudomonas genome set (Supplementary Datasets 1; CheckM >90% completeness, <5% contamination; genomes clustered at 0.99 Mash-estimated ANI) together with the bacLIFE gene-cluster absence/presence matrix and cluster representative protein sequences generated by the Snakemake module (Prokka 1.14.6 gene prediction, linclust clustering at --min-seq-id 0.9 --cov-mode 0 -c 0.8, MCL inflation 3.0). Run BLASTP with the 10 hrc and 16 hrp reference protein sequences as queries against the gene-cluster representatives using the paper's annotation thresholds of minimum subject cover 70% and minimum sequence similarity 40%. Map accepted BLASTP hits back to bacLIFE gene cluster IDs, then use the absence/presence matrix to expand each hit cluster to the genomes carrying it. Build a binary per-genome presence/absence table (rows = genomes, columns = hrc/hrp genes) with a custom R/Python script, written as hrc_hrp_presence_absence.tsv. Validate that the table reproduces the reported Fig. 4 pattern: hrc genes broadly distributed across all Pseudomonas lifestyles, hrp genes almost exclusive to plant-pathogenic P. syringae, plus hrc/hrp co-occurrence in non-syringae P. gingeri and P. mendocina strains suggesting horizontal gene transfer.

## Related tools

- **BLASTP** (Searches hrc/hrp reference proteins against gene-cluster representatives with minimum subject cover 70% and minimum sequence similarity 40%) — https://github.com/Carrion-lab/bacLIFE
- **Prokka 1.14.6** (Predicts genes in each input genome to produce the protein sequences clustered into gene clusters) — https://github.com/Carrion-lab/bacLIFE
- **linclust (MMseqs2)** (Clusters predicted genes at >90% identity and 80% minimum coverage (--min-seq-id 0.9 --cov-mode 0 -c 0.8) to build the gene clusters being queried) — https://github.com/Carrion-lab/bacLIFE
- **MCL (Markov clustering algorithm)** (Performs the gene clustering combined with linclust that defines the cluster representatives searched with BLASTP) — https://github.com/Carrion-lab/bacLIFE
- **bacLIFE** (Snakemake-based workflow producing the absence/presence matrix and cluster representatives that hrc/hrp hits are mapped back to) — https://github.com/Carrion-lab/bacLIFE
- **CheckM** (Quality-filters input genomes (>90% completeness, <5% contamination) before clustering so presence/absence calls are reliable)

## Examples

```
makeblastdb -in cluster_reps.faa -dbtype prot && blastp -query hrc_hrp_refs.faa -db cluster_reps.faa -qcov_hsp_perc 70 -out hrc_hrp_hits.tsv
```

## Evaluation signals

- hrc genes are detected as broadly present across genomes spanning all Pseudomonas lifestyles, matching Fig. 4's genus-wide distribution
- hrp genes are recovered almost exclusively from plant-pathogenic P. syringae genomes in the presence/absence table
- Non-syringae hrc/hrp hits are restricted to the expected few strains — four P. gingeri and three P. mendocina — consistent with the reported horizontal gene transfer finding
- Every BLASTP hit satisfies the paper's thresholds of ≥70% subject cover and ≥40% sequence similarity
- hrc_hrp_presence_absence.tsv is a complete binary matrix with one row per input genome and one column per hrc/hrp gene, keyed to bacLIFE cluster IDs

## Limitations

- Threshold-based annotation (70% subject cover, 40% similarity) may miss divergent T3SS homologs below the similarity cutoff
- hrp presence in non-syringae lineages (P. gingeri, P. mendocina) reflects horizontal gene transfer and should not be interpreted as a strict P. syringae-specific marker
- Presence/absence calls depend on genome quality; genomes failing the CheckM filter (>90% completeness, <5% contamination) can produce false absences

## Evidence

- [methods] BLASTP thresholds: "using BLASTP with a minimum subject cover of 70% and a minimum sequence similarity of 40%"
- [methods] linclust parameters: "cluster all sequences that are >90% identical with a minimum coverage of 80% (parameters --min-seq-id 0.9 --cov-mode 0 -c 0.8)"
- [methods] CheckM quality filter: "we used CheckM87 quality analysis to only retain high-quality genomes (>90% completeness and <5% contamination)"
- [methods] Pseudomonas dataset: "each of them with genome datasets corresponding to different bacterial genera: Burkholderia/ Paraburkholderia and Pseudomonas (Supplementary Datasets 1)"
- [readme] bacLIFE description: "bacLIFE is a streamlined computational workflow that annotates bacterial genomes and performs large-scale comparative genomics to predict bacterial lifestyles"
