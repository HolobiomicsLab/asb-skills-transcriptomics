---
name: genome-transcriptome-coordinate-transform
description: Use when phasiRNA/siRNA clusters were predicted separately against a
  reference genome and a reference transcriptome and need to be unified into a single,
  deduplicated PHAS locus set with gene annotations.
when_to_use_negative:
- Predictions were made on only a single reference sequence (no genome/transcriptome
  duality), so there is nothing to transform or merge — use the phase module outputs
  directly instead.
- Input cluster files are not phase module outputs (phase_o.txt/phase_a.txt) or the
  annotation is not GFF3 format, since the integration module expects these specific
  file types.
- The analysis only needs raw phasiRNA cluster prediction with no cross-reference
  unification or gene annotation — the phase module alone suffices.
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3170
tools:
- name: PhasiHunter
  role: Provides the integration module that performs the coordinate transform, deduplication,
    merging, annotation, and classification of phasiRNA clusters across reference
    systems
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bedtools
  role: Implements the coordinate system transform between genome and transcriptome
    coordinate systems during result integration
- name: Biopython
  role: Python dependency used by PhasiHunter (with python3 dictionary data structure)
    for the integration logic
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:134256b2cec905444a546188285ae26edb804b9283fe7568c3ae3236a54fad12
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
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# genome-transcriptome-coordinate-transform

## Summary

PhasiHunter's integration step merges phasiRNA/all-siRNA cluster predictions made independently on cDNA (transcriptome) and gDNA (genome) references by transforming coordinates between the two reference systems with bedtools and a Python3 dictionary data structure, then deduplicating, merging, annotating, and classifying clusters against GFF3 gene annotation.

## When to use

Use when phasiRNA/siRNA clusters were predicted separately against a reference genome and a reference transcriptome and need to be unified into a single, deduplicated PHAS locus set with gene annotations.

## When NOT to use

- Predictions were made on only a single reference sequence (no genome/transcriptome duality), so there is nothing to transform or merge — use the phase module outputs directly instead.
- Input cluster files are not phase module outputs (phase_o.txt/phase_a.txt) or the annotation is not GFF3 format, since the integration module expects these specific file types.
- The analysis only needs raw phasiRNA cluster prediction with no cross-reference unification or gene annotation — the phase module alone suffices.

## Inputs

- phase_o.txt: phasiRNA cluster output from the phase module (produced from both cDNA and gDNA reference mappings)
- phase_a.txt: all-siRNA cluster output from the phase module
- Reference genome GFF3 annotation file (e.g. oryza_sativa_gdna.gff3)
- Reference genome and transcriptome FASTA files used as the coordinate systems being transformed between (e.g. oryza_sativa_gdna.fa, oryza_sativa_cdna.fa)

## Outputs

- integration_o.txt: integrated phasiRNA clusters (table with fields PHAS_gene, strand, phasiRNA_position, phasiRNA_abundance, phasiRNA_record, phasiRNA_sequence, phasiRNA_length, phase_ratio, phase_number, phase_abundance, phase_score, pvalue, PHAS_gene_annotation, marker)
- integration_a.txt: integrated all-siRNA clusters
- integration_s.txt: integration summary
- integration_p.txt: PHAS loci information

## How to apply

Run the PhasiHunter integration module, supplying the phase module outputs phase_o.txt (phasiRNA clusters) and phase_a.txt (all-siRNA clusters) from both the cDNA and gDNA reference mappings via -io and -ia, plus the reference genome GFF3 annotation (e.g. oryza_sativa_gdna.gff3) via -an. Set the gDNA-based PHAS loci flag to y so genome- and transcriptome-coordinate clusters are jointly processed; the module performs the coordinate system transform using bedtools and a Python3 dictionary structure. Re-apply the same prediction cutoffs used in the phase step (phase number 4 via -pn, phase length 21 via -pl, pvalue cutoff 0.001 via -pv, cluster island 5 via -il) so integrated clusters meet the original significance criteria. Keep the default -dp y setting to discard results supported only by the phase-score method, then annotate integrated PHAS loci and siRNA clusters against the GFF3 gene annotation. The module emits four outputs: integration_o.txt, integration_a.txt, integration_s.txt (summary), and integration_p.txt (PHAS loci information).

## Related tools

- **PhasiHunter** (Provides the integration module that performs the coordinate transform, deduplication, merging, annotation, and classification of phasiRNA clusters across reference systems) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bedtools** (Implements the coordinate system transform between genome and transcriptome coordinate systems during result integration)
- **Biopython** (Python dependency used by PhasiHunter (with python3 dictionary data structure) for the integration logic)

## Examples

```
phasiHunter integration -io /home/user/test_osa/phase_o.txt -ia /home/user/test_osa/phase_a.txt -an /home/user/test_osa/oryza_sativa_gdna.gff3 -g y -o /home/user/test_osa/integration_o.txt -a /home/user/test_osa/integration_a.txt -s /home/user/test_osa/integration_s.txt -po /home/user/test_osa/integration_p.txt -j 1 -pn 4 -pl 21 -pv 0.001 -il 5
```

## Evaluation signals

- integration_s.txt exists after the integration run, confirming the module completed.
- integration_o.txt contains the expected table header fields: PHAS_gene, strand, phasiRNA_position, phasiRNA_abundance, phasiRNA_record, phasiRNA_sequence, phasiRNA_length, phase_ratio, phase_number, phase_abundance, phase_score, pvalue, PHAS_gene_annotation, marker.
- Every reported integrated cluster satisfies the reapplied cutoffs: phase number >= 4, phase length 21, pvalue < 0.001, cluster island 5.
- Clusters supported only by the phase-score method (P method) are absent when the default discard setting (-dp y) is in effect.
- Integrated clusters are annotated via PHAS_gene_annotation against the supplied GFF3 gene annotation, with gDNA-enabled (g=y) loci present alongside transcriptome-coordinate clusters.

## Limitations

- Results supported only by the phase-score method are discarded by default (-dp y), so phase-score-only predictions are excluded from integrated output unless this is disabled.
- Integration depends on having both the cDNA-based and gDNA-based phase module outputs and a genome GFF3 annotation; missing any of these prevents the joint processing of genome- and transcriptome-coordinate clusters.
- No discussion-section content was available in the article, so additional limitations or reproducibility details are not extractable from that section.

## Evidence

- [methods] integration via bedtools and dictionary: [evidence span withheld — anchor status: not_found]
- [abstract] automatic integration claim: "PhasiHunter can identify phasiRNAs and their regulatory networks based on multiple reference sequences and the predicted results can be automatically integrated"
- [readme] integration CLI: "phasiHunter integration -io /home/user/test_osa/phase_o.txt -ia /home/user/test_osa/phase_a.txt -an /home/user/test_osa/oryza_sativa_gdna.gff3 -g y -o /home/user/test_osa/integration_o.txt"
- [readme] integration parameters: "-j 1 -pn 4 -pl 21 -pv 0.001 -il 5"
- [methods] discard only P method result: "-dp: str   -- y | n, discard only P method result, default=y"
