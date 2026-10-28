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

TakeMeter is a text classifier built using posts from the public r/nba community.

It classifies NBA posts into three categories: `report`, `analysis`, and `take`.

The goal is to distinguish straightforward factual reporting from evidence-backed basketball reasoning and from opinions, predictions, jokes, or emotional reactions.

---

## Label Taxonomy

### `report`

**Definition:** A post that mainly reports an NBA event, transaction, injury, quote, result, or highlight without developing the poster's own argument.

**Example 1:**

> "[Charania] Phoenix Suns center Oso Ighodaro has agreed to a three-year, $29 million contract extension through the 2030-31 season..."

**Example 2:**

> "[Charania] Just in: Sacramento Kings guard Ben Simmons is believed to have sustained a torn Achilles tendon at practice Sunday..."

### `analysis`

**Definition:** A post that makes a basketball claim or judgment and supports it with specific facts, statistics, comparisons, game information, or developed reasoning.

**Example 1:**

> "[Quinn] Bronny hasn't reached the point of offensive viability in 2 years and the defensive threshold is so damn high at his size..."

**Example 2:**

> "[OC] People debate over the bar for making the Hall of Fame for players, but nobody does for coaches. I've analyzed all 22 coaches in the HoF..."

### `take`

**Definition:** A post that mainly expresses a judgment, prediction, joke, emotional response, or strong basketball opinion without a developed evidence-backed argument.

**Example 1:**

> "With Ben Simmons now out with a torn Achilles injury... Horrible to see, especially as the season hasn't even started. Seems like another season with possibly even more players having the same injury."

**Example 2:**

> "Cavs struggle to make a half court shot and Kenny Atkinson has to take matters in his own hands."

### The hardest boundary

**Which two labels:** `analysis` and `take`

**The decision rule I used every time:**

If the conclusion is supported by a specific statistic, comparison, game event, or concrete basketball reason, I label the post `analysis`.

If the post is mainly a judgment, prediction, joke, or emotional reaction without developed reasoning, I label it `take`.

For the `report` versus `analysis` boundary, if the post mainly repeats or summarizes news, a quote, transaction, injury, result, or highlight without adding its own argument, I label it `report`, even when the reported information contains statistics.

---

## The Dataset

**Where the posts came from:**

The posts came from the public r/nba Reddit community. I collected complete post text rather than trimming posts to shorter excerpts.

**How I labelled them:**

I first labeled 20 posts manually without AI assistance and marked those rows `cold`.

For the remaining posts, I used AI to suggest labels using my `report`, `analysis`, and `take` definitions. I then read and reviewed the AI-assisted rows and corrected labels when necessary before finalizing them.

I finished with 180 labeled examples. I originally planned for 200, but I used the project's timebox/stop rule and documented the smaller dataset rather than rushing the remaining examples.

**Counts per label:**

| Label | Count | Share |
|---|---:|---:|
| `report` | 111 | 61.7% |
| `analysis` | 35 | 19.4% |
| `take` | 34 | 18.9% |
| **Total** | **180** | **100%** |

No label exceeds the project's 70% maximum.

**Three hard cases**

**1.**

> **The post:**  
> "With Ben Simmons now out with a torn Achilles injury... Horrible to see, especially as the season hasn't even started. Seems like another season with possibly even more players having the same injury."

> **Could have been:** `report` or `take`

> **I chose, because:** `take`. The injury itself is factual, but the post goes beyond simply reporting it by expressing an emotional reaction and speculating that the season could have even more similar injuries.

**2.**

> **The post:**  
> "[Quinn] Bronny hasn't reached the point of offensive viability in 2 years and the defensive threshold is so damn high at his size. If he's never gonna make shots, he has to be a Jamal Shead-level pest to justify a roster spot and he isn't."

> **Could have been:** `analysis` or `take`

> **I chose, because:** `analysis`. It contains a strong judgment, but the conclusion is supported with specific basketball reasoning about offense, size, defense, shooting, and a player comparison.

**3.**

> **The post:**  
> "[Highlights] Isaiah Evans full highlights... 21 Points on 6/13 FG, 5/10 from 3, 4/4 from the line, 2 Rebounds, 3 Assists..."

> **Could have been:** `report` or `analysis`

> **I chose, because:** `report`. The post contains detailed statistics, but the statistics are mainly being presented as factual information. It does not use those numbers to develop an argument or explain why the performance happened.

---

## The Training Run

**Base model:**

`distilbert-base-uncased`

**Settings:**

- Epochs: 3
- Learning rate: `2e-5`
- Batch size: 16
- Maximum sequence length: 128
- Seed: 42
- Device: CPU
- PyTorch version: `2.14.1+cpu`

**Anything I changed from the defaults, and why:**

I did not change the training hyperparameters. I used the starter notebook's default values so that the first training run would provide a consistent baseline for later evaluation.

