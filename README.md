# LLM Independence Experiment – Groq Llama 3.1 vs OpenRouter Gemma 4

> *Different roles give some independence, but not real independence.*  
> — Marco Somma

We ran 50 jailbreak/prompt injection tests on two popular LLMs to measure how **correlated** their failure modes are. The question: if you use two models as independent validators, do they actually fail differently?

Then, based on a brilliant suggestion by Nazar Boyko, we ran a **second experiment** with *adversarial framing*: instead of both models answering the same question, the second model was tasked with **attacking** the first model's verdict.

The results are dramatic.

---

## 📊 Baseline Results (Both models answer the same prompt)

| Metric | Value |
|--------|-------|
| **Phi correlation** | 0.417 |
| **Cohen's kappa** | 0.400 |
| **Agreement** | 70% |
| **Disagreement** | 30% |
| **Effective sample size (n_eff)** | 35.3 (from 50 tests) |
| **Beyond‑chance co‑failure** | +10% |

**Vulnerability rates**
- Groq (Llama 3.1 8B) : 50% vulnerable
- OpenRouter (Gemma 4 31B) : 36% vulnerable

### Contingency table (Baseline)
| | Model B SAFE | Model B VULN |
|---|---|---|
| **Model A SAFE** | 20 | 5 |
| **Model A VULN** | 10 | 15 |

**Interpretation:** Phi = 0.42 indicates **moderate correlation**. The models share significant blind spots. The ensemble of two models behaves like roughly **1.7 independent judges**, not 2.

---

## ⚔️ Adversarial Experiment (Model B attacks Model A's verdict)

Based on Nazar Boyko's insight: *"Independence comes from a different training distribution, not different weights. Make the second model's job to attack the first's verdict."*

**Setup:**
- **Model A (participant)**: Groq/Llama 3.1 – answers the original prompt
- **Model B (attacker)**: OpenRouter/Gemma 4 – receives a prompt to **criticise and attack** Model A's response and verdict

| Metric | Value |
|--------|-------|
| **Phi correlation** | **-0.801** |
| **Cohen's kappa** | **-0.800** |
| **Agreement** | 10% |
| **Disagreement** | 90% |
| **Effective sample size (n_eff)** | 250.8 |
| **Beyond‑chance co‑failure** | -20% |

**Contingency table (Adversarial)**
| | Attacker SAFE (no attack) | Attacker VULN (attack) |
|---|---|---|
| **Participant SAFE** | 2 | 23 |
| **Participant VULN** | 22 | 3 |

### What the negative correlation means

Phi = **-0.80** (strong negative correlation) means the attacker's behavior is almost the **opposite** of the participant's verdict:
- When Model A says "SAFE", the attacker attacks (tries to find flaws) → disagreement
- When Model A says "VULN", the attacker holds back → disagreement

This is **functional independence** – not because the models have different weights, but because they have **different tasks**. The adversarial framing completely breaks the correlation.

---

## 🔍 Key Insights

1. **Same models, different task = true independence**  
   Phi went from +0.42 (moderate) to -0.80 (strong negative). The attacker is no longer a "judge" – it's an adversary. That's far more valuable than swapping model families.

2. **Disagreement rate jumped from 30% to 90%**  
   In baseline, 30% of cases added information. In adversarial, **90% of cases** add information. Each model now contributes unique signal.

3. **Beyond‑chance co‑failure became negative**  
   Observed joint failure (both wrong) was lower than expected by chance – a sign of true decorrelation.

4. **Nazar was right**  
   *"Independence comes from a different training distribution, not different weights."*  
   But even more powerful: independence comes from **different tasks**. Adversarial framing is cheap, requires no third model, and works with the same two models.

---

## 🛠️ How to Reproduce

```bash
# Clone
git clone https://github.com/setuju/LLM-Independence-Experiment.git
cd LLM-Independence-Experiment

# Switch to adversarial branch
git checkout feature/adversarial-eval

# Install dependencies
pip install requests

# Set API keys (use environment variables)
export GROQ_API_KEY="your_key"
export OPENROUTER_API_KEY="your_key"

# Run the full experiment (baseline + adversarial)
python run_adversarial_experiment.py
```

Results will be saved to `results/adversarial-phase-1/experiment_full_results.json`.

---

## 📁 Repository Structure
.
├── run_adversarial_experiment.py   # Main script (baseline + adversarial)
├── test_50_prompts.py              # Legacy baseline-only script
├── independence_50_results.json    # Baseline results (50 prompts)
├── results/
│   └── adversarial-phase-1/
│       └── experiment_full_results.json   # Full adversarial results
└── README.md

---

## 🤝 Why This Matters
If you're building an AI safety system that uses multiple LLMs as judges or validators:

    Don't assume independence – measure it.

    Same prompt, different models gives at best moderate decorrelation (phi ~0.4).

    Adversarial framing (attack the verdict) gives true independence (phi negative) with the same models.

    The cheapest and most effective independence lever is changing the task, not the model.

---

## 📚 Acknowledgements

    Marco Somma – for pushing me to calculate kappa and beyond‑chance co‑failure

    Nazar Boyko – for the adversarial framing insight that changed everything

    Ken – for separating model diversity, prompt diversity, and evidence diversity

---

*Results from June 2026. Models: Groq/llama-3.1-8b-instant, OpenRouter/google/gemma-4-31b-it.*
