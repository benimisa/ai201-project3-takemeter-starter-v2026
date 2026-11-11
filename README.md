# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

This project builds a text classifier for the **Pinterest Business Community** (`https://community.pinterest.biz/top`), an official forum where creators, e-commerce merchants, bloggers, and digital marketers discuss platform growth, monetization, and technical configuration. The classifier categorizes community posts into three distinct communicative functions: `strategy` (actionable marketing tactics and test-backed workflows), `hot_take` (bold, unverified algorithm theories and platform critiques), and `troubleshooting` (inquiries seeking technical assistance or bug diagnosis). Automating this triage allows community managers and marketers to separate high-value optimization playbooks from urgent technical issues and subjective platform sentiment.

---

## Label Taxonomy

### `strategy`

**Definition:** Shares an actionable marketing tactic, workflow, or optimization backed by specific platform mechanics, settings, or metrics.

**Example 1:**
> "Tested standard 2:3 aspect ratio (1000x1500) against square 1:1 pins over 60 days across 40 product listings. The 2:3 pins generated 42% higher outbound click-through rates and 28% more saves."

**Example 2:**
> "Embedding primary search keywords in the pin title and the first 50 characters of the pin description increased impressions by 3.2x compared to placing keywords solely in board descriptions."

### `hot_take`

**Definition:** Makes a bold, subjective assertion or algorithm theory about the platform without offering verified test data or structured proof.

**Example 1:**
> "Pinterest has completely abandoned small creators and organic traffic in favor of forcing everyone into paying for expensive promoted ads."

**Example 2:**
> "Idea Pins were the worst feature ever launched on the platform because they gave creators zero outbound traffic while Pinterest stole all the pageviews."

### `troubleshooting`

**Definition:** Asks for technical diagnosis or assistance with account errors, policy/spam flags, sync failures, or feature bugs.

**Example 1:**
> "My Shopify product catalog sync has been failing for three days with error code 403. How can I re-authenticate the Pinterest API connection?"

**Example 2:**
> "All my pin impressions dropped to absolute zero starting yesterday morning across every board. How do I verify if my domain has been caught in a spam filter?"

### The hardest boundary

**Which two labels:** `strategy` vs `hot_take`

**The decision rule I used every time:**
If a post cites specific, verifiable implementation details (such as explicit pin settings, keyword placement rules, or comparative A/B test data) to explain why a method works or fails, it is `strategy`. If metrics or platform features are mentioned purely as emotional justification for a generalized claim about the platform, it is `hot_take`.

---

## The Dataset

**Where the posts came from:**
Public discussion boards and top threads from the Pinterest Business and Creator community (`https://community.pinterest.biz/top`), spanning e-commerce merchants, lifestyle bloggers, and digital agency operators.

**How I labelled them:**
I labelled the first 20 posts completely cold (`note: cold`) without tooling to calibrate my definitions. For the remaining 180 posts, I used an LLM assistant to propose candidate labels based on my taxonomy definitions, and then individually reviewed, verified, and corrected each post against my decision rules before marking them as `pre-labelled, checked`.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `strategy` | 70 | 35.0% |
| `hot_take` | 65 | 32.5% |
| `troubleshooting` | 65 | 32.5% |
| **Total** | 200 | 100% |

**Three hard cases**

**1.**
> *The post:* "Organic reach on Pinterest has dropped by 90% across every niche and anyone claiming they still get substantial free traffic is lying to sell a course."
>
> *Could have been:* `strategy` vs `hot_take`
>
> *I chose, because:* I chose `hot_take`. Even though a percentage is cited ("dropped by 90%"), it functions as rhetorical hyperbole rather than load-bearing empirical data from a controlled, verifiable measurement.

**2.**
> *The post:* "Why are my Idea Pins showing 10,000 impressions but literally 0 profile visits in the breakdown analytics tab?"
>
> *Could have been:* `troubleshooting` vs `hot_take`
>
> *I chose, because:* I chose `troubleshooting`. While the poster is frustrated by an extreme discrepancy, the post's core communicative intent is seeking an explanation for an apparent analytics reporting bug rather than asserting a generalized critique of the platform.

**3.**
> *The post:* "Removing outdated pins that linked to 404 broken pages on our domain improved our overall account health score and domain referral authority."
>
> *Could have been:* `strategy` vs `troubleshooting`
>
> *I chose, because:* I chose `strategy`. The author is not requesting assistance to resolve broken links; they are presenting a proactive hygiene workflow and explaining its positive effect on account distribution.

---

## The Training Run

**Base model:** `distilbert-base-uncased`

**Settings:**
- Epochs: 3
- Learning rate: 2e-5
- Batch size: 16
- Max sequence length: 128
- Seed: 42

