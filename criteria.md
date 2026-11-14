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

## 1. Overall accuracy of at least 0.72 on the held-out test split

**Why this target:** `procedure` is 59% of my 210 posts, so a model that only
ever says `procedure` already scores about 0.59. 0.72 is the smallest number
that means it learned something beyond guessing the biggest label, with room
for the 32-post test split, where one post is worth about 3 points.

---

## 2. Macro F1 of at least 0.60

**Why this target:** accuracy hides my small labels, because getting
`procedure` right carries most of the score. Macro F1 weights `concept`,
`procedure` and `decision` equally, so it can't pass on `procedure` alone.
0.60 is set below criterion 1 because `decision` will drag it down (see 3).

---

## 3. F1 for `decision` of at least 0.40

**Why this target:** `decision` is my smallest label. It has 26 posts, so only
about 4 land in the test split, and one post moves its F1 by 0.2 or more. A
looser target than the others is honest for 4 examples. Below 0.40, the model
would be finding fewer than half of the "what should I do" posts, which is the
label a reader of this forum would most want flagged.

---

## 4. F1 for `procedure` of at least 0.80

**Why this target:** with 124 examples (about 19 in test), `procedure` is the
label with enough data to be held to a high bar. Most posts in it name a form,
a rule, a fee or an institution, so the words themselves carry a strong signal.

---

## 5. The most-confident third of test predictions is at least 0.15 more accurate than the least-confident third

**Why this target:** if a confident guess isn't more reliable than an unsure
one, the confidence score can't be used to route the unsure posts to a person.
The notebook's confidence report measures exactly this gap. 0.15 is about 2
posts out of a third of 32, so it's the smallest gap that isn't noise.

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
