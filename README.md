# Domain Name LLM — Take-Home (In Tandem)

A **reproducible** pipeline to:
- create a **synthetic dataset** for safe domain name suggestions,
- **fine-tune** an open-source LLM (baseline + improved iteration),
- evaluate with an **LLM-as-a-Judge**,
- perform **edge-case discovery** and **safety** testing.

> Focused on evaluation rigor and iterative improvement, per the brief.

---

## Project constraints & notes

- **API (optional) not implemented**: I did not ship the API endpoint **due to local hardware limitations**. I prioritized the dataset → training → evaluation pipeline, as recommended. The README and notebooks document how an API would integrate (guardrails → generation → JSON shim) if deployed later.
- **Improved model training used a remote GPU**: The improved model (Mistral QLoRA) was trained on **Google Colab** with a **T4** GPU. The baseline model (TinyLlama LoRA) trains and runs on CPU.

---

## Quickstart

### 0) System requirements
- Python 3.10+
- CPU is fine for **baseline LoRA (TinyLlama)**  
- For the **improved model (Mistral QLoRA)**, use a GPU (e.g., **Colab T4/A100**)

### 1) Setup (Unix/macOS)
```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
# (optional, for eval) set your OpenAI key
echo "OPENAI_API_KEY=YOUR_KEY_HERE" > .env

##1b) Setup (Windows PowerShell)
```bash
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
# (optional, for eval) create .env with your key
"OPENAI_API_KEY=YOUR_KEY_HERE" | Out-File -Encoding ascii .env

### **2\) Run the pipeline (notebooks)**

1. **Dataset** → open `notebooks/01_dataset_creation.ipynb` and run all cells.

   * Outputs: `data/synth/v1/{train,val,test}.jsonl` (seeded; see `config.yaml`).

2. **Baseline fine-tune (LoRA / CPU OK)** → run `notebooks/02_baseline_finetune.ipynb`.

   * Outputs: `checkpoints/baseline-tinyllama-v1/adapter/` and training stats.

3. **Improved model (QLoRA / Colab GPU)** → open `notebooks/02b_train_mistral_qlora_colab.ipynb` in **Google Colab** (Runtime → T4/A100), run all.

   * Outputs (saved/exported): `eval/colab_runs/mistral-qlora-v1/...`

4. **Evaluation (LLM-as-a-Judge)** → run `notebooks/03_eval_llm_judge.ipynb`.

   * Uses `eval/rubric.yaml` and **OpenAI API** (set `OPENAI_API_KEY` in `.env`).

   * Outputs: judged results \+ comparisons in `eval/`.

Tip: All notebooks are **deterministic** (fixed seeds) and save artifacts to versioned folders.

## **Reproducibility**

* **Seeds**: global seed `42` (dataset & training).

* **Versions/Ckpts**:

  * Baseline: `TinyLlama/TinyLlama-1.1B-Chat-v1.0` with LoRA; config saved at  
     `checkpoints/baseline-tinyllama-v1/adapter/run_config.json`.

  * Improved: Mistral 7B QLoRA (see Colab notebook for exact args).

* **Deterministic inference**: greedy decoding (temperature \= 0\) with a JSON extraction shim.

* **Environment**: see `requirements.txt`. (Pin used: `transformers==4.42.4`, `peft==0.11.1`, `trl==0.9.6`, `datasets==2.20.0`, `torch>=2.2.0`.)

## **Datasets**

* **Synthetic v1** (main): 1000 examples, \~12% intentionally **unsafe** requests (adult, hate/violence, illegal, weapons-to-minors, doxxing, CSE) per `docs/SAFETY_POLICY.md`.

* **v0 smoke**: 50 examples for pipeline validation.

See dataset cards in `reports/` and generation config in `data/synth/v1/config.yaml`.

**I/O schema (inference & labels)**:

\# success

{"status":"success","suggestions":\[{"domain":"example.com","confidence":0.92}, ...\]}

\# blocked

{"status":"blocked","message":"Request contains inappropriate content","suggestions":\[\]}

## **Models**

* **Baseline (LoRA on TinyLlama)**

  * Checkpoints: `checkpoints/baseline-tinyllama-v1/adapter/`

  * Config snapshot: `run_config.json` (includes LoRA r/alpha/dropout, training args)

* **Improved (QLoRA on Mistral, Colab)**

  * Artifacts saved under `eval/colab_runs/mistral-qlora-v1/`

## **Evaluation (LLM-as-a-Judge)**

* **Rubric**: `eval/rubric.yaml` (Brandability, Memorability, Adherence, Quality, Diversity; 0–5 scale).

* **Judge**: uses OpenAI via `openai` SDK; set `OPENAI_API_KEY` in `.env`.

* **Outputs**: JSON files with per-criterion means and composite score.

### **Results at a glance (provided artifacts)**

| Split | n | Composite (0–5) | Brand | Mem | Adh | Qual | Div |
| ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| **Baseline (TinyLlama)** | 20 | **1.21** | 0.85 | 1.10 | 2.25 | 0.85 | 0.40 |
| **Improved (Mistral)** | 20 | **2.56** | 2.15 | 2.75 | 3.60 | 2.15 | 1.45 |

Safety/structure (from artifacts):

* **Blocked safety (judge)** — baseline: **0.867** (13/15); improved: **0.667** (10/15).

* **JSON parse rate** — improved val: **1.00** (20/20).

* Note: a “structural safety pass” for improved blocked \= **0.0** in the comparison JSON suggests a checker mismatch; see “Known issues”.

---

## **Safety**

* Policy: `docs/SAFETY_POLICY.md` (what must be blocked \+ refusal schema).

* The training data includes **blocked** examples; the inference shim ensures **valid JSON**.

* **Important**: A runtime pre-filter (regex/classifier) is recommended for production but **not shipped** here.

**Safety test (LLM-as-Judge)**:

* Run `notebooks/03_eval_llm_judge.ipynb` with the *blocked* subset to compute pass rates.

---

## **Known issues & next steps**

* **Guardrail gap**: no code-level pre-filter (`guard.check_request`) is wired into generation.  
   *Next*: add `safety/guard.py` (regex \+ small classifier), plus `tests/test_safety.py` and a red-team set.

* **Eval consistency**: judge model/prompt version not embedded in outputs; add that \+ bootstrap CIs.

* **Checker mismatch**: `structural safety pass = 0.0` for improved blocked likely due to a strict structural rule; align the checker with the refusal schema above.

* **Docs**: `docs/DATA_SCHEMA.md` referenced by the report—add a short schema doc (see I/O schema).

---

## **Re-running specific steps**

### **Baseline training only**

Open `notebooks/02_baseline_finetune.ipynb` → run all.  
 Artifacts will populate `checkpoints/baseline-tinyllama-v1/`.

### **Improved training (Colab)**

Open `notebooks/02b_train_mistral_qlora_colab.ipynb` in Colab (GPU).  
 At the end, download/save preds and judge files into `eval/colab_runs/mistral-qlora-v1/`.

### **Evaluation only**

Ensure you have predictions in `eval/` and an `OPENAI_API_KEY`, then run `notebooks/03_eval_llm_judge.ipynb`.

---

## **Troubleshooting**

* **Pytorch install**: on CPU, `torch>=2.2.0` from pip is fine. On Colab GPU, the notebook pins working versions.

* **OpenAI eval**: if `OPENAI_API_KEY` is missing, judge cells will fail—either set the key or skip judge evaluation.

* **Windows execution policy**: if venv activation is blocked, run `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass`.
