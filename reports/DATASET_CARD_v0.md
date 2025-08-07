# DATASET CARD — Synthetic Domain Names (v0 smoke)

Created via `notebooks/01_dataset_creation.ipynb` to validate the pipeline before v1.
- Total: 50 (blocked ~6; safe ~44)
- Splits: [0.7, 0.15, 0.15]
- Industries: coffee, family therapy, childcare, nonprofit, legal, fitness, wedding, pet care, fintech, hvac, education, grocery, gardening, co-parenting
- Safety: refusal cases included; expected blocked schema in `docs/SAFETY_POLICY.md`
- IO format: JSON {"status","message?","suggestions":[{"domain","confidence"}]}

Notes/limitations:
- Heuristic confidence (not calibrated).
- Simple wordbanks; creativity limited in v0.
- v1 will scale to 1000 examples with same config/seed.
