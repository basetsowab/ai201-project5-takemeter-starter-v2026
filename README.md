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

My community is [Money Stack Exchange](https://money.stackexchange.com/), the
personal finance and money Q&A site. Every post there is a question, so the
classifier sorts each one by **what kind of answer the asker needs**:
`concept`, an explanation of how something works; `procedure`, a specific
factual answer from a rule, form or institution's terms; or `decision`, a
judgment call about what this person should do. A tool like this could send
`decision` posts to people willing to weigh trade-offs, and `procedure` posts
to people who know the rules.

---

## Label Taxonomy

### `concept`

**Definition:** asks how or why a financial product, rule or market mechanism
works in general, and the answer would be the same for anyone who asked.

**Example 1:** ([source](https://money.stackexchange.com/questions/170076/what-exactly-is-the-source-of-the-equity-risk-premium))
> What exactly is the source of the equity risk premium? According to capital
> market theory, the reason equities as an investment class earn a premium on
> bonds is that they are "riskier"…

**Example 2:** ([source](https://money.stackexchange.com/questions/169714/since-brokers-make-money-by-trading-why-restrict-on-ipo-trading))
> Since brokers make money by trading, why restrict on IPO trading? … What do
> these brokers gain by restricting trading?

### `procedure`

**Definition:** asks how to carry out a specific step, or what a rule, form or
fee says applies, where a correct answer exists in the rules or in the
institution's terms. This includes "is this legit or a scam?" checks.

**Example 1:** ([source](https://money.stackexchange.com/questions/170204/how-do-i-sell-all-my-shares-of-a-non-vanguard-etf-including-fractional-shares))
> I own 226.4570 shares of some ETF in a Vanguard account. On the sell screen,
> the number-of-shares field only accepts a whole number. How do I sell all my
> shares…

**Example 2:** ([source](https://money.stackexchange.com/questions/169531/how-to-properly-report-a-conversion-from-traditional-ira-to-roth-ira-in-turbotax))
> How to properly report a conversion from traditional IRA to Roth IRA in
> TurboTax online?

### `decision`

**Definition:** asks which option the poster should choose for their own
situation, where the answer depends on their goals, risk or circumstances
rather than on a rule.

**Example 1:** ([source](https://money.stackexchange.com/questions/170043/should-i-let-my-mother-lien-my-car-as-part-of-her-bankruptcy))
> My mom gifted me a new car 2 years ago. She is now filing for bankruptcy and
> wants to put a lien on my car. … Should I do it and why?

**Example 2:** ([source](https://money.stackexchange.com/questions/169122/pay-off-mortgage-and-invest-or-stay-the-course))
> I owe 45000 at a 3% loan on a home worth 700,000. I have the money to pay
> this off and could invest what I am currently paying per month…

### The hardest boundary

**Which two labels:** `procedure` and `decision`

**The decision rule I used every time:** if a correct answer exists in the tax
code, the law or the institution's terms, the post is `procedure`, even when
it's phrased "should I…". It's `decision` only when two reasonable people,
given the same facts, could rightly choose differently.

---

## The Dataset

**Where the posts came from:** the 274 most recent open questions on Money
Stack Exchange (pulled through the public Stack Exchange API in November
2026), with closed questions and posts with almost no body dropped. 210 of
them were picked at random (seed 5). Each post is the title plus the full
body, untrimmed. Links and authors are in `data/money_se_sources.csv`, since
Stack Exchange content is CC BY-SA. I first chose r/personalfinance, but
Reddit couldn't be reached from the tools I was using, so I switched to the
closest community that could.

**How I labelled them:** all 210 were pre-labelled by Claude (an AI
assistant), and the `note` column in `labels.csv` says so on every row. **No
cold labels: I have not yet done the 20 labels the brief asks for without
help, and I haven't yet reviewed Claude's labels.** 14 rows Claude marked as
unsure are flagged `unsure` in the note.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `procedure` | 124 | 59.0% |
| `concept` | 60 | 28.6% |
| `decision` | 26 | 12.4% |
| **Total** | 210 | 100% |

**Three hard cases**

**1.** ([source](https://money.stackexchange.com/questions/166880/u-s-stocks-sell-highest-cost-long-term-lots-first-to-pay-less-tax))
> *The post:* "Sell Highest-Cost Long-Term Lots First to Pay Less Tax?", from
> someone who bought a little stock every day for five years.
>
> *Could have been:* `decision` or `procedure`
>
> *I chose `procedure`, because:* it reads like a choice, but the tax rules on
> lot selection and long-term vs. short-term gains settle it. Under the
> decision rule, a rule-answerable "should I" is `procedure`.

**2.** ([source](https://money.stackexchange.com/questions/166899/do-multiple-credit-cards-help-or-hurt-credit-score-for-young-person-with-no-cred))
> *The post:* a parent asking whether their 19-year-old should keep a second,
> better rewards card or stick with one, for the sake of his credit score.
>
> *Could have been:* `concept` (how credit scoring treats new accounts) or
> `decision`
>
> *I chose `decision`, because:* the real question is whether *this* son should
> keep the second card, and the answer depends on how he'll use it.

**3.** ([source](https://money.stackexchange.com/questions/166737/what-happens-if-i-overpay-a-credit-card-bill))
> *The post:* "What happens if I overpay a credit card bill?", a hypothetical
> $3,000 payment on a $2,000 bill.
>
> *Could have been:* `procedure` or `concept`
>
> *I chose `concept`, because:* it's a "what happens if" about how card
> balances work for anyone, with no account of the poster's own to settle.

<!-- TODO (basetsowab): these three are the cases Claude found hardest while
     pre-labelling. Replace or rewrite them with posts that made YOU pause
     once you've reviewed the labels. -->

---

## The Training Run

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, max length 128
tokens, seed 42. Trained on CPU (`torch 2.15.0+cpu`).

**Anything I changed from the defaults, and why:** nothing. One thing to know
about this dataset: the average post is about 980 characters, which is more
than 128 tokens, so the model often sees only the title and the first few
sentences.

**Split sizes:** 146 train / 32 val / 32 test.

| Label | Train | Val | Test |
|---|---|---|---|
| `procedure` | 86 | 19 | 19 |
| `concept` | 42 | 9 | 9 |
| `decision` | 18 | 4 | 4 |

`decision` has only **4** test examples, under the 8 the notebook warns about,
so its F1 can only move in steps of about 0.2 and will swing between seeds.

**Result:** the model predicted `procedure` for **all 32** test posts. Accuracy
was 0.594, which is exactly `procedure`'s share of the test split, and macro
F1 was 0.248. F1 was 0.745 for `procedure` and 0.000 for both `concept` and
`decision`. In other words, at these settings the fine-tune learned only the
majority label.

---

## How I Used AI

**Moment 1**

- *What I asked for:* to collect about 200 real posts from my community.
- *What came back:* Reddit was blocked for every tool Claude had, so it
  suggested Money Stack Exchange and pulled 274 recent questions through the
  public API, with links.
- *What I changed:* I switched my community from r/personalfinance to Money
  Stack Exchange.

**Moment 2**

- *What I asked for:* to design the labels and pre-label the posts.
- *What came back:* the `concept` / `procedure` / `decision` taxonomy, the
  decision rule above, and a label for every one of the 210 posts.
- *What I changed:* nothing yet. <!-- TODO (basetsowab): after reviewing,
  say which labels you changed and why. -->

**Pre-labelling disclosure:** every label in `labels.csv` was assigned by
Claude, an AI assistant. As of this commit I have not yet labelled 20 posts
cold or reviewed Claude's labels. The `note` column marks every row as
pre-labelled and not yet reviewed.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

Both models were scored on the same 32 posts in `test_split.csv`. The baseline
is `facebook/bart-large-mnli` zero-shot, using the definitions in
`label_definitions.txt`.

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.719 | 0.594 | -0.125 |
| Macro F1 | 0.603 | 0.248 | -0.354 |
| F1 — `concept` | 0.000 | 0.000 | +0.000 |
| F1 — `decision` | 1.000 | 0.000 | -1.000 |
| F1 — `procedure` | 0.809 | 0.745 | -0.063 |

**What I predicted before I looked:** no prediction was written down before
these numbers were run, so I don't have one to report.
<!-- TODO (basetsowab): leave this honest. Don't backfill a prediction. -->

**What the gap actually means:** my fine-tuning added nothing, and was worse
than no training at all. The zero-shot model, given only three one-line
definitions, beat it by 12.5 points of accuracy and found all 4 `decision`
posts, while the trained model found none. The trained model's 0.594 is
exactly what you get by always answering `procedure`. Neither model gets a
single `concept` post right, so `concept` is the label my definitions and
data both fail to make distinct.

---

## Run Log — Before

Three seeds, default settings, CPU (`torch 2.15.0+cpu`). Results are in
`results_three_seeds_before.json`.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.72 | 0.594 | 0.594 | 0.594 | MISSED |
| 2. Macro F1 | ≥ 0.60 | 0.248 | 0.248 | 0.248 | MISSED |
| 3. F1 `decision` | ≥ 0.40 | 0.000 | 0.000 | 0.000 | MISSED |
| 4. F1 `procedure` | ≥ 0.80 | 0.745 | 0.745 | 0.745 | MISSED |
| 5. Confidence gap (most- minus least-sure third) | ≥ 0.15 | 0.80 | 0.60 | 0.30 | MET |

The spread is 0.000 on every measure. That isn't stability: all three models
gave the same answer to every post.

### Confusion matrix

Seed 42. Seeds 7 and 2024 are identical.

| true \ predicted | concept | procedure | decision |
|---|---|---|---|
| **concept** | 0 | 9 | 0 |
| **procedure** | 0 | 19 | 0 |
| **decision** | 0 | 4 | 0 |

**My biggest off-diagonal number, and what it means:** 9 real `concept` posts
were called `procedure`, and 0 went the other way. That direction is total
because the model never predicts anything but `procedure`. It didn't learn
the `concept`/`procedure` boundary, or any boundary. It learned the
majority label.

---

## Verdicts and Diagnoses

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.72 | MISSED | 0.594 on all three seeds |
| 2 | Macro F1 | ≥ 0.60 | MISSED | 0.248 on all three seeds |
| 3 | F1 `decision` | ≥ 0.40 | MISSED | 0.000 on all three seeds; no post was ever predicted `decision` |
| 4 | F1 `procedure` | ≥ 0.80 | MISSED | 0.745 on all three seeds, high recall but precision only 0.594 |
| 5 | Confidence gap | ≥ 0.15 | MET | 0.80, 0.60 and 0.30, all above 0.15 |

**Diagnoses**

- **Criteria 1–4: a task the model can't reach from this much training.**
  With 146 training posts, batch size 16 and 3 epochs, the model takes about
  30 optimisation steps (10 per epoch) at learning rate 2e-5. The evidence is that every seed
  gives the identical single-column matrix: three different splits and three
  different initialisations all end at "always `procedure`". That pattern
  points at the training budget, not at any one split. The baseline also
  shows the task is reachable: a model with no training at all found every
  `decision` post.
- **Criterion 3: too few examples for a label.** `decision` has 18 training
  posts and 4 test posts. Even a working model would struggle to learn it, and
  its F1 can only move in steps of about 0.2.
- **`concept` for both models: possibly a boundary applied inconsistently.**
  Both the trained model and the baseline score 0.000 on `concept`. That could
  mean my `concept` and `procedure` labels overlap. Several of my unsure
  pre-labels (for example, the I-bond and CD interest posts) sit right on that
  line.
- **Criterion 5 MET, but read it carefully.** Every prediction is `procedure`,
  so "confident" just means "more confidently `procedure`". The posts the
  model is surest about really are `procedure` posts, which shows the model
  picked up some signal even though it never acts on it.

<!-- TODO (basetsowab): the agreement report is the evidence that separates a
     labelling problem from a model problem, especially for the concept /
     procedure line. Add it here once data/staff_labels.csv arrives from your
     TF and agreement.py has run. -->

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

**What I changed:** one setting, `EPOCHS` from 3 to 10, in section 2 of the
notebook. Nothing else changed: same `labels.csv`, same seeds, learning rate
2e-5, batch size 16, max length 128. `RUN_LABEL` was set to `"after"`, so the
before-numbers weren't overwritten.

**Which diagnosis pointed at it:** the before-run gave the same single-column
confusion matrix on all three seeds. That pointed at the training budget
(about 30 steps) rather than at any one split. Ten epochs gives the model
about 100 steps.

### Run Log — After

Results are in `results_three_seeds_after.json`.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.72 | 0.781 | 0.750 | 0.688 | MISSED |
| 2. Macro F1 | ≥ 0.60 | 0.545 | 0.522 | 0.460 | MISSED |
| 3. F1 `decision` | ≥ 0.40 | 0.000 | 0.000 | 0.000 | MISSED |
| 4. F1 `procedure` | ≥ 0.80 | 0.872 | 0.829 | 0.791 | MISSED |
| 5. Confidence gap | ≥ 0.15 | 0.60 | 0.30 | 0.40 | MET |

Confusion matrix after, seed 42:

| true \ predicted | concept | procedure | decision |
|---|---|---|---|
| **concept** | 8 | 1 | 0 |
| **procedure** | 2 | 17 | 0 |
| **decision** | 2 | 2 | 0 |

**Did it help, and how do I know:** yes, a lot, but not enough to flip a
verdict. On the same splits, accuracy rose from 0.594 to between 0.688 and
0.781. `concept` went from F1 0.000 to between 0.588 and 0.762. The model
now actually separates `concept` from `procedure`, which it never did before.
Criteria 1 and 4 are now met on two of three seeds, but seed 2024 falls just
short (0.688 and 0.791), and a target has to hold on all three. The accuracy
spread is now 0.094, and criterion 1's target of 0.72 sits inside that band.
Seed 42 now beats the zero-shot baseline's 0.719, but seed 2024 doesn't, so
the fine-tune isn't reliably better than no training.

---

## What's Still Broken

- **Criterion 3, `decision`, F1 0.000 on every seed.** The model still never
  predicts `decision`: the 4 test posts go to `concept` or `procedure`. With
  only 18 training examples it doesn't learn the label, even though the
  zero-shot baseline found all 4 from the definition alone. The next fix would
  be collecting more `decision` posts (at least 50). Money.SE has fewer of
  them than Reddit would, so that means pulling a bigger, older batch.
- **Criterion 2, macro F1.** It's capped by `decision` scoring 0. With
  `decision` at 0, reaching 0.60 would need `concept` and `procedure` to
  average an F1 of 0.90, well above what either reaches now, so in practice it
  can't pass until criterion 3 does.
- **Criteria 1 and 4.** These miss on one seed out of three. More epochs or
  more data would probably steady them, but I stopped at one change, as the
  brief asks.

**The gap between what I meant my labels to capture and what the model
learned:** I meant `decision` to capture "what should I do", but the model
learned only the line between explaining something and looking up a rule; it
sorts every "should I" post into one of those two. All 4 real `decision`
posts in the after-matrix land in `concept` or `procedure`, and none goes the
other way.

<!-- TODO (basetsowab): check these against the agreement report once you have
     data/staff_labels.csv, and say whether the concept/procedure confusion
     is a labelling problem or a model problem. -->


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
