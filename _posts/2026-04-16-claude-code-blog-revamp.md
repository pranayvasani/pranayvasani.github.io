---
tags: [Personal, AI, Technology]
---

# I Let an AI Revamp My Blog and Lived to Tell the Tale

My blog hadn't changed its look since 2020. It ran on Jekyll's default Minima theme — functional, minimal, and if I'm honest, the digital equivalent of a government form. So I decided to try something different: let Claude Code do the whole thing while I supervised from my phone like a Project Manager who hasn't written code since 2015.

**The Plan (Capital P)**

Claude started by producing a detailed 10-point upgrade plan. Typography. Dark mode. Post cards. Search. Tags. Reading progress bar. I asked if we should start small.

"Let's implement all of them. Why restrict?"

Reader, I said that. I did this to myself.

**The First Attempt: A Lesson in Humility**

The first version went live and I sent Claude a screenshot. The header looked like someone had copy-pasted the entire blog title into a hyperlink using Internet Explorer 6. The PV badge — meant to be a sleek little green square — was just... the letters P and V floating in space, underlined, enormous, and full of hope.

My feedback: *"pathetic."*

To Claude's credit, it did not argue. It diagnosed the problem in seconds — it had replaced Minima's entire CSS from scratch instead of building on top of it — and rewrote the approach. *Import first. Override second.* A lesson that applies to CSS, management, and probably parenting.

**The Codespaces Detour**

Between PRs, I thought I'd preview things locally using GitHub Codespaces. A reasonable idea. Ruby 3.4.7 had other plans.

Bundler 2.1.4 and Ruby 3.4.7 apparently have the kind of relationship where they refuse to be in the same room. We tried `gem install bundler`. We tried `rvm rehash`. We tried `gem exec bundle` (not a real command, as it turns out). At one point Claude suggested switching Ruby versions entirely. On mobile. In a Codespace. Named "glowing space guacamole."

We did not preview locally.

**The PR Avalanche**

By the end, there were four pull requests. PR #14 broke things. PR #15 fixed some of it. PR #16 fixed the rest of it. PR #17 added ten new features. My GitHub notifications looked like a developer having an existential crisis in slow motion.

**The Good Stuff**

When it finally came together, though? Actually good. The blog now has:
- Tag badges on every post
- A working search page (Lunr.js, no backend needed)
- An archive grouped by year
- Related posts at the end of each article
- A reading progress bar
- A PV badge that is, at last, a green square and not a cry for help

Every new post I write will automatically appear in search, archive, and topics — no manual updates needed. That part is genuinely impressive.

**What I Learnt**

Claude Code is fast, capable, and refreshingly unbothered by feedback like "pathetic." It's best used for tasks where you know *what* you want but not *how* — the implementation details, the CSS architecture, the Liquid templating. Where it needs you is context: what looks good, what feels right, what your readers actually need.

Also: never try to fix Ruby version conflicts on a mobile phone. Some battles are not meant to be fought standing in a parking lot squinting at a Codespace named after a vegetable.

The blog lives. The PV badge is green. We move forward.
