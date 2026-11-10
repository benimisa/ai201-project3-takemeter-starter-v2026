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

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



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
