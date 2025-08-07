# DATASET CARD — Synthetic Domain Names (v1)

Created reproducibly in `notebooks/01_dataset_creation.ipynb` using `data/synth/v1/config.yaml`.
- Total: 1000 (blocked ~120; safe ~880)
- Splits: [0.7, 0.15, 0.15]
- Industries: coffee, family therapy, childcare, nonprofit, legal, fitness, wedding, pet care, fintech, hvac, education, grocery, gardening, co-parenting
- Safety: refusal cases per `docs/SAFETY_POLICY.md`
- IO: JSON-only { "status", "message?", "suggestions":[{"domain","confidence"}] }

Notes/limitations:
- Heuristic confidence (not calibrated).
- Wordbanks are simple; creativity bounded.
- This v1 is the baseline fine-tune set; later iterations may augment/adjust.
