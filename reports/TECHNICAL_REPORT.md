### v0 — Dataset creation & initial results

**Methodology (reproducible):**
- Synthetic, programmatic generation with fixed seed **42** (`data/synth/v1/config.yaml`).
- Each example pairs:
  - **Input:** one prompt instructing the model to return **JSON only** (business description, preferred TLDs, constraints: `allow_hyphens`, `allow_numbers`, `prefer_puns`).
  - **Output:** either  
    - `{"status":"success","suggestions":[{"domain","confidence"}]}`, or  
    - `{"status":"blocked","message":"Request contains inappropriate content","suggestions":[]}` for unsafe requests.
- ~**12%** of prompts are intentionally unsafe (adult, hate/violence, illegal, weapons-to-minors, doxxing, CSE) per `docs/SAFETY_POLICY.md`.

**Artifacts:**
- Data (JSONL): `data/synth/v0/{train,val,test}.jsonl`
- Dataset card: `reports/DATASET_CARD_v0.md`
- Safety policy: `docs/SAFETY_POLICY.md`
- Schema: `docs/DATA_SCHEMA.md`

**Counts (v0):**
- **Total:** 50  
- **Approx overall safe vs blocked:** ~44 safe, ~6 blocked (target ≈12% blocked)  
- **Splits (config 70/15/15):** Train **35**, Val **7**, Test **8**

**Distributions (train split sanity check):**
by_industry:
blocked 6
childcare 5
pet_care 4
family_therapy 4
fitness 3
grocery 3
education 2
gardening 2
fintech 2
co_parenting 2

by_complexity:
L1 18
L2 9
N/A 6
L3 2

safety_counts:
safe 29
child_exploitation 2
hate_violence 2
adult_explicit 1
doxxing 1

**Example (train):**
- **Sample input (truncated):** family-friendly gardening in Austin with `.com/.co/.org/.family/.ai`, `allow_hyphens=False`, `allow_numbers=False`, `prefer_puns=False`.
- **Sample output:** `status=success`, suggestions like `sprout.org`, `bloomgarden.com`, `grow.org` with heuristic confidences.

**Notes & limitations:**
- Confidence is a simple heuristic (length + TLD priors + penalties for constraint clashes); not calibrated.
- Wordbanks are compact; creativity bounded in v0—fine for a pipeline smoke test.
- Blocked examples happened to fall into the **train** slice in this v0 sample; overall target remains ≈12%.

**Takeaway:** v0 validates the generation pipeline, schema, and refusal behavior. Next: generate **v1 (n=1000)** for baseline training and formal evaluations.