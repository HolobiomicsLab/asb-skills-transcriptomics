---
name: internal-priming-artifact-filtering
description: Use when building or curating a candidate cleavage-site catalog from
  3'-tag scRNA-seq data (e.g. Microwell-seq) where genomic A-rich stretches downstream
  of a candidate site could indicate internal priming rather than a genuine polyadenylation
  event.
when_to_use_negative:
- Input reads are full-length or otherwise span the actual mRNA 3' end cleavage site,
  so de novo 3' end mapping does not rely on local sequence context (the paper notes
  most 10x reads do NOT span CS, which is the motivating condition)
- Candidate sites are already annotated as internal priming artifacts in an existing
  resource such as PolyASite, making re-scoring redundant
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: cleanUpdTSeq
  role: Scores candidate cleavage sites for internal priming using the local upstream/downstream
    nucleotide context so that high-scoring artifacts can be removed
- name: PolyASite 2.0
  role: Independent poly(A) cluster database used to tag/validate candidate sites
    alongside (and after) internal-priming filtering
- name: scUTRquant
  role: Downstream pipeline that consumes the internal-priming-filtered CS catalog
    for 3'UTR isoform quantification
  repo: github.com/Mayrlab/scUTRquant
- name: Mayrlab/tmuris-peaks
  role: Analysis of 10X Chromium peak widths at cleavage sites curated for minimal
    internal priming upstream of the cleavage site
  repo: Mayrlab/tmuris-peaks
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:a7267f3604ae37dc1caad3288eb4afd9b33fbf8b45d6a096f8959e1ddd98312c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Mayrlab/tmuris-peaks
  - PolyASite 2.0
  - cleanUpdTSeq
  - scUTRquant
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Internal priming artifact filtering

## Summary

Removes mis-called mRNA 3' end cleavage sites caused by internal oligo-dT priming during scRNA-seq library construction, by scoring local nucleotide context around candidate sites (e.g. with cleanUpdTSeq) and dropping high-scoring artifacts from the Microwell-seq-derived CS atlas.

## When to use

Use when building or curating a candidate cleavage-site catalog from 3'-tag scRNA-seq data (e.g. Microwell-seq) where genomic A-rich stretches downstream of a candidate site could indicate internal priming rather than a genuine polyadenylation event.

## When NOT to use

- Input reads are full-length or otherwise span the actual mRNA 3' end cleavage site, so de novo 3' end mapping does not rely on local sequence context (the paper notes most 10x reads do NOT span CS, which is the motivating condition)
- Candidate sites are already annotated as internal priming artifacts in an existing resource such as PolyASite, making re-scoring redundant

## Inputs

- Candidate mRNA 3' end cleavage site coordinates (BED/GTF, e.g. MWS-derived CS intersected with GENCODE-overlapping or PolyASite-supported sites)
- Reference genome FASTA for extracting upstream/downstream nucleotide context around each site
- GENCODE GTF annotations (vM25 mouse / v39 human per workflow) and PolyASite 2.0 poly(A) cluster coordinates

## Outputs

- Filtered set of candidate cleavage sites with high-scoring internal-priming artifacts removed
- Final CS catalog files (mws_cs_annotation.gtf, mws_cs_annotation.bed) with categories (GENCODE-overlapping, PolyASite-supported, novel)

## How to apply

For each candidate cleavage site (e.g. Microwell-seq-derived CS intersected with GENCODE 3' end windows or PolyASite 2.0 clusters), extract the local upstream/downstream nucleotide genomic sequence surrounding the site. Score this context for internal priming with cleanUpdTSeq, which models the sequence features typical of mis-priming (A-rich regions downstream of the site). Remove sites whose scores classify them as high-risk internal-priming artifacts before exporting the final catalog (mws_cs_annotation.gtf/.bed). The rationale is that most 10x/MWS reads do not span the actual cleavage site, so A-rich downstream sequence can anchor the RT primer internally and falsely appear as a 3' end. The paper's companion tmuris-peaks analysis additionally curates cleavage sites for minimal internal priming upstream of the cleavage site when measuring 10X peak widths.

## Related tools

- **cleanUpdTSeq** (Scores candidate cleavage sites for internal priming using the local upstream/downstream nucleotide context so that high-scoring artifacts can be removed)
- **PolyASite 2.0** (Independent poly(A) cluster database used to tag/validate candidate sites alongside (and after) internal-priming filtering)
- **scUTRquant** (Downstream pipeline that consumes the internal-priming-filtered CS catalog for 3'UTR isoform quantification) — github.com/Mayrlab/scUTRquant
- **Mayrlab/tmuris-peaks** (Analysis of 10X Chromium peak widths at cleavage sites curated for minimal internal priming upstream of the cleavage site) — Mayrlab/tmuris-peaks

## Evaluation signals

- A large fraction of high-quality candidate sites should survive filtering and overlap GENCODE annotations; the paper expects the merged catalog to extend GENCODE CS annotations by ~40%, not to eliminate most sites
- No surviving high-confidence site should exhibit the strong downstream A-rich sequence signature characteristic of internal priming (cleanUpdTSeq scores below the artifact threshold)
- Filtered sites should retain support from independent evidence such as PolyASite 2.0 clusters or GENCODE-annotated 3' ends
- The final catalog should export cleanly as valid GTF/BED (mws_cs_annotation.gtf / .bed) and, when used in scUTRquant, produce consistent 3'UTR isoform quantification without elevated internal-priming artifacts
- Peak-width analyses at curated sites (cf. tmuris-peaks) should not show systematic shifts toward longer peaks that would indicate retained mis-primed sites

## Limitations

- Most reads from 10x Genomics / Microwell-seq data do not span the cleavage site, so filtering relies on local sequence features and cannot perfectly distinguish genuine sites from artifacts
- Only internal priming artifacts are removed; other systematic biases in 3'-tag scRNA-seq peak positions remain uncorrected
- Filtering depends on reference sequence context quality; candidates in poorly assembled or A-rich genomic regions may be incorrectly discarded

## Evidence

- [discussion] QC rationale for filtering MWS-derived cleavage sites: "We performed rigorous quality control and filtering for internal priming and show that MWS CS are of high quality"
- [intro] 10x reads do not span cleavage sites, motivating context-based artifact detection: "most reads from 10x Genomics data do not span mRNA 3′ end cleavage sites (CS)"
- [readme] Cleavage sites curated for minimal internal priming are used in downstream peak-width validation: [evidence span withheld — anchor status: external_doc]
- [discussion] Expected outcome: filtered catalog extends GENCODE CS annotations: "extending current GENCODE CS annotations by 40%"
