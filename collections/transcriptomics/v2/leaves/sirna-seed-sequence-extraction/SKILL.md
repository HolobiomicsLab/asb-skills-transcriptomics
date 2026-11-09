---
name: sirna-seed-sequence-extraction
description: Use when you have a 23 bp, 5'→3' oriented siRNA guide sequence (e.g.,
  "UUAUAGAGCAAGAACACUGUUUU" targeting rat Ttr) and need to derive the mer7m8 seed
  to drive seed-match counting in SeedMatchR.
when_to_use_negative:
- You already have a pre-trimmed seed string and only need match counting — call SeedMatchR()
  directly with sequence/seed.name rather than re-deriving it.
- The query is a full off-target search space (bulges/wobbles) requiring full_search()
  views of the siRNA rather than a single seed definition.
- You are doing general k-mer motif scanning unrelated to siRNA seed classes (mer7/mer7m8)
  — get_seed() only extracts defined siRNA seed regions.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SeedMatchR
  role: Provides get_seed() to extract seed sequence information from the siRNA input
    sequence and SeedMatchR()/full_search() to consume the extracted seed for transcriptome-wide
    seed-match counting
  repo: https://github.com/tacazares/SeedMatchR
- name: DESeq2
  role: Downstream differential expression results frame that the extracted seed's
    per-gene match counts annotate
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae011
    title: 'SeedMatchR: identify off-target effects mediated by siRNA seed regions
      in RNA-seq experiments'
  source_content_hash: sha256:a18b103cf198fd9a919ad3aa535a30160ade60713246e1dcb2b041dd430e702b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tacazares/SeedMatchR
  tools:
  - AnnotationHub
  - DESeq2
  - R
  - SeedMatchR
  - ensembldb
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# sirna-seed-sequence-extraction

## Summary

Extracts the canonical siRNA seed region (e.g., mer7m8) from a full-length siRNA guide sequence using SeedMatchR's get_seed() logic, so that seed-mediated off-target seed matches can be counted against 3'UTR features in downstream analyses.

## When to use

Use when you have a 23 bp, 5'→3' oriented siRNA guide sequence (e.g., "UUAUAGAGCAAGAACACUGUUUU" targeting rat Ttr) and need to derive the mer7m8 seed to drive seed-match counting in SeedMatchR.

## When NOT to use

- You already have a pre-trimmed seed string and only need match counting — call SeedMatchR() directly with sequence/seed.name rather than re-deriving it.
- The query is a full off-target search space (bulges/wobbles) requiring full_search() views of the siRNA rather than a single seed definition.
- You are doing general k-mer motif scanning unrelated to siRNA seed classes (mer7/mer7m8) — get_seed() only extracts defined siRNA seed regions.

## Inputs

- siRNA guide sequence, 23 bp, 5'→3' oriented (e.g., "UUAUAGAGCAAGAACACUGUUUU")
- Seed definition name (e.g., "mer7m8") selecting which seed region to extract

## Outputs

- mer7m8 seed sequence string derived from positions 2–8 of the guide
- Seed string consumed by SeedMatchR()/full_search() for per-gene 3'UTR match counting

## How to apply

Provide the full siRNA guide sequence (23 bp, oriented 5'→3', RNA alphabet with U) to SeedMatchR's get_seed() together with a seed.name argument selecting the seed definition (here "mer7m8"), which designates a 7-mer starting at nucleotide 2 paired to position 8 of the guide. The function derives the seed sequence information from the siRNA input sequence rather than requiring a hand-trimmed substring, keeping the seed definition consistent with the package's downstream vcountPattern-based exact matching (max.mismatch = 0, no indels). The extracted seed string is then passed (or recomputed identically) to SeedMatchR()/full_search() so per-gene counts are computed over named 3'UTR feature sequences loaded via load_annotations (e.g., rnor6 with canonical = FALSE, min.feature.width = 8, longest.utr = TRUE). Rationale: mer7m8 is the seed class implicated in cumulative seed-mediated off-target differential expression that SeedMatchR is designed to detect.

## Related tools

- **SeedMatchR** (Provides get_seed() to extract seed sequence information from the siRNA input sequence and SeedMatchR()/full_search() to consume the extracted seed for transcriptome-wide seed-match counting) — https://github.com/tacazares/SeedMatchR
- **DESeq2** (Downstream differential expression results frame that the extracted seed's per-gene match counts annotate)

## Examples

```
guide.seq = "UUAUAGAGCAAGAACACUGUUUU"; seed = get_seed(guide.seq, seed.name = "mer7m8")
```

## Evaluation signals

- The extracted seed corresponds to the mer7m8 definition applied to the 23-bp guide (positions 2–8), not the full guide or a 3' trailer.
- Reimplementing seed counting with vcountPattern semantics (exact matching, max.mismatch = 0) on the extracted seed reproduces the mer7m8 column produced by the package's SeedMatchR() call on identical inputs.
- The annotated example yields Ttr (ENSRNOG00000016275) with mer7m8 = 1 while most genes have 0, consistent with the on-target seed match.
- Guide input must be 23 bp, 5'→3', RNA (U) — a DNA-alphabet or wrong-length input indicates a misapplied extraction.

## Limitations

- Seed extraction is tied to predefined seed names (e.g., mer7m8); other seed views of the siRNA require full_search() instead of a single-seed call.
- Exact-match counting downstream means bulges and wobbles in the target search space are not handled by the standard SeedMatchR() seed path (the forked repository's stated goal to add biological target bulges and wobbles).
- Rat references (rnor6/rnor7) restrict annotation filtering options (canonical = FALSE; no transcript.support filtering), which affects which 3'UTR sequences the seed is matched against.

## Evidence

- [methods] Seed extraction via get_seed: "You can extract the seed sequence information from the siRNA input sequence using the `get_seed()` function."
- [readme] Guide orientation and length: "The guide sequence of interest is 23 bp long and oriented 5’ -> 3’."
- [abstract] Seed-match annotation of DE results: "SeedMatchR is designed to extend current differential expression analysis tools, such as DESeq2, by annotating results with predicted seed matches."
- [readme] Downstream exact-match counting contract: "You can perform a seed match for a single seed using the `SeedMatchR()` function."
