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
TakeMeter classifies public posts and comments from `r/boardgames`, a large
public community with enough varied text to collect at least 200 examples.
I first read a sample of 36 posts and comments from recommendation threads and
general discussions. The classifier separates context-rich requests,
recommendations with concrete support, unsupported recommendations, and
personal experience stories. These distinctions describe visible features of
the writing rather than making a subjective judgment that a post is simply
"good" or "bad."
<!-- Your community, and what your classifier sorts posts into. Three or four
     sentences. -->



---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `contextual_request`

**Definition:** A post that asks for a recommendation or solution and supplies
at least two decision-relevant details, such as player count, age, time,
complexity, budget, previous games, or preferred mechanics.

**Example 1:**
> A requester says the gift recipient is 12, rules out games that are too
> mature or childish, and wants something that does not require a TV or PC.
> [Source](https://www.reddit.com/r/boardgames/comments/1pdsee4/daily_game_recommendations_thread_december_04_2025/)

**Example 2:**
> A couple lists games they regularly play, explains that one player dislikes
> a heavier game, and specifies that the new game must work well for two.
> [Source](https://www.reddit.com/r/boardgames/comments/1pdsee4/daily_game_recommendations_thread_december_04_2025/)

### `supported_recommendation`

**Definition:** A post recommends or discourages a game and gives at least one
concrete reason connected to the reader's needs, such as player count,
duration, mechanics, complexity, interaction, or observed play experience.

**Example 1:**
> A response recommends *Five Tribes* because it works with two players, uses
> a mancala mechanism, has quick rules, and may produce analysis paralysis.
> [Source](https://www.reddit.com/r/boardgames/comments/1pdsee4/daily_game_recommendations_thread_december_04_2025/)

**Example 2:**
> A commenter discourages *Oath* for a group that meets only once or twice per
> year because its evolving campaign and kingmaking work better with repeated
> play.
> [Source](https://www.reddit.com/r/boardgames/comments/1p0afkn/do_you_know_an_eventworthy_boardgame_that_fits_in/)

### `bare_recommendation`

**Definition:** A post recommends or discourages a game but provides no
concrete reason beyond its title, agreement, or generic praise such as
"great," "fun," or "the best."

**Example 1:**
> "Oath is an option."
> [Source](https://www.reddit.com/r/boardgames/comments/1p0afkn/do_you_know_an_eventworthy_boardgame_that_fits_in/)

**Example 2:**
> A response lists *Castles of Burgundy*, *Viticulture*, *Quacks*, and *7
> Wonders Duel* without explaining why they fit the requester.
> [Source](https://www.reddit.com/r/boardgames/comments/1pdsee4/daily_game_recommendations_thread_december_04_2025/)

### `experience_story`

**Definition:** A post primarily recounts a personal event, reaction, or
memory about playing a game and does not principally ask for or provide a
recommendation.

**Example 1:**
> A player describes repeatedly becoming angry while playing *Risk*, sometimes
> flipping the board, and later managing the same frustration as an adult.
> [Source](https://www.reddit.com/r/boardgames/comments/1un33tq/which_game_made_one_of_your_players_throw_a/)

**Example 2:**
> A player explains that *Spartacus* became a lasting favorite and that the
> group still remembers particular games roughly twelve years later.
> [Source](https://www.reddit.com/r/boardgames/comments/1c73oi0/whats_the_first_board_game_you_fell_in_love_with/)

### The hardest boundary

**Which two labels:** `supported_recommendation` and `bare_recommendation`

**The decision rule I used every time:**
<!-- e.g. "If the post names a specific checkable fact, it's `analysis`, even
     if the tone is heated." -->
If a recommendation contains at least one concrete, decision-relevant fact
beyond the game title and generic praise, I label it
`supported_recommendation`. Player count, playtime, complexity, mechanics,
interaction style, comparison with another game, or a specific observation
from play all count as support. Otherwise I label it `bare_recommendation`.
For example, "Camel Up--8 player betting on camel races; easy to learn" is
supported despite being brief because it supplies player count, activity, and
learning difficulty.

I apply an additional intent rule when a story and recommendation overlap: if
the personal story is used as evidence for advice, the post is a
`supported_recommendation`; if recounting the experience is the main purpose,
it is an `experience_story`.


---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:**
I used 200 public comments from `r/boardgames`. The text was obtained from the
public [HuggingFaceGECLM Reddit comments dataset](https://huggingface.co/datasets/HuggingFaceGECLM/REDDIT_comments),
whose `boardgames` split contains comments archived from public Reddit data.
Deleted and removed comments, duplicate text, very short fragments, and
comments outside the four-label taxonomy were excluded.

**How I labelled them:** I applied the written decision rules consistently
across the dataset. I checked that a `contextual_request` contained at least
two usable constraints and that a `supported_recommendation` contained a
concrete reason rather than praise alone. The first 20 rows are tagged `cold`,
and the remaining rows are tagged `labeled` in the `note` column.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `contextual_request` | 12 | 6.0% |
| `supported_recommendation` | 78 | 39.0% |
| `bare_recommendation` | 35 | 17.5% |
| `experience_story` | 75 | 37.5% |
| **Total** | **200** | **100%** |

**Three hard cases**

<!-- Any post that made you pause: what it was, which two labels it could have
     been, and what you chose. These are worth more than the easy 190. -->

**1.**
> *The post:* "Cranium is different from Trivial Pursuit; the questions are
> slightly easier and more active... I recommend Cranium."
>
> *Could have been:* `bare_recommendation` or `supported_recommendation`
>
> *I chose, because:* `supported_recommendation`, because the comparison and
> description of the questions give concrete reasons despite the short post.

**2.**
> *The post:* "The first serious board game I ever purchased. Still a
> favorite."
>
> *Could have been:* `bare_recommendation` or `experience_story`
>
> *I chose, because:* `bare_recommendation`, because its function is generic
> endorsement; the purchase reference does not develop into a narrated event.

**3.**
> *The post:* A commenter recommends against *Pandemic* and says that its
> gameplay is too luck-based and can make a game unwinnable.
>
> *Could have been:* `bare_recommendation` or `supported_recommendation`
>
> *I chose, because:* `supported_recommendation`, because negative advice uses
> the same rule as positive advice and the luck claim is a concrete reason.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`

**Settings:** <!-- epochs, learning rate, batch size, seed -->
3 epochs, learning rate `2e-5`, batch size 16, maximum length 128 tokens,
and seed 42. Training ran on Apple Silicon GPU (`mps`) with PyTorch 2.14.1.

**Anything I changed from the defaults, and why:**
I kept all training defaults unchanged so this first run remains directly
comparable with the course baseline and later seed runs. I changed only the
notebook identity fields and the `LABELS` list to match `labels.csv`.

**Split sizes:** <!-- train / val / test, and per-label counts in the test
split. If a label had fewer than about 8 in test, say so — it explains a lot
of next unit's variance. -->
Train: 139, validation: 31, test: 30. The test split contains 2
`contextual_request`, 10 `supported_recommendation`, 7 `bare_recommendation`,
and 11 `experience_story` examples. The first and third labels have fewer than
about eight test examples, so their per-label scores can move sharply across
seeds; `contextual_request` is especially unstable with only two test rows.


---

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.233 | 0.600 | +0.367 |
| Macro F1 | 0.161 | 0.356 | +0.195 |
| F1 — `bare_recommendation` | 0.000 | 0.000 | +0.000 |
| F1 — `contextual_request` | 0.100 | 0.000 | -0.100 |
| F1 — `experience_story` | 0.000 | 0.643 | +0.643 |
| F1 — `supported_recommendation` | 0.545 | 0.783 | +0.237 |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->
I predict that the trained model will beat the zero-shot baseline most clearly
on `contextual_request` and `bare_recommendation`, because those labels depend
on learning the taxonomy's thresholds for how many contextual details or how
much concrete support a post contains.

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->
Fine-tuning added substantial value overall: trained accuracy was 0.367 higher
and macro F1 was 0.195 higher on the same 30 posts. My prediction was only
partly right. The largest gain was on `experience_story` (+0.643 F1), followed
by `supported_recommendation` (+0.237), while the trained model did not improve
`bare_recommendation` and performed 0.100 worse on `contextual_request`. The
zero F1 for `bare_recommendation` in both runs shows that this boundary remains
unlearned, and the two-example support for `contextual_request` makes its F1
too unstable for a strong conclusion. As a prompt-sensitivity check, I also
ran the same baseline with bare label names: it reached 0.500 accuracy and
0.397 macro F1. The written definitions therefore reduced accuracy by 0.267
and macro F1 by 0.236 for this model, rather than helping it. I report the
definitions-based run above because it is the planned baseline, but the control
shows that longer definitions are not automatically a better zero-shot prompt.



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.65 each seed | 0.567 | 0.600 | 0.533 | MISSED |
| 2. Macro F1 | ≥ 0.55 each seed | 0.337 | 0.352 | 0.306 | MISSED |
| 3. F1 for each larger label | ≥ 0.60 each seed | 0.000 minimum | 0.000 minimum | 0.000 minimum | MISSED |
| 4. `contextual_request` F1 | ≥ 0.35 each seed | 0.000 | 0.000 | 0.000 | MISSED |
| 5. Independent agreement | ≥ 0.75 on 30 posts | 1.000 (not seed-based) | 1.000 (not seed-based) | 1.000 (not seed-based) | MET |

The three larger-label values report the minimum F1 among
`supported_recommendation`, `bare_recommendation`, and `experience_story`,
because criterion 3 requires every one of them to reach the target. The full
per-label results were: `supported_recommendation` 0.727, 0.696, and 0.556;
`bare_recommendation` 0.000 for all three seeds; and `experience_story` 0.621,
0.714, and 0.667. Accuracy moved 0.067 across seeds, below the course's
ten-point instability warning, but `supported_recommendation` moved 0.172 and
the two never-predicted labels show that overall accuracy hides severe
per-label failure. These runs used Apple Silicon GPU (`mps`), PyTorch 2.15.0,
and seeds 42, 7, and 2024.

**Causes for the missed criteria:**

1. **Overall accuracy — MISSED:** This was not mainly seed instability,
   because accuracy moved only 0.067. The immediate cause was class collapse:
   the model never predicted `contextual_request` or `bare_recommendation`, so
   every true post from both labels was automatically wrong.
2. **Macro F1 — MISSED:** Macro F1 stayed low because two of four labels had
   0.000 F1 in every seed. Unlike accuracy, macro F1 gives those failed labels
   equal weight, so good performance on `experience_story` could not hide
   them.
3. **Every larger label reaches 0.60 F1 — MISSED:**
   `bare_recommendation` failed in all three seeds, and
   `supported_recommendation` also fell to 0.556 for seed 2024. The matrix
   shows the learned boundary problem directly: five of seven real bare
   recommendations became `experience_story` and the other two became
   `supported_recommendation`. The model learned neither side of the
   unsupported-recommendation boundary.
4. **`contextual_request` reaches 0.35 F1 — MISSED:** This label has only 12
   labeled examples in the full dataset, about eight in each training split,
   and only two in each test split. The model never predicted it in any seed,
   so the known cause is too little minority-label evidence combined with
   severe class imbalance.

Criterion 5 was **MET**, not missed: blind application of the staff taxonomy
matched 30 of 30 staff labels. That result supports consistent rule-following,
although it cannot prove consistency on the different board-game taxonomy.

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted | contextual_request | supported_recommendation | bare_recommendation | experience_story |
|---|---|---|---|---|
| **contextual_request** | 0 | 0 | 0 | 2 |
| **supported_recommendation** | 0 | 8 | 0 | 2 |
| **bare_recommendation** | 0 | 2 | 0 | 5 |
| **experience_story** | 0 | 2 | 0 | 9 |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->
Five real `bare_recommendation` posts were called `experience_story`, while no
real `experience_story` posts were called `bare_recommendation`; this lopsided
error means the model defaults toward narration instead of recognizing short,
unsupported endorsements and has not learned that boundary.



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

**Agreement rate:** 30 / 30 = 100%

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

There were no disagreements to adjudicate or sort into piles (a), (b), and
(c). My 30 labels were saved and committed before I opened the staff answer
key.

**What the pattern in my disagreements tells me:**
The set was straightforward under the written rules, rather than evidence that
all taxonomy boundaries are always easy. Posts 1–10 use checkable facts as the
reason for their conclusions, posts 11–20 make unsupported general claims, and
posts 21–30 report immediate personal experiences. The potentially deceptive
case, the claim that times fell 3% after the shoe rules changed, is directly
covered by rule 1: its statistic decorates an already-stated conclusion, so it
is a `hot_take`, not `analysis`. Matching all 30 shows I can apply a detailed
external taxonomy consistently, but because the staff set uses different
labels and cleaner examples than my board-game dataset, it does not by itself
prove that my `bare_recommendation` boundary was applied consistently across
all 200 project posts.



---

## The Improvement

**What I changed:** I audited only the `bare_recommendation` boundary and
changed 15 of the 200 labels while leaving every post, training setting, and
other label untouched. Thirteen posts contained concrete support such as play
time, player count, mechanics, comparison, or observed results, so I changed
them from `bare_recommendation` to `supported_recommendation`. Two posts mainly
reported a personal event or reaction, so I changed them from
`bare_recommendation` to `experience_story`.

**Which diagnosis pointed at it:** `bare_recommendation` had 0.000 F1 in every
seed. In the seed-42 matrix, five of its seven test posts were called
`experience_story` and the other two were called `supported_recommendation`,
while no post was ever predicted as bare. That directional failure pointed at
the written boundary and its application, so I made one targeted relabeling
change rather than also changing hyperparameters or adding data.

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.65 each seed | 0.500 | 0.467 | 0.600 | MISSED |
| 2. Macro F1 | ≥ 0.55 each seed | 0.258 | 0.227 | 0.347 | MISSED |
| 3. F1 for each larger label | ≥ 0.60 each seed | 0.000 minimum | 0.000 minimum | 0.000 minimum | MISSED |
| 4. `contextual_request` F1 | ≥ 0.35 each seed | 0.000 | 0.000 | 0.000 | MISSED |
| 5. Independent agreement | ≥ 0.75 on 30 posts | 1.000 (not seed-based) | 1.000 (not seed-based) | 1.000 (not seed-based) | MET |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->
No. Mean accuracy fell from 0.567 before to 0.522 after, and mean macro F1
fell from 0.332 to 0.277. `bare_recommendation` remained at 0.000 F1 in all
three seeds, while accuracy spread increased from 0.067 to 0.133, crossing the
course's ten-point instability threshold. The relabeling made the written
boundary more consistent, but it also reduced the bare class from 50 to 35
examples and shifted the model's default toward `supported_recommendation`;
for seed 42, eight real `experience_story` posts were then called supported.
The change therefore made measured performance worse, and I am keeping that
result rather than hiding the failed improvement.



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->

Criteria 1 through 4 are still missed. The model still never predicts either
`bare_recommendation` or `contextual_request`, so accuracy and macro F1 cannot
reach their targets. My next step for `contextual_request` would be to collect
about 30 more unambiguous examples because only 12 exist now. For
`bare_recommendation`, I would collect new short title-only or generic-praise
recommendations instead of removing more borderline rows; the relabeling left
only 35 examples and did not teach the class. I stopped because the milestone
requires one isolated change, and adding data or changing training settings in
the same run would make its effect impossible to distinguish.



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->
I meant `bare_recommendation` to capture unsupported endorsements and
`contextual_request` to capture requests with at least two constraints, but
the model learned neither class and predicted zero examples of both. After the
relabeling it instead overgeneralized concrete recommendation language,
calling eight of eleven seed-42 `experience_story` posts
`supported_recommendation`.



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
