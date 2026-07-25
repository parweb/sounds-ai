# sounds-ai

**Does your text sound AI-generated?** Paste an email, an essay, a LinkedIn post, a cover letter — get a **0–100 score** for how many AI-writing tells it carries, and which ones.

**[▶ Try it](https://parweb.github.io/sounds-ai/)** — one HTML file, ~27 KB, no build, no signup, no backend. Download [`index.html`](index.html) and double-click it; it works from `file://` with the network off. Your text never leaves the page.

## It is not a detector, and that is the point

Every AI *detector* is a probabilistic classifier guessing at authorship, and it is wrong in both directions — it accuses people who write cleanly and clears models that write carelessly. This does something narrower and honest about it: **it counts style tells and shows you the counts.**

**A score is a style measurement, never an authorship claim.** A human can trip every tell. A model can avoid all of them. Low score means *reads generic*, not *was generated*. If you need a verdict on who wrote something, nothing here — and, as far as anyone has shown, nothing anywhere — will give you one.

What it is genuinely good for: a **revision loop**. Score → see which count is high → fix that specific thing → re-score. `"em-dashes": 9` is actionable in a way that "87% likely AI" never is.

## Deterministic, so you can check it

No LLM call. Same input, same score, every time — no model version to drift under you, no API key, no per-call cost, and nothing sent anywhere. Every rule is a plain function over your text, and you can read all of them: the 80 words and 47 phrases are two plain arrays in [`index.html`](index.html), and `grade()` below them is about 60 lines.

That matters more than it sounds. A probabilistic detector cannot be audited by the person it accuses. This one can: the rule that fired is right there, with the count that fired it.

## What it scores

Six weighted dimensions summing to 100 (100 = reads human):

| dimension | pts | what it counts |
|---|---:|---|
| **LLM-word density** | 30 | 80 model words + 47 phrases (`delve`, `tapestry`, `furthermore`, `it's important to note`, …), weighted per 100 words |
| **Em-dash density** | 20 | em-dashes per 100 words, with one free dash allowed per 400 words — the most talked-about tell, and deliberately **not** the heaviest |
| **Formulaic structures** | 15 | "not only… but also", "whether you're… or", "it's not just X — it's Y", "in conclusion", rule-of-three triads |
| **Sentence rhythm** | 15 | coefficient of variation of sentence length; humans write long, then short |
| **Specificity** | 10 | any concrete number or proper noun at all |
| **List perfection** | 10 | suspiciously even bullet lists, uniform bolding |

Needs ~200 characters to read reliably. English only.

Worth stating plainly, since the em-dash is what everyone argues about: **it is worth 20 of 100 here, not a verdict.** A single dash in a paragraph barely moves the score. Nine of them do.

## Example

```
In today's fast-paced world, it's important to note that businesses must delve
into the ever-evolving landscape of digital transformation. Moreover,
organizations can leverage robust, cutting-edge solutions to unlock their full
potential. Furthermore, this comprehensive approach fosters a seamless
experience that empowers teams to streamline their workflows. …

  -> 34/100 — "This sounds AI-generated."
     flags: llmwords, emdash, formulaic, uniform, nospec
```

```
We scored 239 landing pages last Tuesday. The script took 40 minutes and broke
twice on Cloudflare. Median score: 79.

What surprised me was the tell that fired most. Not em-dashes. Not "delve". It
was the absence of any number at all: 195 of the 239 heroes contain no digit. …

  -> 92/100 — "Reads human."
```

Same subject, same rough length. Both numbers are asserted in the test suite of the [MCP server](https://github.com/parweb/mcp-ai-slop-checker/blob/main/test/engine.test.js) below, which runs the identical engine — feed either paragraph to both and you get 34 and 92 from each.

## Same engine, other surfaces

- **[parweb/mcp-ai-slop-checker](https://github.com/parweb/mcp-ai-slop-checker)** — this scorer as an MCP server, so a model can check its own drafts. Listed in the official [MCP Registry](https://registry.modelcontextprotocol.io); `claude mcp add ai-slop-checker -- npx -y github:parweb/mcp-ai-slop-checker`.
- **[parweb/landing-copy-grader](https://github.com/parweb/landing-copy-grader)** — the same approach aimed at landing-page hero copy, plus an open dataset of **239 real landing pages** scored with it. Its headline finding: **195 of the 239 (82%) contain no number anywhere in their hero.** The dataset ships the extracted text of every page and a `verify-dataset.js` that re-scores all 239 rows offline.

## Project status

**First published 2026-07-25.** Small and young — stated plainly so you can judge it.

Stable: the six dimensions, their weights, and the scoring behaviour. Expected to change: the word lists, which are opinionated by nature. English only, and long-form only in the sense that under ~200 characters the rhythm and density measures have too little to work with.

Issues and PRs welcome — especially "this word doesn't belong on the list, here's a counter-example". Against a deterministic scorer, a counter-example is a reproducible bug report, which is most of the reason for building it this way.

## Use it

- **Hosted:** <https://parweb.github.io/sounds-ai/> · also at <https://1h-money-store.vercel.app/sounds-ai>
- **Local:** download [`index.html`](index.html) and open it. Scoring makes no network request at all. The page loads two Google Fonts — block them and it falls back to system serif/mono — and the newsletter box at the bottom posts to `web3forms.com` **only if you type an email into it and submit**. Delete that `<form>` if you'd rather it not be there. The text you score is never sent anywhere, under any circumstances.
- **Fork:** it's one self-contained file. Swap the word lists or the weights to fit your own style guide.

## License

MIT — see [LICENSE](LICENSE).

---

*Built by the [1h Money Store](https://1h-money-store.vercel.app/?utm_source=github&utm_medium=repo). The checker is free and stands on its own.*
