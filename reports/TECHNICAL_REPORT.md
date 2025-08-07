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


### v1 (baseline training set) — Dataset stats & notes

**Scale-up (reproducible):**
- Same synthetic pipeline, fixed seed **42** (`data/synth/v1/config.yaml`).
- Safe/blocked mix targets **~12% blocked** per `docs/SAFETY_POLICY.md`.  
  *Observed on train:* 84 / 700 = **12.0%** blocked (matches target).

**Counts (v1):**
- **Total:** 1000  
- **Splits (70/15/15):** Train **700**, Val **150**, Test **150**

**Distributions (train split sanity check):**
by_industry:
blocked 84
grocery 49
gardening 48
coffee_shop 48
childcare 48
education 48
fitness 46
pet_care 46
family_therapy 46
nonprofit 44

by_complexity:
L1 310
L2 210
L3 96
N/A 84

safety_counts:
safe 616
child_exploitation 20
adult_explicit 19
doxxing 16
weapons_minor 15
hate_violence 8
illegal 6

**Example (train):**
- **Sample input (truncated):** premium education in San Diego; TLDs `.com/.co/.org`; constraints: `allow_hyphens=False`, `allow_numbers=False`, `prefer_puns=False`.
- **Sample output:** `status=success` with names like `academyclass.com`, `classstudy.org`, `learnacademy.org` *(plus a couple borderline blends like `ledemy.com`, `tuarn.co`)*.

**Notes & limitations:**
- **Quality blips:** A few blends (e.g., `ledemy`, `tuarn`) are pronounceable but feel off-brand. This is acceptable for a baseline but flagged for a **v1.1 data pass** (add extra filters/wordbanks to prune awkward blends and near-duplicates).
- Confidence remains a heuristic (length + TLD priors + small penalties), used as a feature for analysis—not a calibrated metric.
- “blocked” appears as an `industry` label for refusal rows by design; we’ll keep it, but we’ll also report safety separately in evals.

**Decision:** v1 is good enough to train the **baseline** now. We’ll plan a **v1.1** generator tweak (stronger blend filter + duplicate suppression) during the first improvement cycle if baseline evals confirm it’s needed.
