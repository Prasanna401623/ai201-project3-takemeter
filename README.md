# TakeMeter — World Cup Discourse Classifier

A fine-tuned text classifier that evaluates discourse quality in r/worldcup posts and comments, built for AI201 Project 3.

---

## Community Choice

**Community:** r/worldcup (Reddit)

r/worldcup is a subreddit dedicated to FIFA World Cup discussion. With the 2026 World Cup currently underway, the community is extremely active — producing hundreds of posts and thousands of comments daily. The discourse varies enormously in quality: some posts offer genuine tactical insight, others are bold unsupported claims, many are pure emotional reactions to live match events, and others are calm measured takes. This variation makes it an ideal fit for a classification task — the distinctions between post types are real, meaningful to community members, and frequent enough to collect 200+ labeled examples without difficulty.

---

## Label Taxonomy

### `analysis`
The post makes a specific argument backed by tactical observations, statistics, historical comparisons, or concrete evidence. It reasons rather than just asserts.

> "Spain's possession stats are misleading this tournament. 70% of their passes are lateral or backward — they're not actually creating, just recycling. Compare their xG to their possession % and the gap is massive."

> "Morocco plays like Spain/Latin America historic style. Ivory Coast and Senegal are now full of players from the Premier League. Money and infrastructure grew a lot there last decade."

### `hot_take`
A bold, confident opinion or general claim stated without supporting evidence. The post asserts rather than argues — the claim may be true, but it is not backed up.

> "Ronaldo is finished. He's been invisible this entire tournament and shouldn't even be in the squad."

> "VAR is ruining football. Every big moment gets killed by a five-minute review. The game was better without it."

### `opinion`
A calm, measured observation or take stated without strong evidence and without emotional urgency. The post shares a reasonable view but neither argues with evidence nor reacts to a specific moment.

> "Haaland is blunt and doesn't mince words. It's what makes him so likeable. Of course he is going to give 100% the rest of the tournament. But he's also realistic. France are on another level to Norway and he knows it."

> "I think Brazil will do better in the knockouts — group stages never really show what a team is capable of."

### `reaction`
An immediate emotional response to a specific moment or event in a match — a goal, red card, substitution, final whistle. The post expresses a feeling tied to something that just happened, not a general claim.

> "YESSSSS THAT GOAL!!! I can't believe it, what a tournament this has been!!!"

> "RED CARD?? Are you serious right now?! That was never a sending off, terrible decision."

---

## Data Collection

**Source:** r/worldcup — post titles, post bodies, and top-level comments from match threads and general discussion threads collected during the 2026 FIFA World Cup.

**Collection method:** Manual copy-paste from Reddit into a spreadsheet, reading each example before labeling it. Claude was used to assist with pre-labeling batches, with every label reviewed and corrected before acceptance.

**Total examples:** 200

**Label distribution:**

| Label | Count | Percentage |
|---|---|---|
| `analysis` | 48 | 24% |
| `hot_take` | 47 | 23.5% |
| `opinion` | 53 | 26.5% |
| `reaction` | 52 | 26% |

**Train / Validation / Test split:** 140 / 30 / 30 (70% / 15% / 15%), stratified by label.

---

## Difficult Annotation Examples

**Example 1:** *"Kylian Mbappé reaches Miroslav Klose's goal record at the World Cup with 16 goals."*
Initially labeled `analysis` because it references a specific statistic. Changed to `hot_take` because simply stating a stat without any argument, reasoning, or evidence-based claim is not analysis. Analysis requires the post to *do something* with the evidence.

**Example 2:** *"Mbappe will break Messi record plus maybe will do double the goals Messi has done in the World Cup as he will play next two World Cups easily."*
Initially labeled `reaction` because it felt like an excited response. Changed to `hot_take` because it makes a bold prediction with zero evidence and is not tied to any specific live match moment.

