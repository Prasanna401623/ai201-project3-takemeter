# TakeMeter — Planning Document

## 1. Community

**Chosen community:** r/worldcup (Reddit)

r/worldcup is a subreddit dedicated to FIFA World Cup discussion. With the 2026 World Cup currently underway, the community is extremely active — producing hundreds of posts and thousands of comments daily. The discourse varies enormously in quality: some posts offer genuine tactical insight, others are bold unsupported claims, and many are pure emotional reactions to live match events. This variation makes it an ideal fit for a classification task — the distinctions between post types are real, meaningful to community members, and frequent enough to collect 200+ labeled examples without difficulty.

---

## 2. Label Taxonomy

### `analysis`
**Definition:** The post makes a specific argument backed by tactical observations, statistics, historical comparisons, or concrete evidence. It reasons rather than just asserts.

**Example 1:**
> "Morocco's defensive shape has been exceptional — they've maintained a 4-4-2 mid-block and forced opponents into fewer than 8 shots on target across 3 games. Their pressing triggers are well-drilled."

**Example 2:**
> "Spain's possession stats are misleading this tournament. 70% of their passes are lateral or backward — they're not actually creating, just recycling. Compare their xG to their possession % and the gap is massive."

---

### `hot_take`
**Definition:** A bold, confident opinion or general claim stated without supporting evidence. The post asserts rather than argues — the claim may be true, but it is not backed up.

**Example 1:**
> "Ronaldo is finished. He's been invisible this entire tournament and shouldn't even be in the squad."

**Example 2:**
> "VAR is ruining football. Every big moment gets killed by a five-minute review. The game was better without it."

---

### `reaction`
**Definition:** An immediate emotional response to a specific moment or event in a match — a goal, red card, substitution, final whistle. The post expresses a feeling tied to something that just happened, not a general claim.

**Example 1:**
> "YESSSSS THAT GOAL!!! I can't believe it, what a tournament this has been!!!"

**Example 2:**
> "RED CARD?? Are you serious right now?! That was never a sending off, terrible decision."

---

## 3. Hard Edge Cases

**The overlap:** `hot_take` and `reaction` can look similar — both are emotional, both may lack evidence. The boundary is whether the post is responding to a specific live moment or making a general claim.

**Decision rule:** If the post is clearly triggered by something that just happened in a match (a goal, a card, a result), label it `reaction`. If the post makes a general claim about a player, team, referee, or the tournament as a whole — even if emotional — label it `hot_take`.

**Worked example:**
> "That referee is an absolute joke, worst I've seen this tournament."

This could be `reaction` (they're angry at a specific call) or `hot_take` (general claim about the referee overall). → Label as `hot_take` because the claim is general ("worst this tournament"), not tied to one specific moment.

**Borderline case to watch:** Short posts with no context — "Mbappe 🐐" or "Brazil is done" — may be reactions to a live moment or just general opinions. Rule: if there's no identifiable match event being referenced, label as `hot_take`.

---

## 4. Data Collection Plan

**Source:** r/worldcup — post titles, post bodies, and top-level comments from match threads and general discussion threads.

**Collection method:** Manual copy-paste into a CSV spreadsheet. I will read each example before labeling it.

**Target distribution:** ~70 examples per label (210 total) to keep distribution balanced.

**If a label is underrepresented:** If any label falls below 20% after 150 examples, I will specifically seek out posts of that type — e.g. searching for tactical discussion posts if `analysis` is scarce.

**CSV format:** Two required columns: `text` (the post/comment), `label` (string label). A third `notes` column for difficult cases.

---

## 5. Evaluation Metrics

**Primary metric:** Per-class F1 score (harmonic mean of precision and recall for each label).

**Why not accuracy alone:** Accuracy can be misleading if the dataset is imbalanced. If 60% of examples are `reaction`, a model that always predicts `reaction` gets 60% accuracy without learning anything. F1 per class exposes this.

**Also reporting:** Overall accuracy (for comparability with baseline), confusion matrix (to see which label pairs get confused), and per-class precision and recall separately.

**Baseline comparison:** Zero-shot Llama 3.3 70B via Groq — same test set, same metrics.

---

## 6. Definition of Success

**Minimum threshold:** Fine-tuned model achieves overall accuracy ≥ 0.70 on the test set, with no individual class F1 below 0.55.

**Meaningful improvement:** Fine-tuned model outperforms the zero-shot baseline by at least 10 percentage points in overall accuracy.

**Deployment bar:** If per-class F1 is ≥ 0.70 across all three labels, the classifier would be genuinely useful as a community moderation or discourse-quality tool.

---

## 7. AI Tool Plan

### Label stress-testing
Before annotating, I will give Claude my label definitions and edge case description and ask it to generate 10 posts that sit at the boundary between `hot_take` and `reaction`. If any are unclassifiable, I will tighten the decision rule before starting annotation.

### Annotation assistance
I may use an LLM to pre-label a batch of examples by providing my label definitions and unlabeled posts. I will review and correct every pre-assigned label before accepting it. Any pre-labeled examples will be disclosed in the AI usage section of the README.

### Failure analysis
After evaluation, I will paste my misclassified examples into Claude and ask it to identify patterns — e.g. "are most errors on short posts?" or "is one label pair consistently confused?" I will verify any identified patterns by re-reading the examples myself before including them in the report.

---

## 8. Hard Annotation Decisions (updated during Milestone 3)

*(To be filled in during data collection — at least 3 specific examples that were genuinely difficult to label, with my reasoning and final decision.)*
