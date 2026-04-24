# TEM-1 Pilot — 2026-04-22

## Headline result
Spearman ρ = 0.7302 [95% BCa CI 0.7149, 0.7445], N = 4996 single-residue variants, ESM2-650M masked-marginal scoring vs Stiffler et al. 2015 DMS (ampicillin 2500 µg/mL).

**H2 (pre-registered primary): supported.** Threshold was ρ ≥ 0.35 with CI excluding 0.

## Supporting metrics
- AUC (binary fit/unfit at median cutoff): 0.883
- MCC: 0.628
- Top-recall @ 10%: 0.289 (144/499)

## Sequence provenance
Stiffler et al. 2015 used the TEM-1A allele. UniProt P62593 canonical is TEM-1B. These differ at two residues:
- Position 59: Stiffler V, UniProt I
- Position 159: Stiffler A, UniProt V

Scoring was performed on Stiffler's exact sequence, reconstructed from ProteinGym's `mutated_sequence` column. Using the UniProt canonical sequence would introduce systematic error at these two positions.

## Data provenance
ProteinGym v1.3 substitutions, Zenodo record 14997691
Archive: DMS_ProteinGym_substitutions.zip
Assay file: BLAT_ECOLX_Stiffler_2015.csv (4,996 variants, 1,540,539 bytes)

## Model
esm2_t33_650M_UR50D (Lin et al., Science 2023)
Masked-marginal scoring per Meier et al., NeurIPS 2021

## Reproducibility
Seed 42 (numpy, torch, Python random)
Compute: Google Colab free tier, Tesla T4 GPU
ESM2 forward passes: 263 (one per residue), wall time ~37 seconds

## Notes
- Spearman ρ = 0.73 is at the high end of published ESM2-650M results on β-lactamase DMS
- Top-recall@10% appearing low alongside high Spearman is expected: ESM2 discriminates deleterious/neutral well but is less precise in the top decile. This is a documented behavior in the ProteinGym paper and consistent with our use case (tolerance quantification, not ranking)
