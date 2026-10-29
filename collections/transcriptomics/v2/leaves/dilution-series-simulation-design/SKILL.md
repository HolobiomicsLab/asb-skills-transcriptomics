---
name: dilution-series-simulation-design
description: Use when quantifying detection sensitivity of a mutational signature
  fitting algorithm (e.g. mmsig) for low-abundance signatures, by generating simulated
  mutation catalogs with known ground-truth signature admixtures.
when_to_use_negative:
- The analysis goal is fitting signatures to real patient tumor catalogs rather than
  benchmarking a fitting algorithm against known admixtures
- Targeted capture panels covering a few megabases are the input context, since simulated
  catalogs of 5000 mutations far exceed the low mutational burdens such panels yield
- A de novo signature extraction benchmark (e.g. NMF) is required rather than reference-based
  fitting sensitivity
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3676
- http://edamontology.org/topic_0092
tools:
- name: mmsig
  role: R package providing signature_ref (COSMIC v3.1 SBS profiles) and mm_96_classes
    used as the basis for building background and dilution-weighted probability profiles
  repo: https://github.com/evenrus/mmsig
- name: R
  role: Language in which the multinomial sampling, set.seed(1) reproducibility and
    Poisson-noise perturbation are implemented
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-021-01938-0
    title: 'mmsig: a fitting approach to accurately identify somatic mutational signatures
      in hematological malignancies'
  source_content_hash: sha256:c36d49dfc515ecdf598760339db889dd944322033dd6cbfae960a373d4e47d95
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/evenrus/mmsig
  tools:
  - R
  - mmsig
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3676
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# dilution-series-simulation-design

## Summary

Designs in-silico dilution series of mutational signatures to benchmark the sensitivity of signature-fitting algorithms: known target signatures (e.g. SBS2 at 1–10%, SBS-MM1 at 1–20%) are admixed at controlled fractions into an average background profile (here the median MM profile: SBS1 4.8%, SBS2 5.1%, SBS5 34.1%, SBS8 18.2%, SBS9 18.9%, SBS-MM1 18.8%) before multinomial sampling of fixed-size catalogs.

## When to use

Use when quantifying detection sensitivity of a mutational signature fitting algorithm (e.g. mmsig) for low-abundance signatures, by generating simulated mutation catalogs with known ground-truth signature admixtures.

## When NOT to use

- The analysis goal is fitting signatures to real patient tumor catalogs rather than benchmarking a fitting algorithm against known admixtures
- Targeted capture panels covering a few megabases are the input context, since simulated catalogs of 5000 mutations far exceed the low mutational burdens such panels yield
- A de novo signature extraction benchmark (e.g. NMF) is required rather than reference-based fitting sensitivity

## Inputs

- COSMIC v3.1 SBS reference signature profiles (mmsig signature_ref data object with sub/tri columns)
- mm_96_classes: the 96 trinucleotide class definitions (mmsig data object)
- Stated median signature contributions for the average MM genome (SBS1 4.8%, SBS2 5.1%, SBS5 34.1%, SBS8 18.2%, SBS9 18.9%, SBS-MM1 18.8%)

## Outputs

- simulated_catalogs.csv: dilution-series catalogs of 96-class mutation counts (100 simulations of 5000 mutations per dilution level) plus Poisson-noise variants
- Ground-truth dilution levels (SBS2 1–10%, SBS-MM1 1–20%) for downstream sensitivity/specificity evaluation

## How to apply

Build a background 96-trinucleotide-class probability profile from median signature contributions — for multiple myeloma, SBS1 (4.8%), SBS2 (5.1%), SBS5 (34.1%), SBS8 (18.2%), SBS9 (18.9%) and SBS-MM1 (18.8%) loaded from mmsig's signature_ref data object with sub/tri columns. Construct the dilution series by re-weighting the target signatures (SBS2 across 1–10%, SBS-MM1 across 1–20%) within the base profile, then draw multinomial random samples over the 96 classes to generate 100 simulated catalogs of 5000 mutations per dilution level, using set.seed(1) for reproducibility. Optionally generate Poisson-noise perturbed variants of each catalog to mimic count stochasticity, and write the catalogs to a CSV file. This design lets you fit signatures with mm_fit_signatures at each dilution level and measure the minimum abundance at which the target signature is still recovered.

## Related tools

- **mmsig** (R package providing signature_ref (COSMIC v3.1 SBS profiles) and mm_96_classes used as the basis for building background and dilution-weighted probability profiles) — https://github.com/evenrus/mmsig
- **R** (Language in which the multinomial sampling, set.seed(1) reproducibility and Poisson-noise perturbation are implemented)

## Examples

```
library(mmsig); data(signature_ref); set.seed(1); sim <- rmultinom(100, 5000, prob = weighted_sig_profile)  # then re-weight SBS2 (1–10%) and SBS-MM1 (1–20%) per dilution level
```

## Evaluation signals

- Each simulated catalog sums to exactly 5000 mutations
- Realized SBS2 fractions fall within 1–10% and realized SBS-MM1 fractions within 1–20% of the specified dilution levels within multinomial sampling tolerance
- Re-running with set.seed(1) reproduces identical catalogs
- Poisson-noise variants deviate from their parent catalogs only by count stochasticity while preserving the same signature admixture

## Limitations

- Simulated catalogs use an 'average' MM genome background, so results may not reflect heterogeneity of individual tumor profiles
- Accuracy is only meaningful for catalogs above a size floor — fitting accuracy is optimal for catalogs of more than 500 mutations
- Signatures occupying largely the same feature space cause inter-bleeding, which limits how low a dilution level can be distinguished

## Evidence

- [methods] Dilution series construction over 96 trinucleotide classes: [evidence span withheld — anchor status: not_found]
- [methods] Sampling and Poisson-noise design: [evidence span withheld — anchor status: not_found]
- [methods] Validation of simulated catalogs: [evidence span withheld — anchor status: not_found]
- [discussion] Mutational signature analysis is subject to a degree of uncertainty: "Because different mutational processes occupy largely the same feature space, mutational signature analysis is subject to a degree of uncertainty."
- [discussion] Catalog size floor for fitting accuracy: "The accuracy of mutational signature fitting, in general, is optimal when applied to catalogs of more than 500 mutations, which can be consistently obtained by WGS"
- [readme] mmsig installable from GitHub with data objects used to build profiles: "devtools::install_github("evenrus/mmsig")"
