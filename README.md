# Project 3: TakeMeter — r/AmITheAsshole Classifier

## Overview
This project fine-tunes a DistilBERT text classifier to categorize posts from
r/AmITheAsshole into two categories: `interpersonal` (conflict with someone the
poster knows) and `stranger_conflict` (conflict with a stranger). It compares
the fine-tuned model against a zero-shot Groq baseline.

## Label Taxonomy
`interpersonal` — The conflict is between the poster and someone they have an
ongoing relationship with (family, romantic partner, friends, or coworkers).
The key signal is that the poster knows this person by name or role.

`stranger_conflict` — The conflict involves someone the poster has no prior
relationship with (a customer, a stranger in public, a new neighbor, etc.).
The key signal is that the poster has no ongoing relationship with the other party.

## Dataset
- 200 labeled examples from r/AmITheAsshole
- interpersonal: 96 examples
- stranger_conflict: 104 examples
- Collected manually from public Reddit posts
- Split: 70% train / 15% validation / 15% test

## Tools and Models
- Base model: distilbert-base-uncased (HuggingFace)
- Fine-tuning: Google Colab (T4 GPU)
- Baseline LLM: Groq llama-3.3-70b-versatile
- Training libraries: transformers, datasets, scikit-learn

## Results

### Overall Accuracy
| Model | Accuracy |
|-------|----------|
| Zero-shot baseline (Groq) | 0.867 |
| Fine-tuned DistilBERT | 0.700 |

### Per-Class Metrics — Fine-Tuned Model
| Label | Precision | Recall | F1 |
|-------|-----------|--------|----|
| interpersonal | 0.80 | 0.53 | 0.64 |
| stranger_conflict | 0.65 | 0.87 | 0.74 |

### Per-Class Metrics — Baseline (Groq)
| Label | Precision | Recall | F1 |
|-------|-----------|--------|----|
| interpersonal | 0.79 | 1.00 | 0.88 |
| stranger_conflict | 1.00 | 0.73 | 0.85 |

### Confusion Matrix (Fine-Tuned Model)
|  | Predicted: interpersonal | Predicted: stranger_conflict |
|--|--------------------------|------------------------------|
| True: interpersonal | 8 | 7 |
| True: stranger_conflict | 2 | 13 |

## Error Analysis

The fine-tuned model made 9 errors on the 30-example test set. All wrong
predictions had confidence between 0.50 and 0.52 — essentially coin flips.
This suggests the model was not confidently wrong, but genuinely uncertain
at the decision boundary.

**Main pattern: 7 of 9 errors were interpersonal posts predicted as
stranger_conflict.** The model is biased toward predicting stranger_conflict
when uncertain.

### 3 Specific Wrong Predictions

**Error #1 — interpersonal predicted as stranger_conflict (confidence: 0.51)**
> "my mom and stepdad have been living with us for nearly 3 years because one
> random day in November her landlord told her that he was selling the house
> she was renting..."

The main conflict is with the poster's mom (interpersonal), but the post
mentions a landlord (a stranger) early on as backstory. The model likely
latched onto the stranger mention and mislabeled the post. This reveals a
weakness: when a post mentions both a known person and a stranger, the model
struggles to identify who the *main* conflict is with.

**Error #2 — interpersonal predicted as stranger_conflict (confidence: 0.50)**
> "AITA for saying no to my sister for asking for her husband, my father and
> a random stranger to be in my house for a hush-hush meeting for a job?"

The word "random stranger" appears directly in the title, but the actual
conflict is with the poster's sister and father. The model likely triggered
on the surface-level keyword rather than understanding the relational structure
of the post. This is a labeling boundary issue — the post involves both types,
but the primary conflict is interpersonal.

**Error #3 — stranger_conflict predicted as interpersonal (confidence: 0.52)**
> "AITA for snapping at my boyfriend for telling a stranger that I got fillers?"

This post is labeled stranger_conflict because the conflict is about the
poster's boyfriend revealing private information to a stranger — the stranger
is the subject of the conflict. However, the post is framed entirely around
the boyfriend relationship, so the model predicted interpersonal. This is a
genuine hard case: the surface language is interpersonal but the labeling
decision requires understanding the indirect nature of the conflict.

### Why Did the Baseline Beat Fine-Tuning?

The Groq baseline (0.867) outperformed the fine-tuned DistilBERT (0.700).
Several factors likely explain this:

1. **Small dataset**: 200 examples (140 training) is on the low end for
fine-tuning. DistilBERT did not have enough examples to learn the boundary
reliably.
2. **Label ambiguity**: Several posts involve both known people and strangers.
The decision boundary requires understanding who the *primary* conflict is
with — a subtle distinction that a large LLM handles better than a small
fine-tuned model with limited data.
3. **Keyword sensitivity**: The fine-tuned model appears to rely heavily on
surface keywords like "stranger" rather than understanding relational context.
The Groq model, with its larger context window and instruction-following
ability, handles these cases better.

### What Would Improve the Fine-Tuned Model?
- More training examples (400-500 instead of 200)
- Cleaner boundary: exclude posts that involve both label types
- Tighter label definition to reduce ambiguity at the boundary

## Sample Classifications

| Text (truncated) | True Label | Predicted | Confidence |
|------------------|------------|-----------|------------|
| "A woman at the grocery store kept cutting in front of me..." | stranger_conflict | stranger_conflict | 0.91 |
| "My boyfriend and I have been together for 3 years. Last night..." | interpersonal | interpersonal | 0.88 |
| "AITA for saying no to my sister for asking for her husband...a random stranger..." | interpersonal | stranger_conflict | 0.50 |

## Reflection
This project revealed that the distinction between interpersonal and
stranger_conflict is harder than it first appears. Posts frequently mention
both known people and strangers, and the correct label depends on identifying
the *primary* conflict — a semantic judgment that requires more than keyword
matching. A fine-tuned model with 140 training examples cannot reliably learn
this; a large LLM with strong instruction-following handles it better.

One way the spec helped: the decision rule in planning.md ("label by whoever
the MAIN conflict is with") gave me a clear annotation principle, but it also
revealed that many posts are genuinely ambiguous, which contributed to noisy
training data.

One way implementation diverged from the spec: I originally planned to use
annotation assistance (LLM pre-labeling) only for batch efficiency, but I
ended up using it as a consistency check — comparing my label to the model's
and reconsidering cases where we disagreed.

## AI Usage
I used Claude (Anthropic) for the following:
1. **Label stress-testing**: Before annotating, I asked Claude to generate
boundary cases to test my label definitions.
2. **Annotation assistance**: I used Claude to pre-label batches of 20-30
posts, then reviewed and corrected each label myself. All final labels reflect
my own judgment.
3. **Failure analysis**: After getting wrong predictions from the notebook,
I asked Claude to identify patterns across the errors, then verified the
patterns by re-reading the examples myself.
