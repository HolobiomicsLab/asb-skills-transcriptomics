---
name: f1-cross-parental-origin-read-tagging
description: Use when processing scRNA-Seq reads from a CAST×B6 F1 cross where reads
  must be assigned to maternal vs. paternal transcript copies, e.g., before EMASE-based
  multi-read allocation or unique-read allele counting.
when_to_use_negative:
- Input reads are from a non-F1 or non-mouse strain cross where CAST/B6-specific variants
  (Sanger REL-1505) and a two-parent transcriptome are not applicable
- Data are UMI counts already assigned per allele (e.g., Larsson et al. 224-cell fibroblast
  UMI counts) — alignment/tagging has already been done upstream
- The downstream analysis only needs unique-read counts and discarding multi-reads
  is acceptable — though the article warns this raises variability and sampling zeros
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: g2gtools
  role: Incorporates known SNPs and short indels (Sanger REL-1505) into the GRCm38
    reference genome to reconstruct the CAST genome sequence
  repo: http://churchill-lab.github.io/g2gtools/
- name: bowtie v1.0.0
  role: Builds the diploid transcriptome index and aligns single-end cell reads with
    '-v 3 --all --best --strata' to produce parentally-tagged allele-specific alignments
- name: EMASE
  role: Consumes the alignments downstream to resolve retained multi-reads via EM-based
    weighted allocation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:321142e7738888d83937961e1637d6b6f52b756467ec43f07ced4db169975c03
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMASE
  - bowtie
  - bowtie v1.0.0
  - g2gtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# f1-cross-parental-origin-read-tagging

## Summary

Constructs a diploid (CAST/B6) transcriptome so that single-cell RNA-Seq reads can be aligned to parental-origin-specific sequences and emitted as allele-specific alignments tagged by maternal/paternal copy, enabling ASE analysis in F1 hybrid single cells.

## When to use

Use when processing scRNA-Seq reads from a CAST×B6 F1 cross where reads must be assigned to maternal vs. paternal transcript copies, e.g., before EMASE-based multi-read allocation or unique-read allele counting.

## When NOT to use

- Input reads are from a non-F1 or non-mouse strain cross where CAST/B6-specific variants (Sanger REL-1505) and a two-parent transcriptome are not applicable
- Data are UMI counts already assigned per allele (e.g., Larsson et al. 224-cell fibroblast UMI counts) — alignment/tagging has already been done upstream
- The downstream analysis only needs unique-read counts and discarding multi-reads is acceptable — though the article warns this raises variability and sampling zeros

## Inputs

- Single-end FASTQ reads from Deng et al. pre-implantation embryo cells (CAST×B6 F1 hybrid, 286 cells) or other F1 hybrid scRNA-Seq libraries
- GRCm38 mouse reference genome sequence
- CAST-specific SNP and short indel variant set (Sanger REL-1505)
- Ensembl Release 78 transcript annotations for B6 (and CAST) transcripts

## Outputs

- Diploid transcriptome FASTA with maternal/paternal copy labels per transcript
- bowtie v1.0.0 index of the diploid transcriptome
- Per-cell allele-specific alignment files with reads tagged by parental origin (retaining all best-scoring alignments in the top stratum, up to 3 mismatches)

## How to apply

Reconstruct the CAST/EiJ genome by incorporating known SNPs and short indels (Sanger REL-1505) into the GRCm38 reference using g2gtools, then build the CAST-specific transcriptome and combine it with the B6 transcriptome (Ensembl Release 78) into a single diploid transcriptome FASTA carrying maternal/paternal copy labels. Build a bowtie (v1.0.0) index over this diploid FASTA and align single-end Deng et al. embryo reads with '-v 3' (up to three mismatches) plus '--all --best --strata' so both genomic and allelic multi-reads are retained rather than discarded. The rationale is that discarding multi-mapping reads increases variability in allelic proportions and creates sampling zeros, so retaining and later resolving them (e.g., via EMASE EM weighted allocation) preserves information. Emit the parentally-tagged alignment files for downstream allele-specific counting.

## Related tools

- **g2gtools** (Incorporates known SNPs and short indels (Sanger REL-1505) into the GRCm38 reference genome to reconstruct the CAST genome sequence) — http://churchill-lab.github.io/g2gtools/
- **bowtie v1.0.0** (Builds the diploid transcriptome index and aligns single-end cell reads with '-v 3 --all --best --strata' to produce parentally-tagged allele-specific alignments)
- **EMASE** (Consumes the alignments downstream to resolve retained multi-reads via EM-based weighted allocation)

## Examples

```
bowtie -v 3 --all --best --strata diploid_transcriptome cell_reads.fastq > cell.as.alignments.sam
```

## Evaluation signals

- The bowtie index over the diploid transcriptome builds successfully without errors
- Alignment output files exist for each Deng et al. cell under the '-v 3 --all --best --strata' parameterization
- Reads in the output alignments carry parental-origin (maternal/paternal) tags consistent with the transcriptome copy labels
- Multi-mapping reads (both genomic and allelic) are present in the alignments rather than filtered, since --all --best --strata retains them
- Each alignment maps to a transcript labeled as CAST- or B6-derived in the combined diploid FASTA

## Limitations

- Only known SNPs and short indels from Sanger REL-1505 distinguish the CAST genome from GRCm38; unannotated variants (e.g., larger structural differences) are not modeled
- Versioning gaps: no software version is stated for g2gtools or EMASE, which limits exact reproducibility of the tagging step
- Multi-reads are retained but not resolved at this stage — their allele assignment depends on a downstream EM allocation step (EMASE)

## Evidence

- [discussion] Reconstruct CAST genome by incorporating SNPs and indels into reference genome with g2gtools: "We reconstructed the CAST genome by incorporating known SNPs and short indels (Sanger REL-1505) into the reference mouse genome sequence"
- [discussion] Alignment to phase-known diploid transcriptome with bowtie parameters: [evidence span withheld — anchor status: not_found]
- [abstract] Rationale for retaining multi-reads: "discarding multi-mapping reads leads to higher variability in estimates of allelic proportions, an increased frequency of sampling zeros"
- [discussion] Downstream EMASE weighted allocation of retained multi-reads: "multi-reads are resolved by computing a weighted allocation based on the estimated probability of each alignment"
- [discussion] Bowtie version and diploid index construction: "We constructed a bowtie (v1.0.0) index to represent the diploid transcriptome"
