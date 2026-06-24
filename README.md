# ai201-project3-takemeter
# TakeMeter — NBA Discourse Quality Classifier

A fine-tuned text classifier that evaluates discourse quality in r/nba by categorizing posts as `analysis`, `hot_take`, or `reaction`. Built using `distilbert-base-uncased` fine-tuned on a manually annotated dataset of NBA Reddit posts.

---

## Community Choice

**Community:** r/nba  
**Why:** r/nba is one of the most active sports communities on Reddit, with thousands of posts and comments daily. Discourse quality varies enormously — some posts build structured arguments using advanced statistics and historical comparisons, while others are pure emotional reactions to games or bold unsupported opinions. The distinction between evidence-based argument and assertion is something community members actively recognize and debate, making it a strong fit for a classification task with meaningful, grounded labels.

---

## Label Taxonomy

| Label | Definition |
|---|---|
| `analysis` | A post that makes a structured argument supported by specific, verifiable evidence (stats, historical comparisons, tactical observations). The claim would hold even if emotional framing were removed. |
| `hot_take` | A bold, confident opinion stated without supporting evidence. The post asserts a position rather than arguing for it. |
| `reaction` | An immediate emotional response to a specific game, play, or moment. Little to no argument — expressing a feeling, not making a claim. |

### Examples per label

**`analysis`**
- *"Mutombo is the 2nd best rim protector of all time based on blocks per game and defensive rating."*
- *"The Bulls will not get past the Pacers in the playoffs with their current point guard situation."*

**`hot_take`**
- *"Luka Doncic will never make another NBA finals."*
- *"The NBA should raise the draft age to 20."*

**`reaction`**
- *"KAT played amazing in the series, but foul trouble really hurt him."*
- *"Brunson did not play a good all around series if we're being honest."*

---

## Data Collection

**Source:** r/nba — public posts and comments collected manually from unpopular opinion threads, game threads, and stats/breakdown threads.

**Labeling process:** Each post was read individually and assigned exactly one label using the definitions above. Ambiguous cases were resolved using the decision rule: remove opinion framing — if a real argument remains with specific verifiable evidence, label `analysis`; if not, label `hot_take`. Posts under 20 words with no stats default to `hot_take`.

### Label Distribution

| Label | Count | % |
|---|---|---|
| hot_take | 60 | 65% |
| analysis | 16 | 17% |
| reaction | 16 | 17% |
| **Total** | **92** | **100%** |

### Difficult-to-Label Examples

**Case 1:**
- Post: *"Giannis became underrated."*
- Ambiguity: Could be `hot_take` (bold claim) or `reaction` (responding to recent discourse)
- Decision: `hot_take` — asserts a position without evidence as a standalone claim.

**Case 2:**
- Post: *"This is easily the biggest collection of lukewarm takes I've ever seen."*
- Ambiguity: Could be `reaction` (emotional response) or `hot_take` (opinion about discourse quality)
- Decision: `hot_take` — makes a claim about discourse quality, not reacting to a game event.

**Case 3:**
- Post: *"OKC hate was never about the fans."*
- Ambiguity: Could be `hot_take` or `reaction` depending on context
- Decision: `hot_take` — asserts a position without evidence as a standalone claim.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased` (HuggingFace)  
**Training framework:** HuggingFace `transformers` + `datasets` + `scikit-learn`  
**Environment:** Google Colab free tier, T4 GPU  

**Training setup:**
- Epochs: 3
- Learning rate: 2e-5
- Batch size: 16
- Train/val/test split: 70% / 15% / 15% (64 / 14 / 14 examples)

**Hyperparameter decision:** Kept the default learning rate of 2e-5 rather than increasing it because DistilBERT is already pretrained and a higher learning rate risks catastrophic forgetting of the base model's language representations on a small dataset of 92 examples. With only 64 training examples, stability was prioritized over convergence speed.

---

## Baseline

**Model:** Groq `llama-3.3-70b-versatile` (zero-shot, no fine-tuning)

**Prompt used:**
```
You are classifying NBA Reddit posts into exactly one of three categories.

analysis: The post makes a structured argument supported by specific, verifiable evidence such as statistics, historical comparisons, or tactical observations.
Example: "Mutombo is the 2nd best rim protector of all time based on blocks per game and defensive rating."

hot_take: A bold, confident opinion stated without supporting evidence. The post asserts a position rather than arguing for it.
Example: "Luka Doncic will never make another NBA finals."

reaction: An immediate emotional response to a specific game, play, or moment with little to no argument.
Example: "KAT played amazing in the series, but foul trouble really hurt him."