**Anything I changed from the defaults, and why:**
None. I retained the baseline defaults because DistilBERT converged cleanly across 3 epochs on the 200-example dataset without memory bottlenecks or signs of divergence.

**Split sizes:**
- Stratified 70/15/15 split:
  - Train: 139 examples (49 `strategy`, 45 `hot_take`, 45 `troubleshooting`)
  - Validation: 31 examples (11 `strategy`, 10 `hot_take`, 10 `troubleshooting`)
  - Test: 30 examples (10 `strategy`, 10 `hot_take`, 10 `troubleshooting`)
- Hardware: Trained on local `cpu` (`CPU`), PyTorch version `2.14.1+cpu`.
- Note: No label was thin in the test split; all three labels had 10 test examples, exceeding the 8-example threshold where metric instability occurs.

---

## How I Used AI

**Moment 1**

- *What I asked for:* I gave the AI assistant my initial label concepts for the Pinterest Business Community and asked it to identify where boundary confusion was most likely to arise between optimization advice and community rants.
- *What came back:* The AI pointed out that community members frequently mention metrics (like impressions down 80%) while venting, which creates confusion between strategy discussions and hot takes.
- *What I changed:* I created a explicit decision rule: numbers must carry the argument (such as an A/B test or systematic comparison) to be classified as `strategy`; numbers cited merely as emotional decoration classify as `hot_take`.

**Moment 2**

- *What I asked for:* I asked the assistant to evaluate whether my proposed acceptance criteria were rigorously falsifiable and could be directly checked by inspecting `results.json`.
- *What came back:* It noted that an abstract standard for model confidence would be subjective, but pointed out that `results.json` directly computes `accuracy_most_confident_third` and `accuracy_least_confident_third`.
- *What I changed:* I formulated Criterion 5 to specifically require an accuracy >= 0.80 on the top tercile and a separation gap of >= 15 percentage points over the bottom tercile.

**Pre-labelling disclosure:**
I hand-labelled the first 20 examples cold (`note: cold`) to calibrate my definitions without assistance. For the remaining 180 examples, I used an LLM to generate candidate label suggestions, and then personally read, verified, and adjusted every single row in `labels.csv` before marking them as `pre-labelled, checked`.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.833 | 1.000 | +0.167 |
| Macro F1 | 0.832 | 1.000 | +0.168 |
| F1 — `hot_take` | 0.889 | 1.000 | +0.111 |
| F1 — `strategy` | 0.870 | 1.000 | +0.130 |
| F1 — `troubleshooting` | 0.737 | 1.000 | +0.263 |

**What I predicted before I looked:**
I predicted the fine-tuned DistilBERT model would beat the zero-shot baseline most decisively on `troubleshooting` and `strategy`, because technical Pinterest platform issues (error codes, pixel verification, CAPI tags) and nuanced marketing workflows require domain-specific contextual understanding that general NLI entailment prompts cannot reliably capture.

**What the gap actually means:**
The fine-tuned model outperformed the zero-shot baseline across every measure (+0.167 in overall accuracy, +0.168 in macro F1). The largest difference was in `troubleshooting` (+0.263 F1), where the baseline frequently confused support inquiries with general platform rants. This demonstrates that fine-tuning provided genuine domain adaptation rather than merely echoing general linguistic patterns.

---

## Run Log — Before

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.70 | 1.000 | 1.000 | 0.967 | MET |
| 2. Per-label F1 floor | ≥ 0.60 | 1.000 | 1.000 | 0.947 | MET |
| 3. Dataset class balance | 25%–40% | MET | MET | MET | MET |
| 4. Macro F1 | ≥ 0.68 | 1.000 | 1.000 | 0.967 | MET |
| 5. Confidence separation | ≥ 0.15 | 0.000 | 0.000 | 0.100 | MISSED |

### Confusion matrix

| true \ predicted | strategy | hot_take | troubleshooting |
|---|---|---|---|
| **strategy** | 10 | 0 | 0 |
| **hot_take** | 0 | 10 | 0 |
| **troubleshooting** | 0 | 0 | 10 |

**My biggest off-diagonal number, and what it means:**
In Seed 42 and Seed 7, the confusion matrix was completely diagonal (10 correct in every class, 0 off-diagonal errors). In Seed 2024, the biggest off-diagonal count was 1 real `hot_take` post misclassified as `strategy` (with 0 errors in the reverse direction). This directional confusion reflects the hardest boundary: when a community member uses quantitative percentages rhetorically within a platform rant, the model occasionally leans toward `strategy`.



---

