# Project 3: TakeMeter — Planning

## Community
r/AmITheAsshole (r/AITA) is a subreddit where users describe a conflict and ask
whether they were in the wrong. It is a strong fit for classification because posts
follow a consistent structure (situation → conflict → question), the vocabulary is
rich and varied, and the two label types I chose produce genuinely different kinds
of language. Interpersonal conflicts tend to use relationship words ("my mom",
"my boyfriend", "my coworker") while stranger conflicts tend to use anonymous
references ("a woman", "some guy", "a customer"). This lexical signal should be
learnable by a fine-tuned model.

## Labels

`interpersonal` — The conflict is between the poster and someone they have an
ongoing relationship with (family, romantic partner, friends, or coworkers).
The key signal is that the poster knows this person by name or role.

Example 1: "My boyfriend and I have been together for 3 years. Last night I told
him I didn't want to go to his sister's wedding and he hasn't spoken to me since.
AITA?"

Example 2: "I told my manager that her feedback in front of the whole team was
unprofessional. She said I was being too sensitive. AITA?"

`stranger_conflict` — The conflict involves someone the poster has no prior
relationship with (a customer, a stranger in public, a new neighbor, etc.).
The key signal is that the poster has no ongoing relationship with the other party.

Example 1: "A woman at the grocery store kept cutting in front of me in the
checkout line. I finally said something loudly and she started crying. AITA?"

Example 2: "I was at a coffee shop and a guy kept playing music out loud with no
headphones. I asked the barista to say something instead of asking him directly.
AITA?"

## Hard Edge Cases
The hardest case is a conflict with a neighbor or coworker the poster has only
met once or twice. Decision rule: if the poster refers to an ongoing relationship
("my neighbor of 5 years", "my coworker on my team") → `interpersonal`. If the
interaction is one-time or incidental ("a neighbor I've never spoken to",
"someone in my office building") → `stranger_conflict`. When genuinely unclear,
I default to `stranger_conflict` and note it in a comments column.

## Data Collection Plan
I will manually copy-paste posts from reddit.com/r/AmITheAsshole, targeting
100 examples per label (200 total). I will browse the Hot, Top (past month), and
New feeds to get variety. If one label is underrepresented after 150 posts, I will
search specific keywords to find more — e.g. "stranger" or "customer" to find
more stranger_conflict posts.

## Evaluation Metrics
I will report accuracy, precision, recall, and F1 per class. Accuracy alone is not
enough because if the dataset is 70/30 split, a model that always predicts the
majority class gets 70% accuracy while being useless. F1 per class shows whether
the model is actually learning both categories. I will also inspect the confusion
matrix to see which direction errors go — whether the model confuses
stranger_conflict as interpersonal or vice versa.

## Definition of Success
I will consider the classifier genuinely useful if the fine-tuned model achieves
at least 0.75 F1 on both classes on the test set, and meaningfully outperforms
the zero-shot Groq baseline (by at least 0.05 accuracy). A model that performs
below baseline after fine-tuning would suggest a labeling or data quality problem.

## AI Tool Plan

**Label stress-testing:** Before annotating 200 examples, I will give Claude my
two label definitions and ask it to generate 5-10 posts that sit at the boundary
between the two labels. If the generated posts are ones I cannot cleanly classify,
I will tighten my definitions before starting annotation.

**Annotation assistance:** I will use an LLM (Claude) to pre-label batches of
examples before reviewing them myself. I will track which examples were pre-labeled
and note this in the AI usage section of my README. I will review every pre-labeled
example and correct any I disagree with.

**Failure analysis:** After getting my wrong predictions from the notebook, I will
paste them into Claude and ask it to identify patterns — e.g. are most errors posts
where the poster mentions both a known person and a stranger? I will verify the
patterns myself before writing them up in my evaluation report.
