---
name: ortholog-gene-matching-with-blast
description: Use when matching cell types across a species pair requires gene/gene
  correspondences and a simple sequence-alignment-based ortholog map (e.g., BLAST
  one-to-one matches) is acceptable, or when benchmarking against embedding-based
  gene matching.
when_to_use_negative:
- When higher-quality gene matches are needed — the article reports ProtBERT-based
  gene matching gives better cell type matches than BLAST within TACTiCS
- When input data is single-species, since ortholog matching only applies to cross-species
  comparisons
- When trained TACTiCS models and precomputed protein embeddings are already available,
  making sequence alignment unnecessary
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3170
tools:
- name: BLAST
  role: Sequence alignment tool producing the one-to-one ortholog gene matches used
    by Seurat as the baseline gene-matching layer
- name: Seurat
  role: Consumes BLAST one-to-one gene matches to perform cross-species cell type
    matching for ADS/recall benchmarking
- name: TACTiCS
  role: Alternative pipeline whose ProtBERT-based gene matching is compared against
    the BLAST-based approach
  repo: https://github.com/kbiharie/TACTiCS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:7b58a5bcf1c13ccb255863cbb0f5554d594fe71dcd751eb07a4bbc294876968b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - BLAST
  - PyTorch
  - SAMap
  - Seurat
  - TACTiCS
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# ortholog-gene-matching-with-blast

## Summary

Establishes cross-species one-to-one orthologous gene correspondences using BLAST alignments of protein sequences, providing the gene-matching layer needed for baseline cross-species cell type matching pipelines such as Seurat and as a comparison point against ProtBERT embedding-based gene matching in TACTiCS.

## When to use

Use when matching cell types across a species pair requires gene/gene correspondences and a simple sequence-alignment-based ortholog map (e.g., BLAST one-to-one matches) is acceptable, or when benchmarking against embedding-based gene matching.

## When NOT to use

- When higher-quality gene matches are needed — the article reports ProtBERT-based gene matching gives better cell type matches than BLAST within TACTiCS
- When input data is single-species, since ortholog matching only applies to cross-species comparisons
- When trained TACTiCS models and precomputed protein embeddings are already available, making sequence alignment unnecessary

## Inputs

- Protein sequences (e.g., FASTA) for the genes of each species
- Preprocessed scRNA-seq count data (50k subsets available via Zenodo doi 10.5281/zenodo.7582460) for human, mouse, and marmoset primary motor cortex

## Outputs

- One-to-one orthologous gene correspondence list/matrix per species pair
- Per-cell-type match scores (ADS and recall) at subclass and cross-species label resolutions, e.g. assembled in ads_recall_benchmark.csv

## How to apply

For each species pair (e.g., human–mouse, human–marmoset, mouse–marmoset), run BLAST one-to-one protein sequence searches between the two genomes to obtain orthologous gene pairs. Feed the resulting gene correspondence map into a cross-species cell type matcher — in the TACTiCS benchmark it is used with Seurat to record alignment dissimilarity score (ADS) and recall at both subclass and cross-species label resolutions. Rationale: the article shows that gene matching quality drives cell type matching quality, since ProtBERT-based gene matching within TACTiCS yields better cell type matches than BLAST, making BLAST-matched Seurat a meaningful lower-bound baseline. Compare resulting per-cell-type scores (e.g., the L6b contrast between TACTiCS and SAMap) and assemble ADS/recall values into a results table such as ads_recall_benchmark.csv.

## Related tools

- **BLAST** (Sequence alignment tool producing the one-to-one ortholog gene matches used by Seurat as the baseline gene-matching layer)
- **Seurat** (Consumes BLAST one-to-one gene matches to perform cross-species cell type matching for ADS/recall benchmarking)
- **TACTiCS** (Alternative pipeline whose ProtBERT-based gene matching is compared against the BLAST-based approach) — https://github.com/kbiharie/TACTiCS

## Examples

```
blastp -query human_proteins.fasta -subject mouse_proteins.fasta -outfmt 6 -max_target_seqs 1 -out human_mouse_orthologs.tsv
```

## Evaluation signals

- The resulting gene correspondence map contains strictly one-to-one gene pairs for each species pair
- Resulting cell type matches can be scored with ADS and recall at both subclass and cross-species label resolutions and compared to TACTiCS and SAMap baselines
- ADS/recall values assemble into a complete benchmark table (ads_recall_benchmark.csv) covering all three species pairs
- BLAST-based Seurat results should be a reproducible lower bound relative to TACTiCS, which achieves higher ADS and recall at subclass resolution across all three pairwise comparisons

## Limitations

- BLAST-based gene matching underperforms embedding-based matching: ProtBERT gene matching within TACTiCS yields better cell type matches than BLAST
- Requires one-to-one orthologs, which may be unavailable or ambiguous for lineage-specific genes
- Benchmark scope limited to primary motor cortex scRNA-seq of human, mouse, and marmoset

## Evidence

- [abstract] Seurat BLAST baseline: "our model outperforms Seurat and the state-of-the-art method SAMap"
- [abstract] BLAST comparison: "our gene matching method results in better cell type matches than BLAST in our model"
- [results] workflow Seurat step: [evidence span withheld — anchor status: not_found]
- [methods] TACTiCS usage: "As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities."
