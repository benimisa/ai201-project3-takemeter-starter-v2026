# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1. Overall test accuracy

The model achieves an overall accuracy of at least 0.70 on the held-out test split.

**Why this target:**
With three balanced classes, random guessing yields approximately 0.33. Reaching at least 0.70 accuracy demonstrates that the fine-tuned DistilBERT model has extracted substantive semantic patterns from Pinterest Business posts, reliably identifying the poster's underlying communicative intent for more than 2 out of every 3 held-out posts.

---

## 2. Per-label F1 floor

Every individual label (`strategy`, `hot_take`, `troubleshooting`) achieves an F1 score of at least 0.60 on the held-out test split.

**Why this target:**
In a 30-item test set with ~10 posts per category, a single false positive or negative swings F1 by 5 to 10 points. Setting a hard floor of 0.60 across all three labels prevents the model from collapsing on the difficult boundary between subjective platform speculation (`hot_take`) and test-backed optimization (`strategy`) while only scoring well on distinct technical support requests.

---

## 3. Dataset class balance

No single label represents more than 40% of the total dataset, and no label represents less than 25% (between 50 and 80 posts per class in a 200-post dataset).

**Why this target:**
The course specifies a strict 70% majority ceiling, but in a three-label setup, an even stricter band (25% to 40%) is essential. Because Pinterest forums feature high volumes of emotional venting when algorithm updates roll out, keeping `hot_take`, `strategy`, and `troubleshooting` within 25–40% prevents the model from developing a majority-class guessing heuristic.

---

## 4. Macro F1 score

The macro-averaged F1 score across all three classes is at least 0.68 on the held-out test split.

**Why this target:**
Macro F1 weights each category equally regardless of slight support variations in the 15% test split. A macro F1 target of 0.68 requires consistently high harmonic mean between precision and recall across both the nuanced marketing tactics (`strategy`) and the opinionated platform claims (`hot_take`).

---

## 5. Confidence separation gap

The model's accuracy on the most-confident third of test predictions is at least 0.80, and is at least 15 percentage points higher than its accuracy on the least-confident third.

**Why this target:**
Confidence is tracked directly in `results.json`. If a model's top-tercile confidence does not outperform its lowest tercile by at least 15 percentage points (and reach at least 0.80), its softmax probabilities reflect uncalibrated overconfidence rather than true epistemic certainty on clear vs. borderline Pinterest community posts.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