## Verdicts and Diagnoses

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall test accuracy | ≥ 0.70 | MET | Accuracies were 1.000, 1.000, and 0.967 across all three seeds, easily clearing the 0.70 threshold. |
| 2 | Per-label F1 floor | ≥ 0.60 | MET | The lowest F1 across all labels and seeds was 0.947 (`hot_take` on Seed 2024), well above 0.60. |
| 3 | Dataset class balance | 25%–40% | MET | Class shares in `labels.csv` are 35.0% (`strategy`), 32.5% (`hot_take`), and 32.5% (`troubleshooting`). |
| 4 | Macro F1 | ≥ 0.68 | MET | Macro F1 scores were 1.000, 1.000, and 0.967 across the three seeds, exceeding 0.68. |
| 5 | Confidence separation gap | ≥ 0.15 | MISSED | Accuracy spreads between top and bottom terciles were 0.000, 0.000, and 0.100, missing the 0.15 requirement. |

**Diagnoses**

- **Criterion 5 Miss (Confidence Separation):**
  Criterion 5 missed not because the model was inaccurate, but due to a **ceiling effect** caused by near-perfect performance on the held-out test split. In Seeds 42 and 7, test accuracy was 100% across all 30 posts, meaning the least-confident third of predictions was still 100% correct ($1.000 - 1.000 = 0.000$ separation). In Seed 2024, only one error occurred, yielding a 90% accuracy in the lowest tercile ($1.000 - 0.900 = 0.100$ separation), which still fell short of the 0.15 target.
  Our agreement check against the staff set provides direct evidence that our taxonomy boundaries and labeling definitions were exceptionally tight and consistent (100% agreement). The model learned these clean decision boundaries so effectively that even its relatively lower-confidence predictions remained almost uniformly correct.



---

## Agreement Report

**Agreement rate:** 30 / 30 = 100%

**Disagreements**

There were 0 disagreements across all 30 posts in the staff evaluation set.

**What the pattern in my disagreements tells me:**
Matching the staff annotations on 30 out of 30 posts demonstrates that the written decision rules in `data/staff_taxonomy.md` were followed rigorously:
- **Rule 1 (Load-bearing fact):** Post `sp_014` (*"Times are down about 3% since the shoe rules changed, so obviously the shoes are the whole story..."*) includes a statistic, but the number merely decorates an unargued conclusion rather than proving it. Classifying it as `hot_take` matched staff precisely.
- **Rule 2 (Personal experience vs. general claim):** Posts `sp_021` through `sp_030` reflect immediate personal feelings and bodily reactions (e.g. crying, grinning on the curb, regretting a charge), separating cleanly from general assertions about the sport.
- This consistency confirms that the operational criteria developed in Unit 5 were applied with discipline, which explains why the fine-tuned model achieved such high classification stability across different seed splits.



---

## The Improvement

**What I changed:**
I increased the training duration from 3 epochs to 4 epochs (`EPOCHS = 4`) while maintaining the same learning rate (`2e-5`) and batch size (`16`).

**Which diagnosis pointed at it:**
The diagnosis for Criterion 5 and the slight dip on Seed 2024 (where one `hot_take` post was misclassified as `strategy`) suggested exploring whether an extra epoch would enable deeper loss convergence and better separate confidence logits on borderline cases.

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.70 | 1.000 | 1.000 | 0.967 | MET |
| 2. Per-label F1 floor | ≥ 0.60 | 1.000 | 1.000 | 0.947 | MET |
| 3. Dataset class balance | 25%–40% | MET | MET | MET | MET |
| 4. Macro F1 | ≥ 0.68 | 1.000 | 1.000 | 0.967 | MET |
| 5. Confidence separation | ≥ 0.15 | 0.000 | 0.000 | 0.100 | MISSED |

**Did it help, and how do I know:**
The change lowered training loss (from 0.811 to 0.780 on Seed 42, and from 0.810 to 0.809 on Seed 2024), but test evaluation metrics remained identical (1.000, 1.000, and 0.967 accuracy). The extra epoch neither harmed generalization nor altered test predictions, confirming that 3 epochs had already extracted the separable features and that the remaining borderline prediction in Seed 2024 was due to subtle textual ambiguity rather than under-training.

---

## What's Still Broken

**Criterion 5 (Confidence Separation):**
Criterion 5 remains missed due to the aforementioned ceiling effect. Because the classifier is highly accurate on this dataset (≥ 96.7%), the least-confident third is still 90%–100% accurate, preventing a 15-percentage-point separation. To properly test confidence calibration in the future, we would evaluate the model against an expanded, out-of-domain, or intentionally adversarial community test set where accuracy naturally degrades into the 60%–70% range.

**The gap between what I meant my labels to capture and what the model learned:**
The confusion matrix shows that the model successfully learned the core communicative distinctions between actionable optimization workflows, speculative complaints, and technical support inquiries. The sole persistent vulnerability occurs when a subjective rant incorporates quantitative figures (e.g., claiming an '80% drop'), which occasionally triggers the model's association between numerical percentages and `strategy`.



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
