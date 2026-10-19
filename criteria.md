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

## 1.

The classifier should achieve at least **65% overall accuracy** on the held-out test set across all three seeds.

**Why this target:**

My taxonomy has three labels: `report`, `analysis`, and `take`, so random guessing would be around 33%. I chose 65% because it is substantially above chance while still being realistic for a small dataset of about 200 labeled r/nba posts.

---

## 2.

Each of the three labels should achieve an **F1 score of at least 0.60** on the held-out test set across all three seeds.

**Why this target:**

I want the classifier to learn all three categories rather than getting high overall accuracy by performing well on only one or two labels. Since the dataset contains only about 200 examples, the test set for each label will be relatively small, so I chose 0.60 instead of a very high per-label target.

---

## 3.

No single label should make up more than **60% of the 200-post labeled dataset**.

**Why this target:**

The classifier needs enough examples of `report`, `analysis`, and `take` to learn the differences between them. Allowing one label to dominate the dataset could cause the model to favor that label instead of learning the taxonomy.

---

## 4.

A second person using my written label definitions and decision rules should agree with my labels on at least **80% of a sample of 20 posts**.

**Why this target:**

The boundary between `analysis` and `take` can sometimes be subjective, especially when a post contains both an opinion and some supporting information. I chose 80% because the definitions and decision rules should make most posts consistently labelable even if some borderline cases remain.
---

## 5.

Across all three seeds, predictions with model confidence of at least **0.80** should be correct at least **75% of the time**.

**Why this target:**

A high-confidence prediction should be more dependable than an uncertain one. Because `report`, `analysis`, and `take` can overlap in borderline posts, I chose 75% as a useful reliability target for predictions where the model reports high confidence.

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