You must respond with ONLY one of these three words: analysis, hot_take, reaction
No punctuation. No explanation. Just the label.
```

**How results were collected:** The prompt was run against all 14 test set examples via the Colab notebook's Section 5 baseline cells. All 14 responses were parseable.

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|---|---|
| Groq zero-shot baseline | 78.6% |
| Fine-tuned DistilBERT | 57.1% |

Fine-tuning showed a regression of 21.4 percentage points compared to the baseline. This is directly attributable to the small and imbalanced dataset — only 64 training examples with 42 being `hot_take`, leaving the model with insufficient signal to learn `analysis` and `reaction` boundaries.

### Per-Class Metrics — Fine-Tuned Model

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 0.00 | 0.00 | 0.00 | 3 |
| hot_take | 0.73 | 0.89 | 0.80 | 9 |
| reaction | 0.00 | 0.00 | 0.00 | 2 |
| **macro avg** | **0.24** | **0.30** | **0.27** | **14** |

### Per-Class Metrics — Groq Baseline

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 0.50 | 0.33 | 0.40 | 3 |
| hot_take | 0.90 | 1.00 | 0.95 | 9 |
| reaction | 0.50 | 0.50 | 0.50 | 2 |
| **macro avg** | **0.63** | **0.61** | **0.62** | **14** |

### Confusion Matrix — Fine-Tuned Model

| | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
|---|---|---|---|
| **True: analysis** | 0 | 1 | 2 |
| **True: hot_take** | 0 | 8 | 1 |
| **True: reaction** | 0 | 2 | 0 |

### Wrong Predictions Analysis

**Wrong prediction 1:**
- Post: *"Defensively, as advertised. Completely wrecked Charlotte's offense. Not just with blocks, but altering shots, passes, drives with his length."*
- True label: `analysis` | Predicted: `reaction` (confidence: 0.35)
- Analysis: This post describes specific defensive actions in detail, which qualifies as evidence-based observation. The model predicted `reaction` likely because the post lacks explicit statistics or numbers — it reads as in-the-moment commentary even though it is making a structured observational argument. The model appears to have learned that `analysis` requires numerical evidence specifically, not qualitative tactical observation.

**Wrong prediction 2:**
- Post: *"KAT spent the whole finals wrapping Wemby up."*
- True label: `reaction` | Predicted: `hot_take` (confidence: 0.36)
- Analysis: This short post describes a specific in-game observation from the finals, which is a `reaction`. The model predicted `hot_take` because the sentence structure — a short declarative claim about a player — resembles the pattern of unsupported assertions in the training data. With only 11 `reaction` examples in training, the model did not learn to distinguish short observational claims from bold opinions.

**Wrong prediction 3:**
- Post: *"Unpopular opinion perhaps, but it is not that GSW are getting worse, it seems every contender team has developed a plan to beat GSW."*
- True label: `analysis` | Predicted: `hot_take` (confidence: 0.36)
- Analysis: This post begins with "unpopular opinion" — a phrase that appeared frequently in `hot_take` training examples. The model latched onto this surface signal rather than reading the actual content, which is a reasoned argument about competitive adaptation across the league. This is a clear case of the model learning spurious correlations from the training data rather than the intended distinction.

### Sample Classifications

| Post | Predicted Label | Confidence |
|---|---|---|
| "Luka Doncic will never make another NBA finals." | hot_take | 0.71 |
| "Mutombo is the 2nd best rim protector of all time." | hot_take | 0.48 |
| "KAT played amazing in the series, but foul trouble really hurt him." | reaction | 0.38 |
| "The NBA should raise the draft age to 20." | hot_take | 0.65 |
| "Defensively, as advertised. Completely wrecked Charlotte's offense." | reaction | 0.35 |

**Correct prediction explained:** *"Luka Doncic will never make another NBA finals"* was correctly predicted as `hot_take` with 71% confidence. The post is a short, declarative future prediction with no supporting evidence — exactly matching the `hot_take` definition. The model's highest confidence scores cluster around short, unambiguous `hot_take` examples, which dominate the training set.

---

## Reflection: What the Model Learned vs. What I Intended

The model learned to predict `hot_take` for almost everything. This is visible in the confusion matrix — all 3 `analysis` examples and both `reaction` examples were predicted as either `hot_take` or `reaction`, with `analysis` achieving F1 of 0.00. The fine-tuned model essentially collapsed to a near-majority-class predictor.

What I intended the model to learn was the distinction between posts that provide evidence versus posts that assert without evidence versus posts that express emotion. What it actually captured was the surface pattern that most posts in the dataset are short declarative sentences — which in the training data were predominantly labeled `hot_take`.

The core problem is class imbalance: 42 of 64 training examples were `hot_take`. With only 11 `analysis` and 11 `reaction` examples, the model had insufficient signal to learn those boundaries. The Groq zero-shot baseline substantially outperformed the fine-tuned model (78.6% vs 57.1%) precisely because it had no prior bias from the imbalanced training data.

A larger, balanced dataset of 200+ examples with ~67 per class would likely reverse this outcome.

---

## Spec Reflection

**One way the spec helped:** The spec's insistence on writing `planning.md` before collecting data forced me to define decision rules for edge cases upfront. Having the `analysis` vs `hot_take` boundary written down before annotating meant I applied it consistently rather than making ad hoc decisions as I went.

**One way implementation diverged:** The spec assumed a balanced dataset of 200+ examples. In practice, time constraints resulted in only 92 examples with significant class imbalance (65% `hot_take`). This caused the fine-tuned model to underperform the zero-shot baseline — the opposite of the expected outcome. The spec's warning about class imbalance proved accurate: a label above 70% produces a model that predicts that class most of the time.

---

## AI Usage

**Instance 1: Label stress-testing**
I provided Claude with my three label definitions and asked it to generate boundary-case posts between `analysis` and `hot_take`. It produced posts that used a single stat as decoration for an opinion. This confirmed my decision rule — "remove the opinion framing; if a real argument remains, label it `analysis`" — and I tightened the definition of `analysis` to require evidence that is central to the claim, not decorative.

**Instance 2: Failure pattern analysis**
After training, I reviewed my wrong predictions and used Claude to identify systematic patterns. The analysis identified two patterns: (1) posts beginning with "unpopular opinion" were consistently predicted as `hot_take` regardless of content, and (2) short posts under 10 words were almost always predicted as `hot_take`. Both patterns reflect spurious correlations learned from the imbalanced training data rather than the intended label distinctions.

**Annotation assistance:** No LLM was used to pre-label examples. All 92 examples were labeled manually using the definitions in `planning.md`.
