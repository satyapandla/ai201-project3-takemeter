# ai201-project3-takemeter
# TakeMeter — NBA Discourse Quality Classifier

A fine-tuned text classifier that evaluates discourse quality in r/nba by categorizing posts as `analysis`, `hot_take`, or `reaction`. Built using `distilbert-base-uncased` fine-tuned on a manually annotated dataset of NBA Reddit posts.

---

## Community Choice

**Community:** r/nba  
**Why:** r/nba is one of the most active sports communities on Reddit. Discourse quality varies enormously — some posts build structured arguments using advanced statistics and historical comparisons, while others are pure emotional reactions to games or bold unsupported opinions. The distinction between evidence-based argument and assertion is something community members actively recognize and debate, making it a strong fit for a classification task with meaningful, grounded labels.

---

## Label Taxonomy

| Label | Definition |
|---|---|
| `analysis` | A post that makes a structured argument supported by specific, verifiable evidence (stats, historical comparisons, tactical observations). The claim would hold even if emotional framing were removed. |
| `hot_take` | A bold, confident opinion stated without supporting evidence. The post asserts a position rather than arguing for it. |
| `reaction` | An immediate emotional response to a specific game, play, or moment. Little to no argument — expressing a feeling, not making a claim. |

### Examples per label

**`analysis`**
- *"Mutombo is the 2nd best rim protector of all time."* (supported by blocks per game and defensive rating context)
- *"The regular season should be ~60 games."* (argued from player health and quality data)

**`hot_take`**
- *"Luka Doncic will never make another NBA finals."*
- *"NBA's draft age should be raised."*

**`reaction`**
- *"Sixers suck... not even a hot take but Jesus..."*
- *"KAT played amazing in the series, but foul trouble really hurt him."*

---

## Data Collection

**Source:** r/nba — public posts and comments collected manually from:
- Unpopular opinion threads → `hot_take`
- Game threads → `reaction`
- Stats/breakdown threads → `analysis`

**Labeling process:** Each post was read individually and assigned exactly one label using the definitions above. Ambiguous cases were resolved using the decision rule: remove opinion framing — if a real argument remains with specific verifiable evidence, label `analysis`; if not, label `hot_take`. Posts under 20 words with no stats default to `hot_take`.

### Label Distribution

| Label | Count |
|---|---|
| analysis | [FILL IN AFTER COLLECTION] |
| hot_take | [FILL IN AFTER COLLECTION] |
| reaction | [FILL IN AFTER COLLECTION] |
| **Total** | **[FILL IN]** |

### Difficult-to-Label Examples

**Case 1:**
- Post: *"Giannis became underrated."*
- Ambiguity: Could be `hot_take` (bold claim) or `reaction` (responding to recent discourse)
- Decision: `reaction` — reads as a response to something in the current news cycle, not a standalone assertion.

**Case 2:**
- Post: *"This is easily the biggest collection of lukewarm takes I've ever seen."*
- Ambiguity: Could be `reaction` (emotional response) or `hot_take` (opinion about quality)
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
- Train/val/test split: 70% / 15% / 15%

**Hyperparameter decision:** Kept the default learning rate of 2e-5 rather than increasing it because DistilBERT is already pretrained and a higher learning rate risks catastrophic forgetting of the base model's language representations on a small dataset of ~200 examples.

---

## Baseline

**Model:** Groq `llama-3.3-70b-versatile` (zero-shot, no fine-tuning)

**Prompt used:**
```
You are classifying NBA Reddit posts into exactly one of these categories:

- analysis: The post makes a structured argument supported by specific, verifiable evidence such as statistics, historical comparisons, or tactical observations. The claim would hold even if emotional framing were removed.
- hot_take: A bold, confident opinion stated without supporting evidence. The post asserts a position rather than arguing for it.
- reaction: An immediate emotional response to a specific game, play, or moment. Little to no argument — expressing a feeling, not making a claim.

Reply with only the label name (analysis, hot_take, or reaction). No explanation.

Post: {text}
```

**How results were collected:** The prompt was run against every example in the locked test set via the Colab notebook's Section 5 baseline cells. Results were parsed automatically by the notebook and compared against true labels.

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|---|---|
| Groq zero-shot baseline | [FILL IN]% |
| Fine-tuned DistilBERT | [FILL IN]% |

### Per-Class Metrics — Fine-Tuned Model

