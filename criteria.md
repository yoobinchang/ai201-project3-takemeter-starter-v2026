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

## 1. Overall accuracy is at least 0.65 for every seed

The trained classifier must achieve at least **0.65 held-out accuracy** in
each of the three required runs (seeds 42, 7, and 2024).

**Why this target:** With four labels, chance performance is about 0.25, so
0.65 would be meaningfully useful rather than a small improvement over
guessing. Requiring it for all three seeds also prevents one favorable split
from making the model look reliable.

---

## 2. Macro F1 is at least 0.55 for every seed

The classifier must achieve a **macro F1 of at least 0.55** in each of the
three runs.

**Why this target:** Accuracy can hide poor treatment of
`contextual_request`, which is only 6% of the dataset. Macro F1 gives every
label equal weight; 0.55 is demanding for a 200-row, four-label dataset while
still acknowledging the instability of the smallest class.

---

## 3. The three larger labels each reach F1 0.60

`supported_recommendation`, `bare_recommendation`, and `experience_story`
must each achieve an **F1 of at least 0.60** in every seed.

**Why this target:** These labels contain 65, 50, and 73 examples, so each
should place enough examples in the held-out split to justify a stronger
per-label target. Their hardest distinction is observable: concrete support
versus generic praise, or advice versus narration.

---

## 4. The smallest label reaches F1 0.35

`contextual_request` must achieve an **F1 of at least 0.35** in every seed.

**Why this target:** This label has only 12 examples, so a typical held-out
split may contain only about two examples and one error can move its F1
sharply. The target is intentionally lower than for the other labels, but it
still requires the model to learn something better than never predicting the
minority class.

---

## 5. Independent labelling agreement is at least 0.75

On a random audit sample of at least 30 dataset posts, a second application of
the written taxonomy must agree with the committed labels on at least **75%**
of posts.

**Why this target:** The closest boundary is between supported and bare
recommendations, so perfect agreement is unrealistic. Agreement below 0.75
would mean that the definitions or their application are too inconsistent for
model errors to be interpreted confidently.

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