**Example 3:** *"That referee is an absolute joke, worst I've seen this tournament."*
Genuine borderline case between `reaction` and `hot_take`. Labeled `hot_take` because the claim is general — "worst I've seen this tournament" is a sweeping judgment about the referee overall, not a response to one specific moment.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased` (HuggingFace)

**Training setup:** Fine-tuned using HuggingFace `Trainer` on Google Colab with a T4 GPU. Training took approximately 30 seconds for 5 epochs on 140 examples.

**Final hyperparameters:**

| Parameter | Value | Notes |
|---|---|---|
| `num_train_epochs` | 5 | Increased from default 3 after poor initial results |
| `learning_rate` | 5e-5 | Increased from default 2e-5 after poor initial results |
| `per_device_train_batch_size` | 16 | Default — fits T4 GPU comfortably |
| `weight_decay` | 0.01 | Default |
| `warmup_steps` | 50 | Default |

**Key hyperparameter decision:** The default settings (3 epochs, learning rate 2e-5) produced validation accuracy of only 0.267 — barely above random guessing (0.25). Increasing the learning rate to 5e-5 and epochs to 5 improved validation accuracy to 0.533 at epoch 4. The model is loaded from the best checkpoint (epoch 4) automatically.

---

## Baseline Description

**Model:** `llama-3.3-70b-versatile` via Groq API (zero-shot, no task-specific training)

**Prompt approach:** The system prompt provided the community name, all four label definitions copied from planning.md, one example post per label, and explicit instructions to output only the label name with no explanation.

**How results were collected:** The same 30-example test set used for the fine-tuned model was passed to the Groq API one example at a time with a 0.1 second delay between requests. All 30 responses were parseable — 0 unparseable outputs.

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|---|---|
| Zero-shot baseline (Groq Llama 3.3 70B) | 0.633 |
| Fine-tuned DistilBERT | 0.500 |

The baseline outperformed the fine-tuned model by 13.3 percentage points — a regression rather than an improvement.

---

### Per-Class Metrics — Fine-Tuned Model

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| `analysis` | 0.38 | 0.86 | 0.52 | 7 |
| `hot_take` | 0.00 | 0.00 | 0.00 | 7 |
| `opinion` | 0.57 | 0.50 | 0.53 | 8 |
| `reaction` | 0.83 | 0.62 | 0.71 | 8 |
| **accuracy** | | | **0.50** | 30 |
| macro avg | 0.44 | 0.50 | 0.44 | 30 |

---

### Per-Class Metrics — Zero-Shot Baseline

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| `analysis` | 0.75 | 0.43 | 0.55 | 7 |
| `hot_take` | 0.43 | 0.43 | 0.43 | 7 |
| `opinion` | 0.71 | 0.62 | 0.67 | 8 |
| `reaction` | 0.67 | 1.00 | 0.80 | 8 |
| **accuracy** | | | **0.63** | 30 |
| macro avg | 0.64 | 0.62 | 0.61 | 30 |

---

### Confusion Matrix — Fine-Tuned Model

| True \ Predicted | analysis | hot_take | opinion | reaction |
|---|---|---|---|---|
| **analysis** | 6 | 0 | 1 | 0 |
| **hot_take** | 4 | 0 | 2 | 1 |
| **opinion** | 3 | 1 | 4 | 0 |
| **reaction** | 3 | 0 | 0 | 5 |

The `confusion_matrix.png` file in this repo shows the full visualization.

---

### Wrong Predictions — Analysis of 3 Failures

**Failure 1:** *"Can't believe the US scored 4 goals!"* — True: `reaction`, Predicted: `analysis` (confidence: 0.36)

This is a short, excited reaction to a match result. The model predicted `analysis` despite there being no evidence, reasoning, or tactical content whatsoever. This reveals that the model learned a superficial signal: posts mentioning specific numbers (like "4 goals") tend to be analysis. The model latched onto the number "4" rather than the emotional framing of the post.

**Failure 2:** *"Kylian Mbappé reaches Miroslav Klose's goal record at the World Cup with 16 goals."* — True: `hot_take`, Predicted: `analysis` (confidence: 0.41)

This post states a statistic but makes no argument with it. We labeled it `hot_take` because it asserts without reasoning. The model predicted `analysis` because stat-mentioning language is a strong surface signal for analysis in our training data. The model hasn't learned the distinction between *citing a stat to argue something* vs. *just stating a stat*.

**Failure 3:** *"Paraguay was dirty af for 90 minutes and an impotent Turkey tripped over themselves trying to score goals."* — True: `hot_take`, Predicted: `analysis` (confidence: 0.47)

This is a bold, unsupported opinion — a clear `hot_take`. The model predicted `analysis` with the highest confidence of any wrong prediction. The post is detailed and references specific match events, which the model associates with analysis. The model seems to have learned "mentions specific things = analysis" rather than "makes an evidence-based argument = analysis."

---

### Sample Classifications

| Post (truncated) | True Label | Predicted | Confidence |
|---|---|---|---|
| "Argentina looked comfortable, never really had to go through the gears..." | `analysis` | `analysis` | 0.52 |
| "YESSSSS THAT GOAL!!! I can't believe it..." | `reaction` | `reaction` | 0.81 |
| "Ronaldo is finished. He's been invisible this entire tournament..." | `hot_take` | `opinion` | 0.33 |
| "I think Brazil will do better in the knockouts..." | `opinion` | `opinion` | 0.61 |
| "Turkey is the first team to lose a World Cup game with 70%+ possession..." | `analysis` | `analysis` | 0.58 |

The correctly predicted `analysis` example ("Argentina looked comfortable...") is reasonable because the post contains specific tactical observations about Argentina's play style and Austria's inability to respond — exactly the kind of reasoning the label is meant to capture.

---

### Reflection: What the Model Learned vs. What Was Intended

The intended distinction was about *reasoning structure*: does the post argue with evidence, assert boldly, observe calmly, or react emotionally? What the model actually learned was closer to *surface-level text features*: longer posts with specific names, stats, or match references got labeled `analysis`; short emotional posts got labeled `reaction`; everything else defaulted to `opinion`.

The most revealing failure is `hot_take` — the model predicted it zero times on the test set. This means it never learned the `hot_take` boundary at all. The likely reason: `hot_take` posts look very similar to `opinion` posts on the surface (both lack evidence, both can be emotional), and 47 training examples was not enough to learn the subtle tonal difference between "bold and assertive" vs. "calm and measured."

The zero-shot baseline outperforming the fine-tuned model suggests that the large language model's general understanding of tone, confidence, and argument structure is more useful for this task than the surface-level patterns DistilBERT learned from 140 training examples.

---

## Spec Reflection

**One way the spec helped:** The spec's emphasis on designing labels before collecting data saved significant rework. By defining decision rules for the `hot_take` vs. `reaction` and `hot_take` vs. `opinion` boundaries upfront, annotation was faster and more consistent. Without that upfront design work, the dataset would have been noisier.

**One way implementation diverged:** The spec suggested the fine-tuned model would likely outperform the zero-shot baseline, which is the typical result. In this project the opposite happened — the baseline won by 13 points. This divergence was actually more informative than a straightforward win would have been: it revealed that 200 examples with 4 subjective labels is at the lower edge of what DistilBERT can learn from, and that the label distinctions are subtle enough that a large general model handles them better than a small specialized one.

---

## AI Usage

**Instance 1 — Annotation assistance:** Claude was used to pre-label batches of 10-20 posts at a time by providing label definitions and raw post text. Every pre-assigned label was reviewed and corrected before being accepted into the dataset. Approximately 200 examples were pre-labeled this way with corrections made to roughly 25% of them. This is disclosed here per the project requirements.

**Instance 2 — Failure analysis:** After evaluation, the list of 15 wrong predictions was analyzed with Claude to identify patterns. Claude identified that the model was over-predicting `analysis` for posts mentioning specific names or numbers, and that `hot_take` was being absorbed into both `analysis` and `opinion`. These patterns were verified by manually re-reading the misclassified examples before being included in the report above.

**Instance 3 — Label stress-testing:** Before annotation began, Claude was asked to generate boundary cases between `hot_take` and `reaction`, and between `hot_take` and `opinion`. Several generated examples were genuinely unclassifiable under the initial definitions, which led to tightening the decision rules (specifically adding the "would a neutral fan nod along or push back?" test for `opinion` vs. `hot_take`).