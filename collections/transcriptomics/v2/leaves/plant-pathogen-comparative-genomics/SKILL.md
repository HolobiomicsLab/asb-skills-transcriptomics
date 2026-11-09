---
name: plant-pathogen-comparative-genomics
description: Use when you have a quality-filtered pan-genome set with gene-cluster
  absence/presence data and want to map reference gene families (e.g. secretion-system
  genes) across strains to test lifestyle-associated distribution patterns.
when_to_use_negative:
- Input is already a gene presence/absence or feature table and the question is lifestyle
  classification via machine learning (use bacLIFE's lifestyle-prediction module instead
  of re-mapping genes with BLASTP)
- Genomes have not been quality filtered or redundancy-reduced (raw NCBI downloads
  should first pass CheckM >90% completeness / <5% contamination and 0.99 Mash-ANI
  clustering)
- The target gene family lacks validated reference protein sequences, so BLASTP hits
  at 70% subject cover / 40% similarity cannot be trusted
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3299
tools:
- name: bacLIFE
  role: Generates the gene-cluster absence/presence matrix and cluster representative
    sequences that BLASTP hits are mapped onto
  repo: https://github.com/Carrion-lab/bacLIFE
- name: Prokka 1.14.6
  role: Gene prediction on each input genome prior to clustering
- name: linclust (MMseqs2)
  role: Clusters predicted proteins at >90% identity and 80% coverage (--min-seq-id
    0.9 --cov-mode 0 -c 0.8) to form gene clusters
- name: MCL (Markov clustering algorithm)
  role: Refines linclust clusters into similar-function gene clusters (with diamond
    all-vs-all at e-value 1e-5)
- name: BLASTP
  role: Searches reference hrc/hrp proteins against cluster representatives at ≥70%
    subject cover and ≥40% sequence similarity
- name: CheckM
  role: Quality analysis retaining only high-quality genomes (>90% completeness, <5%
    contamination) as the mapping input set
- name: Snakemake
  role: Orchestrates the annotation/clustering module producing the absence/presence
    matrix
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:909ab882ba6f6abc014bb74c43ed410be89aab30ee9f012044c3569bd8587adc
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
  - Snakemake
  - bacLIFE
  - linclust (MMseqs2)
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# plant-pathogen-comparative-genomics

## Summary

A comparative genomics skill for mapping candidate lifestyle/pathogenicity genes (e.g. T3SS hrc/hrp genes) across a bacterial pan-genome using bacLIFE gene clusters and BLASTP-based presence/absence profiling, to test whether core structural genes are conserved while accessory virulence genes are lifestyle-restricted.

## When to use

Use when you have a quality-filtered pan-genome set with gene-cluster absence/presence data and want to map reference gene families (e.g. secretion-system genes) across strains to test lifestyle-associated distribution patterns.

## When NOT to use

