---
name: cosmic-signature-reference-curation
description: Use when fitting reference signatures to WGS catalogs from hematological
  malignancies or other cancers where the default COSMIC signature set contains processes
  that are biologically implausible for the cohort (e.g., platinum exposure absent)
  or where genome-wide vs exome context matters.
when_to_use_negative:
- The cohort includes platinum-exposed patients — in that case SBS35 must be kept
  in the reference (as in the two platinum-exposed t-AML samples) rather than removed.
- De novo signature extraction is the goal (e.g., NMF on a large cohort) rather than
  fitting samples against a curated reference set.
- The mutation catalog comes from targeted capture panels covering only a few megabases,
  where mutational burden is too low for meaningful reference fitting regardless of
  reference curation.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: mmsig
  role: Provides the signature_ref COSMIC v3.1 reference dataset, mm_96_classes channel
    definitions, and mm_fit_signatures with force_include to fit the curated reference
  repo: https://github.com/evenrus/mmsig
- name: deconstructSigs
  role: Alternative fitting algorithm used to benchmark the curated v2 vs v3.1 references
    and compare SBS35 false-positive rates with the 6% exposure cut-off
- name: MutationalPatterns
  role: Third fitting algorithm included in the reference-version reconstruction-accuracy
    comparison
- name: R
  role: Environment in which the reference is subset, normalized, and passed to the
    fitting functions
- name: devtools
  role: Used to install the mmsig development version that ships signature_ref
  repo: https://github.com/evenrus/mmsig
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_005
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s42003-021-01938-0
    title: 'mmsig: a fitting approach to accurately identify somatic mutational signatures
      in hematological malignancies'
  source_content_hash: sha256:9e580e58b41bd1970922974e8265c93f984e989e9f16c3452233204d41cd1179
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/evenrus/mmsig
  tools:
  - BSgenome
  - BSgenome.Hsapiens.UCSC.hg19
  - MutationalPatterns
  - R
  - deconstructSigs
  - devtools
  - ggplot2
  - mmsig
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cosmic-signature-reference-curation

## Summary

Curates the COSMIC mutational signature reference matrix before fitting by selecting the appropriate reference version (COSMIC v2 vs v3.1), removing context-inappropriate signatures (canonical AID SBS84 for genome-wide analysis, platinum SBS35 when patients are not platinum exposed), and optionally supplementing with cancer-specific signatures such as SBS-MM1.

## When to use

Use when fitting reference signatures to WGS catalogs from hematological malignancies or other cancers where the default COSMIC signature set contains processes that are biologically implausible for the cohort (e.g., platinum exposure absent) or where genome-wide vs exome context matters.

## When NOT to use

- The cohort includes platinum-exposed patients — in that case SBS35 must be kept in the reference (as in the two platinum-exposed t-AML samples) rather than removed.
- De novo signature extraction is the goal (e.g., NMF on a large cohort) rather than fitting samples against a curated reference set.
- The mutation catalog comes from targeted capture panels covering only a few megabases, where mutational burden is too low for meaningful reference fitting regardless of reference curation.

## Inputs

- mmsig signature_ref data frame (COSMIC v3.1) with columns sub, tri, and per-signature 96-channel probabilities
- COSMIC v2 signature matrix supplemented with SBS-MM1 (for version-comparison runs)
- Cohort context metadata: exposure status (e.g., platinum-naive vs platinum-exposed t-AML) and sequencing assay type (WGS vs WES)

## Outputs

- Curated sig_ref data frame with sub, tri columns and only cohort-appropriate SBS signature columns (e.g., SBS1, SBS2, SBS5, SBS8, SBS9, SBS13, SBS18, SBS-MM1)
- Per-sample exposure comparison tables when contrasting reference versions (v2 vs v3.1)
- Reconstruction-accuracy comparison across fitting algorithms for each reference version

## How to apply

