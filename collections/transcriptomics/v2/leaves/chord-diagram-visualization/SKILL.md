---
name: chord-diagram-visualization
description: Use when you have per-cluster or per-cell-type SNV scores from ScType/vartrix
  for a tumor scRNA-seq sample and need a visual, similarity-based separation of malignant
  vs non-malignant cell populations.
when_to_use_negative:
- Input lacks single-cell SNV scores (e.g. only a raw count matrix without vartrix/CopyKAT
  output) — the chord structure would have no SNV-similarity basis
- Cell clusters are not yet annotated, since the diagram requires labeled cell types
  (e.g. via ScTypeDB_full.xlsx, Immune system tissue) to be interpretable
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: ScType
  role: Provides the per-cluster SNV scoring option whose score similarities populate
    the chord diagram
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: vartrix
  role: Links called SNVs to individual cell barcodes/clusters to build the genotype
    matrix used for scoring
  repo: https://github.com/10XGenomics/vartrix
- name: CopyKAT
  role: Independent copy-number classification (diploid/aneuploid) used to validate
    the malignant/non-malignant split shown in the chord plot
- name: R
  role: Implementation environment for rendering the chord diagram and computing SNV-score
    similarity
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:299634c392cb3147b9d82b7861896fe97003a2376f7fe168349cebce1297a51a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IanevskiAleksandr/sc-type
  tools:
  - CopyKAT
  - R
  - ScType
  - Strelka2
  - samtools
  - vartrix
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# chord-diagram-visualization

## Summary

Render a chord diagram that links cell types by their per-cluster ScType SNV-score similarity, visually separating non-malignant (ScType SNV score <20, CopyKAT-diploid) from malignant populations in an AML scRNA-seq sample.

## When to use

Use when you have per-cluster or per-cell-type SNV scores from ScType/vartrix for a tumor scRNA-seq sample and need a visual, similarity-based separation of malignant vs non-malignant cell populations.

## When NOT to use

- Input lacks single-cell SNV scores (e.g. only a raw count matrix without vartrix/CopyKAT output) — the chord structure would have no SNV-similarity basis
- Cell clusters are not yet annotated, since the diagram requires labeled cell types (e.g. via ScTypeDB_full.xlsx, Immune system tissue) to be interpretable

## Inputs

- Per-cluster/per-cell-type ScType SNV score matrix (from vartrix genotype matrices over COSMIC-restricted Strelka2 calls)
- Seurat cell-type annotations from sctype_score (e.g. CD34+ HSC/MPP, ISG+ blasts, neutrophils, T cells)
- CopyKAT copy-number classification per cell (diploid vs aneuploid)

## Outputs

- Chord diagram linking cell types by SNV-score similarity, with non-malignant and malignant populations visually separated
- Binary malignant/non-malignant assignment per cell type corroborating the chord sectors

## How to apply

After computing per-cluster ScType SNV scores (SNVs called with samtools mpileup/Strelka2 restricted to COSMIC cancer consensus genes, then linked to cell barcodes via vartrix), compute pairwise SNV-score similarity between annotated cell types. Render these similarities as a chord diagram so that chords connect cell types with similar SNV profiles. Decide the separation boundary using the validated thresholds: non-malignant cells (e.g. CD24+ CD66+ neutrophils, memory CD8+ T cells) show ScType SNV score <20 and are CopyKAT-diploid, whereas malignant CD34+ HSC/MPP and ISG+ blast cells carry higher SNV scores and CopyKAT-aneuploid calls. Correct output shows a clean split of the chord plot into non-malignant and malignant sectors.

## Related tools

- **ScType** (Provides the per-cluster SNV scoring option whose score similarities populate the chord diagram) — https://github.com/IanevskiAleksandr/sc-type
- **vartrix** (Links called SNVs to individual cell barcodes/clusters to build the genotype matrix used for scoring) — https://github.com/10XGenomics/vartrix
- **CopyKAT** (Independent copy-number classification (diploid/aneuploid) used to validate the malignant/non-malignant split shown in the chord plot)
- **R** (Implementation environment for rendering the chord diagram and computing SNV-score similarity)

## Examples

```
./vartrix -v variants.vcf -b cellranger.bam -f ref.fa -c barcodes.tsv -o genome.mtx  # then in R: chordDiagram(snv_score_similarity_matrix)
```

## Evaluation signals

- CD34+ HSC/MPP and CD34+ ISG+ blast clusters show higher ScType SNV scores than CD24+ CD66+ neutrophils and memory CD8+ T cells
- Non-malignant cell types (neutrophils, memory CD8+ T cells) have ScType SNV score <20
- CopyKAT assigns majority-diploid status to the same populations scored as non-malignant
- Chord plot shows a clean, unambiguous separation between non-malignant and malignant sectors without cross-linking noise

## Limitations

- Variant calling from RNA-seq is noise-prone; VarTrix documentation notes that 'generating variants from bulk or single cell RNA-seq datasets is challenging. Noise inherent in reverse transcription leads to a high false positive rate'
- VarTrix ignores all multi-allelic sites, which appear as empty rows in the output matrix
- SNV-score discrimination was demonstrated on AML scRNA-seq; generalization to other tumor types is not established in this source

## Evidence

- [other] SNV-score similarity drives the chord separation: [evidence span withheld — anchor status: not_found]
- [other] Thresholds for the non-malignant class: [evidence span withheld — anchor status: not_found]
- [discussion] vartrix links SNVs to clusters: "Next, ScType connects the SNV to each cluster using vartrix"
- [readme] VarTrix false-positive caveat: [evidence span withheld — anchor status: external_doc]
- [abstract] ScType SNV calling purpose: "based on single-cell calling of single-nucleotide variants"