- Input is already a gene presence/absence or feature table and the question is lifestyle classification via machine learning (use bacLIFE's lifestyle-prediction module instead of re-mapping genes with BLASTP)
- Genomes have not been quality filtered or redundancy-reduced (raw NCBI downloads should first pass CheckM >90% completeness / <5% contamination and 0.99 Mash-ANI clustering)
- The target gene family lacks validated reference protein sequences, so BLASTP hits at 70% subject cover / 40% similarity cannot be trusted

## Inputs

- Non-redundant, quality-filtered genome set (e.g. Pseudomonas Supplementary Dataset 1; CheckM >90% completeness, <5% contamination)
- bacLIFE gene-cluster absence/presence matrix
- bacLIFE cluster representative protein sequences (FASTA) from Prokka + linclust (>90% identity, 80% coverage) + MCL clustering
- Reference protein sequences for target gene families (e.g. 10 hrc and 16 hrp T3SS genes) in FASTA format

## Outputs

- BLASTP hit list linking reference genes to gene-cluster representatives (subject cover ≥70%, similarity ≥40%)
- Binary per-genome target-gene presence/absence table (e.g. hrc_hrp_presence_absence.tsv, rows = genomes, columns = hrc/hrp genes)
- Distribution summary supporting lifestyle-association interpretation (e.g. hrc widespread, hrp restricted to P. syringae plus P. gingeri and P. mendocina strains)

## How to apply

Start from the non-redundant, quality-filtered genome set (e.g. Supplementary Datasets 1, filtered with CheckM at >90% completeness and <5% contamination) together with the bacLIFE gene-cluster absence/presence matrix and cluster representative proteins produced by the Snakemake module (Prokka 1.14.6 gene prediction, linclust clustering at >90% identity / 80% coverage with --min-seq-id 0.9 --cov-mode 0 -c 0.8, MCL inflation 3.0). Run BLASTP of reference protein sequences (e.g. 10 hrc and 16 hrp T3SS genes) against cluster representatives using the paper's thresholds: minimum subject cover 70% and minimum sequence similarity 40%. Map BLASTP hits back to bacLIFE gene cluster IDs and, through the absence/presence matrix, to the genomes carrying each cluster. Build a binary per-genome presence/absence table (rows = genomes, columns = target genes) with a custom R/Python script. Rationale: clustering genes into similar-function clusters before mapping avoids redundant all-vs-all searches and lets distribution patterns be interpreted at pan-genome scale, as in the Pseudomonas hrc/hrp analysis.

## Related tools

- **bacLIFE** (Generates the gene-cluster absence/presence matrix and cluster representative sequences that BLASTP hits are mapped onto) — https://github.com/Carrion-lab/bacLIFE
- **Prokka 1.14.6** (Gene prediction on each input genome prior to clustering)
- **linclust (MMseqs2)** (Clusters predicted proteins at >90% identity and 80% coverage (--min-seq-id 0.9 --cov-mode 0 -c 0.8) to form gene clusters)
- **MCL (Markov clustering algorithm)** (Refines linclust clusters into similar-function gene clusters (with diamond all-vs-all at e-value 1e-5))
- **BLASTP** (Searches reference hrc/hrp proteins against cluster representatives at ≥70% subject cover and ≥40% sequence similarity)
- **CheckM** (Quality analysis retaining only high-quality genomes (>90% completeness, <5% contamination) as the mapping input set)
- **Snakemake** (Orchestrates the annotation/clustering module producing the absence/presence matrix)

## Examples

```
blastp -query hrc_hrp_reference_proteins.faa -subject bacLIFE_cluster_representatives.faa -outfmt 6 -qcovhsp 70 -out hrc_hrp_hits.tsv
```

## Evaluation signals

- Every BLASTP hit satisfies the annotation thresholds: minimum subject cover 70% and minimum sequence similarity 40%
- The output presence/absence table is binary (0/1) with rows = genomes and columns = the 10 hrc + 16 hrp genes, matching hrc_hrp_presence_absence.tsv schema
- Reproduces the reported Fig. 4 pattern: hrc genes broadly distributed across Pseudomonas lifestyles while hrp genes are almost exclusive to plant-pathogenic P. syringae
- Known non-syringae carriers are recovered: four P. gingeri and three P. mendocina strains show hrc/hrp presence, consistent with horizontal gene transfer
- Genome set matches the quality-filtered input: only genomes passing CheckM >90% completeness and <5% contamination are represented as rows

## Limitations

- Detection depends on BLASTP thresholds (70% subject cover, 40% similarity); divergent homologs below these cutoffs will be missed
- Presence is inferred at the gene-cluster level, so paralogs grouped into the same cluster may inflate apparent presence
- Lifestyle-restriction conclusions (e.g. hrp exclusive to P. syringae) depend on genome sampling; the P. gingeri and P. mendocina hits show distributions can extend beyond the focal lifestyle via horizontal gene transfer

## Evidence

- [methods] using BLASTP with a minimum subject cover of 70% and a minimum sequence similarity of 40%: "using BLASTP with a minimum subject cover of 70% and a minimum sequence similarity of 40%"
- [methods] cluster all sequences that are >90% identical with a minimum coverage of 80%: "cluster all sequences that are >90% identical with a minimum coverage of 80% (parameters --min-seq-id 0.9 --cov-mode 0 -c 0.8)"
- [methods] we used CheckM quality analysis to only retain high-quality genomes: "we used CheckM87 quality analysis to only retain high-quality genomes (>90% completeness and <5% contamination)"
- [readme] bacLIFE is a streamlined computational workflow that annotates bacterial genomes and performs large-scale comparative genomics: "bacLIFE is a streamlined computational workflow that annotates bacterial genomes and performs large-scale comparative genomics to predict bacterial lifestyles"
- [results] hrc genes were widely spread among all members of the Pseudomonas genus, whereas the hrp genes were mostly exclusively present in plant pathogenic P. syringae: "hrc genes were widely spread among all members of the Pseudomonas genus, whereas the hrp genes were mostly exclusively present in plant pathogenic P. syringae"
