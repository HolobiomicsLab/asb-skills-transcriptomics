---
name: genome-sequence-reconstruction-with-variant-masking
description: Use when performing allele-specific expression (ASE) analysis in an F1
  hybrid cross (e.g., CAST×B6) and reads must be assigned to parental origin via a
  strain-specific or diploid reference rather than a single generic reference genome.
when_to_use_negative:
- The analysis discards multi-mapping reads (unique-reads counting); even so a diploid
  reference may still be needed, but the '--all --best --strata' multi-read retention
  strategy is wasted effort
- The cross is not between known, well-annotated strains — without a reliable SNP/indel
  variant set for the parental strain, reconstruction cannot be performed
- The data are bulk RNA-Seq or already provide allele-specific alignments from another
  pipeline
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: g2gtools
  role: Incorporates known SNPs and short indels (Sanger REL-1505) into the GRCm38
    reference to reconstruct the CAST genome
  repo: http://churchill-lab.github.io/g2gtools/
- name: bowtie v1.0.0
  role: Indexes the diploid transcriptome and aligns single-end reads with '-v 3 --all
    --best --strata' to obtain allele-specific alignments
- name: scBASE
  role: Downstream consumer of the allele-specific alignments for ASE counting, classification,
    and partial pooling
  repo: https://github.com/kbchoi-jax/scbase
- name: EMASE
  role: EM-based weighted allocation of multi-reads retained by the all-best-strata
    alignment strategy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:e81fe940bb07410eef5d1c8047f5d86f34f72448adf4862f32134c3f7fb5e425
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMASE
  - bowtie
  - bowtie v1.0.0
  - g2gtools
  - scBASE
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# Genome sequence reconstruction with variant masking

## Summary

Reconstructs a strain-specific (CAST/EiJ) genome by incorporating known SNPs and short indels (Sanger REL-1505) into the GRCm38 mouse reference using g2gtools, enabling construction of a phase-known diploid transcriptome for allele-specific read alignment.

## When to use

Use when performing allele-specific expression (ASE) analysis in an F1 hybrid cross (e.g., CAST×B6) and reads must be assigned to parental origin via a strain-specific or diploid reference rather than a single generic reference genome.

## When NOT to use

- The analysis discards multi-mapping reads (unique-reads counting); even so a diploid reference may still be needed, but the '--all --best --strata' multi-read retention strategy is wasted effort
- The cross is not between known, well-annotated strains — without a reliable SNP/indel variant set for the parental strain, reconstruction cannot be performed
- The data are bulk RNA-Seq or already provide allele-specific alignments from another pipeline

## Inputs

- GRCm38 reference mouse genome sequence (FASTA)
- Known SNPs and short indels for CAST (Sanger REL-1505 variant set)
- Gene annotations, Ensembl Release 78 (GTF/BED)
- Single-end scRNA-Seq reads from F1 hybrid cells (e.g., Deng et al. CAST×B6 pre-implantation embryos, 286 cells)

## Outputs

- Reconstructed CAST/EiJ genome sequence
- Diploid transcriptome FASTA with maternal/paternal copy labels
- bowtie (v1.0.0) index of the diploid transcriptome
- Allele-specific alignment files with reads tagged by parental origin

## How to apply

Start from the GRCm38 reference genome and, using g2gtools, incorporate known CAST SNPs and short indels from the Sanger REL-1505 variant call set to reconstruct the CAST/EiJ genome sequence. From this reconstructed genome build the CAST-specific transcriptome (Ensembl Release 78 annotations) and combine it with the B6 transcriptome into a single diploid transcriptome FASTA carrying maternal/paternal copy labels. Construct a bowtie (v1.0.0) index over this diploid transcriptome, then align single-end scRNA-Seq reads (e.g., Deng et al. embryo cells) with '-v 3 --all --best --strata' so up to three mismatches are allowed and genomic and allelic multi-reads are retained for later EM-based weighted allocation (EMASE) rather than being discarded. Correct application is judged by whether variant masking faithfully converts the reference to the strain haplotype and whether the alignment output preserves allele tags for downstream counting.

## Related tools

- **g2gtools** (Incorporates known SNPs and short indels (Sanger REL-1505) into the GRCm38 reference to reconstruct the CAST genome) — http://churchill-lab.github.io/g2gtools/
- **bowtie v1.0.0** (Indexes the diploid transcriptome and aligns single-end reads with '-v 3 --all --best --strata' to obtain allele-specific alignments)
- **scBASE** (Downstream consumer of the allele-specific alignments for ASE counting, classification, and partial pooling) — https://github.com/kbchoi-jax/scbase
- **EMASE** (EM-based weighted allocation of multi-reads retained by the all-best-strata alignment strategy)

## Examples

```
bowtie -v 3 --all --best --strata diploid_transcriptome reads.fastq > allele_specific_alignments.sam
```

## Evaluation signals

- The bowtie index builds successfully over the combined diploid transcriptome FASTA
- Alignment output files exist for the Deng et al. cells and contain reads aligned under the '-v 3 --all --best --strata' parameterization
- Alignments carry parental-origin (maternal/paternal) allele tags suitable for downstream allele-specific counting
- Multi-reads (genomic and allelic) are present in the output rather than silently discarded, since they feed the EMASE EM allocation

## Limitations

- Variant set completeness limits reconstruction fidelity; only known SNPs and short indels (Sanger REL-1505) are incorporated, so structural variants and untyped variants are missed
- No software version stated for g2gtools, hindering exact reproduction of the reconstruction step
- Multi-read retention inflates ambiguous alignments that must be resolved downstream (EMASE EM); discarding them instead raises variability in allelic proportion estimates and spurious monoallelic calls

## Evidence

- [discussion] We reconstructed the CAST genome by incorporating known SNPs and short indels (Sanger REL-1505) into the reference mouse genome sequence: "We reconstructed the CAST genome by incorporating known SNPs and short indels (Sanger REL-1505) into the reference mouse genome sequence"
- [discussion] We aligned reads using bowtie with parameters '–all', '–best', and '–strata', allowing for three mismatches ('-v 3'): [evidence span withheld — anchor status: not_found]
- [discussion] We use an EM algorithm implemented in EMASE software for this step: [evidence span withheld — anchor status: not_found]
- [discussion] using g2gtools (http://churchill-lab.github.io/g2gtools/): [evidence span withheld — anchor status: not_found]
- [readme] Documentation: https://churchill-lab.github.io/scBASE/: "Allele-specific expression (ASE) in single-cell resolution can reveal stochastic and dynamic features of gene expression in greater detail."
