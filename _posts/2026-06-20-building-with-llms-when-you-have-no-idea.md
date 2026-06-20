---
tags: [AI, Mobile, Flutter, Claude, Experiments]
---

# An Experiment: What Can You Build with LLMs When You Have No Idea What You're Doing?

I had an idea I thought might be worth testing. Not a burning problem, not a grand vision, just curiosity about whether something like this could actually be built. So I tried.

---

## The Setup

No mobile development experience. No Android Studio background. Just an idea, Flutter, and Claude as a coding partner.

A couple of hours later, working prototype on a real phone.

Along the way: on-device ML labeling, a natural language query parser, Claude Vision (BYOK at fractions of a cent per item), and a reverse geocoding attempt that swapped a broken Flutter package for a raw HTTP call to a free API. All pieced together without knowing much about mobile development at all.

That part was genuinely interesting. How much you can now build without prior expertise in the stack.

---

## What the Experiment Revealed

The approach hit real limits quickly:

- **Keyword matching on AI-generated descriptions is too brittle.** The gap between how a model describes something and how a user searches for it is wider than expected
- **A core feature depended on data that turned out not to exist** in the source material
- **A major platform already solves the same problem natively**, and better

---

## Actual Lessons

- Spend 30 minutes checking what already exists before building
- Validate data availability before committing to a feature
- Keyword search on prose doesn't work. Embeddings are the right tool
- The speed of going from zero to prototype with LLMs is real, and worth experimenting with more

---

## If This Idea Comes Back

Vector embeddings over keyword matching, and a sharper focus on what existing platforms genuinely don't cover.

For now, useful experiment, clear conclusions, time well spent.