Load the signature reference from mmsig via data(signature_ref) (COSMIC v3.1) or build a COSMIC v2 reference supplemented with SBS-MM1 for older-version comparisons. Subset the matrix to cohort-appropriate signatures: remove canonical AID (SBS84) because it is context-inappropriate for genome-wide analysis, and remove the platinum signature SBS35 when patients are not platinum exposed — for multiple myeloma the reference is trimmed to SBS1, SBS2, SBS5, SBS8, SBS9, SBS13, SBS18, and SBS-MM1. The reference must retain 'sub' and 'tri' context columns plus one normalized (sum-to-1) 96-channel probability column per signature. Prefer COSMIC v3.1 because its signatures are more clearly defined; the v3.1 reference gave higher reconstruction accuracy for mmsig, deconstructSigs, and MutationalPatterns and shifted estimates (higher SBS5, lower SBS1/SBS2/SBS8/SBS9 relative to v2). During fitting, pair the curated reference with force_include for ubiquitous signatures (SBS1, SBS5) so the error-suppression procedure cannot drop them. Note that removing a flat signature risks its mutations being re-assigned to a similar signature, which is why presence/absence is later judged by non-zero 95% bootstrap CIs rather than by reference membership alone.

## Related tools

- **mmsig** (Provides the signature_ref COSMIC v3.1 reference dataset, mm_96_classes channel definitions, and mm_fit_signatures with force_include to fit the curated reference) — https://github.com/evenrus/mmsig
- **deconstructSigs** (Alternative fitting algorithm used to benchmark the curated v2 vs v3.1 references and compare SBS35 false-positive rates with the 6% exposure cut-off)
- **MutationalPatterns** (Third fitting algorithm included in the reference-version reconstruction-accuracy comparison)
- **R** (Environment in which the reference is subset, normalized, and passed to the fitting functions)
- **devtools** (Used to install the mmsig development version that ships signature_ref) — https://github.com/evenrus/mmsig

## Examples

```
data(signature_ref); sig_ref <- signature_ref[c("sub","tri","SBS1","SBS2","SBS5","SBS8","SBS9","SBS13","SBS18","SBS-MM1")]
```

## Evaluation signals

- The curated reference retains exactly the intended signature columns (e.g., the eight hematological-malignancy signatures SBS1, SBS2, SBS5, SBS8, SBS9, SBS13, SBS18, SBS-MM1) plus sub/tri, with SBS84 and SBS35 absent for non-platinum genome-wide cohorts.
- Each signature column is a normalized 96-channel probability distribution over the mm_96_classes trinucleotide contexts.
- Fitting with the curated v3.1 reference yields higher reconstruction accuracy than v2 for mmsig, deconstructSigs, and MutationalPatterns, with the reported directionality: higher SBS1/SBS2/SBS8/SBS9 estimates under v2 and higher SBS5 under v3.1.
- SBS35 is called in the 49 de novo AML samples at only ~21% (mmsig) vs ~70% (deconstructSigs) when SBS35 is excluded from the reference for platinum-naive patients, and both platinum-exposed t-AML patients retain SBS35 with non-zero 95% CI and significant strand bias in C[C>A]C and C[C>T]C contexts.
- force_include=c('SBS1','SBS5') results in non-zero estimates for these ubiquitous signatures in every sample.

## Limitations

- Canonical AID (SBS84) is context-inappropriate for genome-wide analysis and must be removed, while flat signatures may be falsely removed by mmsig's cosine-similarity error suppression because all mutations can be re-classified to another signature with relatively low cosine-similarity loss.
- Reference-version choice materially changes estimates: COSMIC v2 yields higher SBS1, SBS2, SBS8, SBS9 and lower SBS5 than v3.1, so results are not directly comparable across reference versions.
- Whole-exome sequencing caveat: the expected signature profile in coding and non-coding regions can differ, affecting which reference signatures are appropriate.

## Evidence

- [readme] Remove context-inappropriate signatures from the COSMIC reference before fitting: "# remove canonical AID (SBS84) for genome-wide analysis
# remove the platinum signature (SBS35) because the patients are not platinum exposed"
- [abstract] COSMIC v3.1 signatures are more clearly defined, improving fitting: "This is particularly important in the most recent mutational signature reference (COSMIC v3.1) where each signature is more clearly defined."
- [methods] Reference setup is an explicit workflow step loading signature_ref: "## setting up the mutational signature reference"
- [methods] Ubiquitous signatures must be force-included during fitting of the curated reference: "force_include = c("SBS1", "SBS5")"
- [discussion] Flat signatures may be falsely removed by the error-suppression procedure: [evidence span withheld — anchor status: not_found]
