---
name: phased-sirna-loci-deduplication-and-merging
description: Use when phasiRNA prediction (phase module) outputs exist from mappings
  to more than one reference sequence — e.g. a cDNA map and a gDNA map — and clusters
  in different coordinate systems must be reconciled into one unified PHAS locus set.
when_to_use_negative:
- You have prediction results from only a single reference sequence — no cross-reference
  coordinate transform or merging is needed.
- You are still in the prediction stage (raw sRNA data preprocessing or the phase
  module); this skill operates only on phase module outputs (phase_o.txt / phase_a.txt),
  not on .map or FASTQ inputs.
- You need initiator or degradome target verification (the 'target'/'initiator' modules)
  rather than locus consolidation.
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0080
tools:
- name: PhasiHunter
  role: Provides the 'integration' submodule (phasiHunter integration) that performs
    deduplication, merging, annotation, and classification of phasiRNA clusters across
    coordinate systems
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bedtools
  role: Implements the coordinate system transform used to reconcile clusters predicted
    on different reference sequences
- name: Biopython
  role: Python library dependency supporting the dictionary-based merging logic of
    the integration module
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:0f66a9f4a753e9052463c081ce3a49298efda22ac6be91aa17a0c52c0b959b55
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
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# phased-sirna-loci-deduplication-and-merging

## Summary

Integrates phasiRNA/PHAS locus prediction results produced separately on different reference sequences (transcriptome, genome, full-length transcriptome) by a coordinate-system transform implemented with bedtools and python3 dictionaries, then deduplicates, merges, annotates, and classifies phasiRNA clusters.

## When to use

Use when phasiRNA prediction (phase module) outputs exist from mappings to more than one reference sequence — e.g. a cDNA map and a gDNA map — and clusters in different coordinate systems must be reconciled into one unified PHAS locus set.

## When NOT to use

- You have prediction results from only a single reference sequence — no cross-reference coordinate transform or merging is needed.
- You are still in the prediction stage (raw sRNA data preprocessing or the phase module); this skill operates only on phase module outputs (phase_o.txt / phase_a.txt), not on .map or FASTQ inputs.
- You need initiator or degradome target verification (the 'target'/'initiator' modules) rather than locus consolidation.

## Inputs

- phase module phasiRNA cluster output file (phase_o.txt, via -io)
- phase module all-siRNA cluster output file (phase_a.txt, via -ia)
- reference genome GFF3 annotation file (e.g. oryza_sativa_gdna.gff3, via -an)
- flag enabling gdna-based PHAS loci (-g y)
- cutoff parameters: phase number (-pn 4), phase length (-pl 21), pvalue cutoff (-pv 0.001), cluster island (-il 5)

## Outputs

- integration_o.txt — integrated phasiRNA clusters (table with PHAS_gene, strand, phasiRNA_position, phasiRNA_abundance, phasiRNA_record, phasiRNA_sequence, phasiRNA_length, phase_ratio, phase_number, phase_abundance, phase_score, pvalue, PHAS_gene_annotation, marker)
- integration_a.txt — integrated all-siRNA clusters
- integration_s.txt — integration summary
- integration_p.txt — PHAS loci information

## How to apply

Load the phase module outputs (phase_o.txt phasiRNA clusters and phase_a.txt all-siRNA clusters) generated from the cDNA- and gDNA-based mappings, then invoke the PhasiHunter integration module with the reference genome GFF3 annotation and gdna-based PHAS loci enabled (-g y) so genome- and transcriptome-coordinate clusters are jointly processed. The module performs a coordinate transform with bedtools and python3 dictionary structures to place clusters into a common coordinate system, then deduplicates and merges overlapping phasiRNA clusters under the same cutoffs used at prediction time (phase number 4, phase length 21, pvalue cutoff 0.001, cluster island 5). Optionally discard clusters supported only by the phase-score method (-dp y) to retain results backed by the hypergeometric test. Finally, annotate the integrated PHAS loci and siRNA clusters using the GFF3 gene annotation and emit the four integration outputs: integration_o.txt, integration_a.txt, integration_s.txt, and integration_p.txt.

## Related tools

- **PhasiHunter** (Provides the 'integration' submodule (phasiHunter integration) that performs deduplication, merging, annotation, and classification of phasiRNA clusters across coordinate systems) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bedtools** (Implements the coordinate system transform used to reconcile clusters predicted on different reference sequences)
- **Biopython** (Python library dependency supporting the dictionary-based merging logic of the integration module)

## Examples

```
phasiHunter integration -io /home/user/test_osa/phase_o.txt -ia /home/user/test_osa/phase_a.txt -an /home/user/test_osa/oryza_sativa_gdna.gff3 -g y -o /home/user/test_osa/integration_o.txt -a /home/user/test_osa/integration_a.txt -s /home/user/test_osa/integration_s.txt -po /home/user/test_osa/integration_p.txt -j 1 -pn 4 -pl 21 -pv 0.001 -il 5
```

## Evaluation signals

- integration_s.txt (integration summary) exists after the run
- integration_o.txt contains the expected table header fields: PHAS_gene, strand, phasiRNA_position, phasiRNA_abundance, phasiRNA_record, phasiRNA_sequence, phasiRNA_length, phase_ratio, phase_number, phase_abundance, phase_score, pvalue, PHAS_gene_annotation, marker
- Merged clusters respect the applied cutoffs: phase number 4, phase length 21, pvalue 0.001, cluster island 5
- When -dp y is set, results supported only by the phase-score method are excluded from the integrated output
- Clusters originally in genome and transcriptome coordinates appear in a single reconciled coordinate space in the integration outputs

## Limitations

- Version 1.0.1 is the only changelog entry available (25/02/2024); no discussion/changelog-derived limitations or reproducibility details are extractable.
- The integration module requires both phase module outputs (-io and -ia) and a GFF3 annotation; it cannot run from raw mapping files alone.
- Default behavior discards results supported only by the phase-score method (-dp default=y), which may remove phase-score-only loci unless explicitly disabled.

## Evidence

- [methods] Coordinate transform via bedtools: [evidence span withheld — anchor status: not_found]
- [readme] Workflow step 3: "3. PhasiRNA and PHAS loci result integration"
- [readme] Integration command line: "phasiHunter integration -io /home/user/test_osa/phase_o.txt -ia /home/user/test_osa/phase_a.txt -an /home/user/test_osa/oryza_sativa_gdna.gff3 -g y -o /home/user/test_osa/integration_o.txt"
- [abstract] Automatic integration claim: "PhasiHunter can identify phasiRNAs and their regulatory networks based on multiple reference sequences and the predicted results can be automatically integrated"
- [readme] Integration parameters: "-pn 4 -pl 21 -pv 0.001 -il 5"