**Split sizes:**

The notebook used a stratified 70/15/15 train, validation, and test split.

| Label | Train | Validation | Test |
|---|---:|---:|---:|
| `report` | 77 | 17 | 17 |
| `analysis` | 25 | 5 | 5 |
| `take` | 24 | 5 | 5 |
| **Total** | **126** | **27** | **27** |

`analysis` and `take` each have only five examples in the held-out test set. Because those test subsets are small, their individual F1 scores may vary significantly when the split seed changes.

The first training run completed successfully and produced `results.json` and `test_split.csv`.

Initial held-out results for seed 42 were:

- Overall accuracy: `0.630`
- Macro F1: `0.258`
- `report` F1: `0.773`
- `analysis` F1: `0.000`
- `take` F1: `0.000`

The model performed much better on the larger `report` class than on `analysis` and `take`. I will diagnose this further in Unit 6 rather than changing the original criteria after seeing the result.

---

## How I Used AI

**Moment 1**

- **What I asked for:** I asked AI to stress-test the boundaries of my three-label taxonomy using difficult NBA post examples.
- **What came back:** It tested cases such as factual transaction posts, unsupported opinions, statistics-only posts, jokes, predictions, and posts containing evidence-backed reasoning.
- **What I changed:** I clarified my decision rules. In particular, statistics alone do not automatically make a post `analysis`; they need to be used to support a developed conclusion. I also clarified that a post primarily repeating news or a quote remains `report`.

**Moment 2**

- **What I asked for:** After completing my initial cold-labeling step, I asked AI to suggest labels for additional r/nba posts using my definitions of `report`, `analysis`, and `take`.
- **What came back:** AI supplied suggested labels for the later examples.
- **What I changed:** I reviewed the AI-assisted examples myself and corrected labels where the suggestion did not match my written decision rules.

**Pre-labelling disclosure:**

I labeled my first 20 examples manually without AI help and marked them `cold`. After that, I used AI to pre-label later examples. I reviewed the AI-assisted rows before finalizing the dataset.

I also used AI for development troubleshooting, including identifying that VS Code was using the wrong Jupyter kernel and helping me connect the notebook to the `.venv` inside the Project 3 repository.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.185 | 0.630 | +0.444 |
| Macro F1 | 0.111 | 0.258 | +0.146 |
| F1 — `analysis` | 0.333 | 0.000 | -0.333 |
| F1 — `report` | 0.000 | 0.773 | +0.773 |
| F1 — `take` | 0.000 | 0.000 | +0.000 |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->
     I expect the trained model to outperform the zero-shot baseline overall because it was fine-tuned on r/nba examples using my specific label boundaries. I expect `report` to be the strongest label, while `analysis` and `take` may remain more difficult because their boundary is more subjective and they have fewer examples.

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->
     The trained model clearly outperformed the zero-shot baseline overall, improving accuracy from 0.185 to 0.630 and macro F1 from 0.111 to 0.258. The largest improvement was on `report`, where F1 increased from 0.000 to 0.773. However, the baseline performed better on `analysis`, while neither model successfully identified `take` on this test split. This suggests that fine-tuning helped the model learn the dominant `report` class, but it did not yet learn the harder `analysis` and `take` boundaries well.



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


| true \ predicted | report | analysis | take |
|---|---|---|---|
| **report** | 17 | 0 | 0 |
| **analysis** | 5 | 0 | 0 |
| **take** | 5 | 0 | 0 |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->

The biggest confusion was 5 real `analysis` posts being predicted as `report`. There was also an equally large error where 5 real `take` posts were predicted as `report`. The model predicted every test example as `report`, so it learned the majority class but did not learn the boundaries separating `analysis` and `take` from `report`.

This confusion is strongly one-directional: no real `report` posts were predicted as `analysis` or `take`. The agreement report will help determine whether this came from inconsistent labeling or from the model failing to learn the minority classes.



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

**Agreement rate:** **30 / 30 = 100%**

I matched the staff label on all 30 posts.

I found the set relatively straightforward once I consistently applied the written decision rules. The most useful rule was that a specific fact only makes a post `analysis` when that fact actually carries the argument. Posts making unsupported general claims were `hot_take`, while posts mainly describing something that had just happened to the writer and how it felt were `reaction`.

**Disagreements:**

There were no disagreements with the staff labels, so there were no cases to adjudicate or place into piles (a), (b), or (c).


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
Because I had 100% agreement on the staff set, this check did not show evidence that I was inconsistently applying written decision rules. That does not prove that every label in my own r/nba dataset is perfect, since the staff set uses a different taxonomy, but it suggests that I can apply a clearly written taxonomy consistently. Therefore, the trained model's collapse toward `report` is more likely related to the imbalance and small number of `analysis` and `take` examples than to a general inability to follow labeling rules consistently.



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
