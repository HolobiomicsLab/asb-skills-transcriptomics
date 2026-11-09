---
name: pangenome-gene-cluster-assignment
description: Use when specific reference protein sequences must be located in a pan-genome
  gene-cluster matrix to build a per-genome presence/absence profile (e.g., mapping
  hrc/hrp genes across Pseudomonas genomes).
when_to_use_negative:
- Input is already a gene presence/absence table or the genes of interest are annotated
  directly in the genome GFFs — direct table lookup suffices without BLASTP.
- The goal is de novo lifestyle-gene discovery (pLAG prediction via machine learning),
  which is bacLIFE's LAG-prediction module rather than targeted reference-gene mapping.
- Genomes fail the quality filter (>90% completeness, <5% contamination per CheckM)
  — spurious absences would corrupt the presence/absence table.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3174
tools:
- name: bacLIFE
  role: Produces the gene-cluster absence/presence matrix and cluster representative
    sequences that the reference genes are mapped onto
  repo: https://github.com/Carrion-lab/bacLIFE
- name: BLASTP
  role: Aligns hrc/hrp reference proteins to gene-cluster representatives using minimum
    subject cover 70% and sequence similarity 40%
- name: Prokka 1.14.6
  role: Performs gene prediction on each input genome prior to clustering
- name: linclust (MMseqs2)
  role: Clusters genes at >90% identity / 80% coverage to form the initial clusters
- name: MCL (Markov clustering algorithm)
  role: Refines gene clustering after linclust (inflation 3.0)
- name: CheckM
  role: Quality-filters genomes (>90% completeness, <5% contamination) before mapping
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:fa3bcebd11fe53584b9f62e9801b1af4f2e44c426c020d39f24c3215727aa940
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# pangenome-gene-cluster-assignment

## Summary

Assign reference gene families (e.g., hrc/hrp T3SS structural genes) to bacLIFE gene clusters and map their absence/presence across a genus-wide pan-genome, using BLASTP against cluster representatives with paper-defined annotation thresholds.

## When to use

Use when specific reference protein sequences must be located in a pan-genome gene-cluster matrix to build a per-genome presence/absence profile (e.g., mapping hrc/hrp genes across Pseudomonas genomes).

## When NOT to use

- Input is already a gene presence/absence table or the genes of interest are annotated directly in the genome GFFs — direct table lookup suffices without BLASTP.
- The goal is de novo lifestyle-gene discovery (pLAG prediction via machine learning), which is bacLIFE's LAG-prediction module rather than targeted reference-gene mapping.
- Genomes fail the quality filter (>90% completeness, <5% contamination per CheckM) — spurious absences would corrupt the presence/absence table.

## Inputs

- Quality-filtered Pseudomonas genome set (Supplementary Datasets 1; CheckM >90% completeness, <5% contamination)
- bacLIFE gene-cluster absence/presence matrix (genome x cluster binary table)
- Gene-cluster representative protein sequences (FASTA) from Prokka + linclust/MCL clustering
- Reference hrc (10) and hrp (16) protein sequences (FASTA)

## Outputs

- BLASTP hit list of hrc/hrp queries vs. gene-cluster representatives passing 70% subject cover / 40% similarity
- Mapping of gene-cluster IDs to genomes via the absence/presence matrix
- Binary per-genome hrc/hrp presence/absence table (hrc_hrp_presence_absence.tsv)

## How to apply

Start from the bacLIFE Snakemake pipeline outputs: Prokka 1.14.6 gene predictions, linclust (MMseqs2) clusters built at >90% identity and 80% coverage (--min-seq-id 0.9 --cov-mode 0 -c 0.8) with MCL (inflation 3.0) grouping, and the resulting gene-cluster absence/presence matrix plus representative protein sequences. Run BLASTP of the 10 hrc and 16 hrp reference proteins against the cluster representatives applying the paper's annotation thresholds: minimum subject cover of 70% and minimum sequence similarity of 40%. Map accepted BLASTP hits back to bacLIFE cluster IDs and, via the absence/presence matrix, to every genome harboring each hit cluster. Build a binary per-genome hrc/hrp presence/absence table (rows = genomes, columns = genes) with a custom R/Python script, e.g., hrc_hrp_presence_absence.tsv. Validate by confirming the table reproduces the reported distribution: hrc genes broadly present across Pseudomonas lifestyles, hrp genes almost exclusive to plant-pathogenic P. syringae plus P. gingeri and P. mendocina strains.

## Related tools

- **bacLIFE** (Produces the gene-cluster absence/presence matrix and cluster representative sequences that the reference genes are mapped onto) — https://github.com/Carrion-lab/bacLIFE
- **BLASTP** (Aligns hrc/hrp reference proteins to gene-cluster representatives using minimum subject cover 70% and sequence similarity 40%)
- **Prokka 1.14.6** (Performs gene prediction on each input genome prior to clustering)
- **linclust (MMseqs2)** (Clusters genes at >90% identity / 80% coverage to form the initial clusters)
- **MCL (Markov clustering algorithm)** (Refines gene clustering after linclust (inflation 3.0))
- **CheckM** (Quality-filters genomes (>90% completeness, <5% contamination) before mapping)

## Examples

```
blastp -query hrc_hrp_refs.faa -subject cluster_reps.faa -outfmt 6 -out hrc_hrp_hits.tsv; then filter hits at 70% subject cover / 40% similarity and join with the bacLIFE absence/presence matrix in R/Python
```

## Evaluation signals

- All BLASTP hits satisfy both thresholds: subject coverage ≥70% and sequence identity/similarity ≥40%.
- Every hit cluster ID resolves to ≥1 genome through the absence/presence matrix; no orphan cluster IDs remain.
- hrc_hrp_presence_absence.tsv has one row per genome and 26 columns (10 hrc + 16 hrp) with strictly binary (0/1) values.
- Resulting distribution matches Fig. 4 reference: hrc genes widespread across Pseudomonas; hrp genes nearly exclusive to P. syringae plus the reported P. gingeri and P. mendocina strains.
- Genomes included all passed the CheckM filter (>90% completeness, <5% contamination).

## Limitations

- hrp detection in non-syringae P. gingeri and P. mendocina strains suggests horizontal gene transfer, so lifestyle cannot be inferred from hrc/hrp presence alone.
- Thresholds (70% subject cover, 40% similarity, >90% identity / 80% coverage clustering) are paper-specific; stricter or looser settings change cluster membership and hit calls.
- bacLIFE clustering granularity differs from alternative tools such as OrthoFinder, so cluster assignments are method-dependent.

## Evidence

- [methods] using BLASTP with a minimum subject cover of 70% and a minimum sequence similarity of 40%: "using BLASTP with a minimum subject cover of 70% and a minimum sequence similarity of 40%"
- [methods] linclust clustering parameters: "cluster all sequences that are >90% identical with a minimum coverage of 80% (parameters --min-seq-id 0.9 --cov-mode 0 -c 0.8)"
- [methods] CheckM quality filter: "we used CheckM87 quality analysis to only retain high-quality genomes (>90% completeness and <5% contamination)"
- [readme] bacLIFE workflow description: "bacLIFE is a streamlined computational workflow that annotates bacterial genomes and performs large-scale comparative genomics to predict bacterial lifestyles"
- [supplementary] genus-level datasets: "each of them with genome datasets corresponding to different bacterial genera: Burkholderia/ Paraburkholderia and Pseudomonas (Supplementary Datasets 1)"
