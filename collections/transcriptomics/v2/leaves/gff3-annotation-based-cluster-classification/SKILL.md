---
name: gff3-annotation-based-cluster-classification
description: Use when phasiRNA/siRNA cluster predictions exist in multiple coordinate
  systems (cDNA vs gDNA mapping) and must be merged, deduplicated, and annotated with
  GFF3 gene features into a unified PHAS loci catalog.
when_to_use_negative:
- Predictions were produced against only a single reference sequence, so no cross-coordinate-system
  integration is needed.
- The GFF3 gene annotation for the reference genome is unavailable, since annotation
  and classification of integrated PHAS loci depend on it.
- Cluster merging is desired with different thresholds than the phase step without
  rerunning, e.g. the phase number cutoff 4 or pvalue cutoff 0.001 cannot be re-applied
  consistently.
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: PhasiHunter
  role: Provides the integration submodule that deduplicates, merges, annotates, and
    classifies phasiRNA/all-siRNA clusters across reference coordinate systems
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bedtools
  role: Implements the coordinate system transform between genome- and transcriptome-based
    cluster coordinates during integration
- name: Biopython
  role: Supports sequence/annotation handling in the python3 integration workflow
- name: Bowtie
  role: Upstream mapper producing the cDNA- and gDNA-based map files whose cluster
    outputs feed integration
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:a6bde918502ea3f8a277c1e41ef76d353d30b6e298ac83152a927b4d0c5f5329
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuangLab-CBI/PhasiHunter
  tools:
  - Bedtools
  - Biopython
  - Bowtie
  - PhasiHunter
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gff3-annotation-based-cluster-classification

## Summary

Integrates phasiRNA and all-siRNA clusters predicted separately against different reference sequences (transcriptome/cDNA and genome/gDNA) by transforming coordinates with bedtools and python3 dictionaries, then deduplicating, merging, annotating and classifying the merged PHAS loci against a reference GFF3 gene annotation.

## When to use

Use when phasiRNA/siRNA cluster predictions exist in multiple coordinate systems (cDNA vs gDNA mapping) and must be merged, deduplicated, and annotated with GFF3 gene features into a unified PHAS loci catalog.

## When NOT to use

- Predictions were produced against only a single reference sequence, so no cross-coordinate-system integration is needed.
- The GFF3 gene annotation for the reference genome is unavailable, since annotation and classification of integrated PHAS loci depend on it.
- Cluster merging is desired with different thresholds than the phase step without rerunning, e.g. the phase number cutoff 4 or pvalue cutoff 0.001 cannot be re-applied consistently.

## Inputs

- phase module phasiRNA cluster output file phase_o.txt (from -io)
- phase module all-siRNA cluster output file phase_a.txt (from -ia)
- reference genome GFF3 annotation file, e.g. oryza_sativa_gdna.gff3 (from -an)
- gDNA-based PHAS loci enablement flag y|n (from -g)

## Outputs

- integration_o.txt — integrated phasiRNA clusters (PHAS_gene, strand, phasiRNA_position, phasiRNA_abundance, phasiRNA_record, phasiRNA_sequence, phasiRNA_length, phase_ratio, phase_number, phase_abundance, phase_score, pvalue, PHAS_gene_annotation, marker)
- integration_a.txt — integrated all-siRNA clusters
- integration_s.txt — integration summary
- integration_p.txt — PHAS loci information

## How to apply

Load the phase module outputs phase_o.txt (phasiRNA clusters) and phase_a.txt (all-siRNA clusters) produced from both the cDNA and gDNA reference mappings. Run the PhasiHunter integration module with the reference genome GFF3 annotation (e.g. oryza_sativa_gdna.gff3) and enable gDNA-based PHAS loci (-g y) so genome- and transcriptome-coordinate clusters are jointly processed. Apply the same cutoffs used in prediction — phase number 4 (-pn), phase length 21 (-pl), pvalue cutoff 0.001 (-pv), cluster island 5 (-il) — while deduplicating and merging overlapping clusters across coordinate systems; with the default discard setting (-dp y) results supported only by the phase-score method are dropped, keeping hypergeometric-test-supported loci. Bedtools performs the coordinate system transform and python3 dictionaries drive merging, after which clusters are annotated with GFF3 gene features and classified. The module emits four outputs: integration_o.txt, integration_a.txt, integration_s.txt, and integration_p.txt.

## Related tools

- **PhasiHunter** (Provides the integration submodule that deduplicates, merges, annotates, and classifies phasiRNA/all-siRNA clusters across reference coordinate systems) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bedtools** (Implements the coordinate system transform between genome- and transcriptome-based cluster coordinates during integration)
- **Biopython** (Supports sequence/annotation handling in the python3 integration workflow)
- **Bowtie** (Upstream mapper producing the cDNA- and gDNA-based map files whose cluster outputs feed integration)

## Examples

```
phasiHunter integration -io /home/user/test_osa/phase_o.txt -ia /home/user/test_osa/phase_a.txt -an /home/user/test_osa/oryza_sativa_gdna.gff3 -g y -o /home/user/test_osa/integration_o.txt -a /home/user/test_osa/integration_a.txt -s /home/user/test_osa/integration_s.txt -po /home/user/test_osa/integration_p.txt -j 1 -pn 4 -pl 21 -pv 0.001 -il 5
```

## Evaluation signals

- integration_s.txt (integration summary) exists in the output directory.
- integration_o.txt contains the expected table header fields: PHAS_gene, strand, phasiRNA_position, phasiRNA_abundance, phasiRNA_record, phasiRNA_sequence, phasiRNA_length, phase_ratio, phase_number, phase_abundance, phase_score, pvalue, PHAS_gene_annotation, marker.
- Integrated clusters satisfy the applied cutoffs: phase number 4, phase length 21, pvalue 0.001, cluster island 5.
- Results supported only by the phase-score method are discarded when the default -dp y setting is in effect.
- Clusters from both genome and transcriptome coordinate systems appear in the merged output when -g y is set.

## Limitations

- Only a version update log entry (1.0.1, 25/02/2024) is given; no explicit version number, discussion of limitations, or reproducibility details are documented.
- With default -dp y, predictions supported only by the phase-score method are discarded, so phase-score-only loci will be absent from the integrated output.
- Integration requires phase module outputs from both cDNA and gDNA mappings plus a genome GFF3; it cannot be run from raw reads alone.

## Evidence

- [methods] integration module performs deduplication, merging, annotation, and classification of phasiRNA clusters using GFF3 and full-length transcriptome annotation files.: [evidence span withheld — anchor status: not_found]
- [methods] bedtools dictionary transform: [evidence span withheld — anchor status: not_found]
- [readme] integration usage options: "-io: file  --  phase module -o output file
    -ia: file  --  phase module -a output file"
- [readme] integration command with cutoffs: "phasiHunter integration -io /home/user/test_osa/phase_o.txt -ia /home/user/test_osa/phase_a.txt -an /home/user/test_osa/oryza_sativa_gdna.gff3 -g y -o /home/user/test_osa/integration_o.txt"
- [methods] discard P-only default: "-dp: str   -- y | n, discard only P method result, default=y"
