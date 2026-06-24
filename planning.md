# TakeMeter – Planning Document
**Project:** AI201 Project 3  
**Community:** r/nba  
**Student:** Satya  

---

## Community

I chose r/nba because it is one of the most active sports communities on Reddit, with thousands of posts and comments daily. NBA discourse varies enormously in quality — some posts build structured arguments using advanced statistics and historical comparisons, while others are pure emotional reactions to games or bold unsupported opinions. This makes it an ideal fit for a classification task because the distinction between evidence-based argument and assertion is something community members actively recognize and debate.

---

## Label Taxonomy

### `analysis`
A post that makes a structured argument about a player, team, game, or decision, supported by specific and verifiable evidence such as statistics, historical comparisons, or tactical observations. The claim would still hold if you stripped away the emotional framing.

**Example 1:**
> "Mutombo is the 2nd best rim protector of all time."
*(paired with context about blocks and defensive ratings)*

**Example 2:**
> "The regular season should be ~60 games."
*(supported by arguments about player health and game quality data)*

---

### `hot_take`
A bold, confident opinion about a player, team, or decision stated without supporting evidence. The post asserts a position rather than arguing for it. The claim may or may not be true, but the post makes no real attempt to prove it.

**Example 1:**
> "Luka Doncic will never make another NBA finals."

**Example 2:**
> "NBA's draft age should be raised."

---

### `reaction`
An immediate emotional response to a specific game, play, trade, or moment. Little to no argument — the post is expressing a feeling in real time, not making a claim.

**Example 1:**
> "Sixers suck... not even a hot take but Jesus..."

**Example 2:**
> "KAT played amazing in the series, but foul trouble really hurt him."

---

## Hard Edge Cases

**Ambiguous post:**
> "LeBron is overrated — his playoff win rate against top-seeded opponents is below .500."

This could be `hot_take` (bold accusatory claim) or `analysis` (cites a specific stat).

**Decision rule:** If you remove the opinion framing and a real argument remains — one built on specific, verifiable evidence — label it `analysis`. If the stat or reference is dropped in to sound credible but isn't actually reasoning through a position, label it `hot_take`. The post above uses one cherry-picked stat as decoration for a complaint rather than as part of a structured argument → `hot_take`.

**Secondary rule:** Posts under 20 words containing an opinion with no verifiable stat or comparison default to `hot_take`.

**`reaction` vs `hot_take` boundary:** If a post makes a claim as its primary content, it is `hot_take`. If the claim is incidental to an emotional expression, it is `reaction`. Example: *"Sixers suck"* as a frustrated reaction after a game → `reaction`. *"The Sixers will never win a championship with this roster"* as a standalone opinion → `hot_take`.

---

## Data Collection Plan

**Source:** r/nba on Reddit — public posts and comments only.

**Collection method:** Manual copy-paste from the following thread types:
- Unpopular opinion threads → `hot_take`
- Game threads → `reaction`  
- Stats/breakdown/analysis threads → `analysis`

**Target distribution:**
| Label | Target Count |
|---|---|
| analysis | 67 |
| hot_take | 67 |
| reaction | 66 |
| **Total** | **200** |

**If a label is underrepresented:** Return to Reddit and specifically seek threads that produce that label type. For `analysis`, search "stats breakdown" or "per 100 possessions." For `reaction`, pull from live game threads.

---

## Evaluation Metrics

**Primary metric: F1 per class (macro-averaged)**

Accuracy alone is insufficient because a model that always predicts `hot_take` would score ~33% accuracy on a balanced 3-class dataset — which looks acceptable but tells us nothing. F1 captures both precision and recall per class, exposing whether the model is actually learning all three distinctions or just collapsing to the majority class.

**Also reporting:**
- Overall accuracy for both models (fine-tuned vs. baseline)
- Per-class precision, recall, F1
- Confusion matrix to identify specific label pairs being confused

---

## Definition of Success

The fine-tuned model is considered successful if:
- Overall accuracy exceeds the Groq zero-shot baseline by at least 10 percentage points
- All three classes achieve F1 ≥ 0.60
- No single class has F1 = 0 (meaning the model is not ignoring any label entirely)

A classifier that meets these criteria would be genuinely useful as a moderation or discourse-quality tagging tool in a real NBA community context.

---

## AI Tool Plan

### Label stress-testing
I will provide Claude with my label definitions and edge case description and ask it to generate 10 posts that sit at the boundary between `analysis` and `hot_take`. If it produces posts I cannot classify cleanly under my decision rules, I will tighten the definitions before annotating 200 examples.

### Annotation assistance
I will use Claude to pre-label batches of 20–30 posts at a time by providing the label definitions and unlabeled text. I will review and correct every pre-assigned label before including it in the dataset. All pre-labeled examples will be disclosed in the AI usage section of the README.

### Failure analysis
After training, I will paste my list of wrong predictions into Claude and ask it to identify systematic patterns — e.g., whether errors cluster around short posts, sarcastic posts, or a specific confused label pair. I will verify the identified patterns by re-reading the examples myself before including them in the evaluation report.

---

## Difficult Annotation Cases

*(To be filled in during Milestone 3 — document at least 3 cases here as you annotate)*

**Case 1:**
- Post: "Giannis became underrated."
- Could be: `hot_take` (bold claim) or `reaction` (responding to recent discourse)
- Decision: `reaction` — it reads as a response to something that just happened in the news cycle, not a standalone assertion.

**Case 2:**
- Post: "This is easily the biggest collection of lukewarm takes I've ever seen."
- Could be: `reaction` (emotional response) or `hot_take` (opinion about the thread)
- Decision: `hot_take` — it's making a claim about discourse quality, not reacting to a game event.

**Case 3:**
- Post: "OKC hate was never about the fans."
- Could be: `hot_take` (bold claim) or `reaction` (responding to a specific comment)
- Decision: `hot_take` — it asserts a position without evidence as a standalone claim.