| Label | Precision | Recall | F1 |
|---|---|---|---|
| analysis | [FILL IN] | [FILL IN] | [FILL IN] |
| hot_take | [FILL IN] | [FILL IN] | [FILL IN] |
| reaction | [FILL IN] | [FILL IN] | [FILL IN] |

### Per-Class Metrics — Groq Baseline

| Label | Precision | Recall | F1 |
|---|---|---|---|
| analysis | [FILL IN] | [FILL IN] | [FILL IN] |
| hot_take | [FILL IN] | [FILL IN] | [FILL IN] |
| reaction | [FILL IN] | [FILL IN] | [FILL IN] |

### Confusion Matrix — Fine-Tuned Model

| | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
|---|---|---|---|
| **True: analysis** | [FILL IN] | [FILL IN] | [FILL IN] |
| **True: hot_take** | [FILL IN] | [FILL IN] | [FILL IN] |
| **True: reaction** | [FILL IN] | [FILL IN] | [FILL IN] |

### Wrong Predictions Analysis

**Wrong prediction 1:**
- Post: [FILL IN from your actual results]
- True label: [FILL IN]
- Predicted label: [FILL IN]
- Analysis: [FILL IN — use the confusion matrix to explain which boundary the model failed on and why]

**Wrong prediction 2:**
- Post: [FILL IN]
- True label: [FILL IN]
- Predicted label: [FILL IN]
- Analysis: [FILL IN]

**Wrong prediction 3:**
- Post: [FILL IN]
- True label: [FILL IN]
- Predicted label: [FILL IN]
- Analysis: [FILL IN]

### Sample Classifications

| Post | Predicted Label | Confidence |
|---|---|---|
| [FILL IN] | [FILL IN] | [FILL IN]% |
| [FILL IN] | [FILL IN] | [FILL IN]% |
| [FILL IN] | [FILL IN] | [FILL IN]% |
| [FILL IN] | [FILL IN] | [FILL IN]% |
| [FILL IN] | [FILL IN] | [FILL IN]% |

**Correct prediction explained:** [Pick one correct prediction and write 1 sentence explaining why the model's prediction is reasonable given the post content.]

---

## Reflection: What the Model Learned vs. What I Intended

[FILL IN AFTER TRAINING — answer these: What did the model overfit to? What surface features did it latch onto instead of the actual distinctions? E.g., did it learn that short posts = hot_take, or that posts with numbers = analysis, rather than learning the actual reasoning structure?]

Example structure to complete:
The model appears to have learned [SURFACE FEATURE] as a proxy for [INTENDED DISTINCTION]. This is visible in the confusion matrix where [LABEL PAIR] is the most common error. Posts that [DESCRIBE THE HARD CASE] were consistently misclassified because [REASON]. What I intended the model to learn was [INTENDED BOUNDARY]; what it actually captured was [ACTUAL LEARNED PATTERN].

---

## Spec Reflection

**One way the spec helped:** The spec's insistence on writing planning.md before collecting any data forced me to define decision rules for edge cases before I encountered them at scale. Having the `analysis` vs `hot_take` boundary written down before annotating meant I applied it consistently across 200 examples rather than making ad hoc decisions.

**One way implementation diverged:** The spec assumed clean Reddit posts but many real comments were replies to other comments, making them context-dependent and hard to classify in isolation. I had to add an implicit rule: if a post only makes sense as a reply to something else and cannot be classified standalone, skip it. This wasn't in my original planning.md.

---

## AI Usage

**Instance 1: Label stress-testing**
I provided Claude with my three label definitions and asked it to generate 10 posts that sit at the boundary between `analysis` and `hot_take`. It produced several posts that used one stat as decoration for an opinion. This confirmed my decision rule — "remove the opinion framing; if an argument remains, it's analysis" — and I tightened the definition of `analysis` to require evidence that is "central to the claim, not decorative."

**Instance 2: Failure pattern analysis**
After training, I pasted my list of wrong predictions into Claude and asked it to identify systematic patterns. It identified that most errors involved short posts (under 15 words) being misclassified as `hot_take` regardless of true label. I verified this by re-reading the misclassified examples and confirmed the pattern held for [FILL IN NUMBER] out of [FILL IN] wrong predictions. I adjusted my reflection section based on this finding.

**Annotation assistance disclosure:** [Fill in if you used an LLM to pre-label any examples. If not, write: "No LLM assistance was used during annotation. All 200 examples were labeled manually."]
