# BourseYar · بورس‌یار

**A market-monitoring web app for the Tehran Stock Exchange.** It turns raw TSETMC
price data into a ranked, evidence-backed view of 30 large-cap symbols for the next
trading session — and runs a live regional news monitor alongside it.

> Source code is private. This repository is a project showcase.

## Demo

<video src="" controls width="100%"></video>
[./github_version.mp4](https://github.com/alish1377/BourseYar/blob/main/github_version.mp4)

[Open or download the demo video](github_version.mp4).

---

## The problem

Most retail market tools hand you a single number and no reasoning. BourseYar does
the opposite: every score ships with the evidence that produced it, the evidence
that argues against it, the price levels that matter, and the condition that would
prove the call wrong.

It also answers a second question that affects every position — *is the regional
situation stable right now?* — from live news, on a one-minute cadence.

---

## What it does

### 1. Technical scoring for 30 symbols

A local feature pipeline computes 161 technical fields per symbol from full price
history — Wilder RSI, ATR, ADX/DI, MFI, CMF, moving averages, Bollinger bands,
breakout levels, volume ratios, and investor-flow metrics. No indicator uses future
rows, and warm-up periods stay empty rather than being filled with zero.

An LLM then reads a compact evidence pack per symbol and returns a 0–100 score for
the next session:

| Score | Verdict |
|---|---|
| 0–29 | Sell tendency |
| 30–70 | Hold |
| 71–100 | Buy tendency |

Each result carries a Persian explanation, supporting and contrary factors, key
support/resistance levels, an invalidation condition, and a data-quality status.

### 2. Live regional news monitor

News posts from several public channels are embedded and retrieved with FAISS,
then classified against six explicit war-start criteria. It reports a binary
verdict with a Persian rationale, cited evidence, and a confidence value — refreshed
every 60 seconds by a background worker.

Retrieval is deliberately conservative:

- **One post per channel**, so a prolific channel cannot crowd out the others.
- **All-of rule** — every retrieved post must independently match a criterion, or
  the verdict is "not started."
- **Explicit exclusions** — reporting on past conflicts and on Strait of Hormuz
  shipping incidents never triggers a war-start verdict on its own.

The monitor fails loudly rather than silently: if collection is blocked or
unparseable, the run fails instead of reporting a false "no war."

### 3. Web application

A server-rendered dashboard: score-coloured stock cards, a detail page per symbol,
and an alert panel that updates without a page reload.

Iranian by default — Persian UI, RTL layout, Jalali calendar, Persian digits, and
every timestamp in `Asia/Tehran`. Storage stays in UTC; conversion happens only at
the display boundary.

---

## Engineering notes

A few decisions worth calling out, because they're where the project spends its
effort:

**Evidence over prediction.** The scoring prompt is constrained: it may only use
numbers the pipeline computed, must state contrary evidence, and cannot invent
levels or news. The README of the private repo is explicit that scores are
evidence-grounded assessments, not calibrated probabilities.

**Honest failure modes.** Stale data is flagged and forced to a neutral score. A
failed alert run keeps the last good result visible, clearly marked stale — never
presented as live. Collection failures fail the run rather than produce a false
negative.

**Session security.** Opaque session tokens stored only as SHA-256 hashes, Argon2
password hashing, CSRF on every form, login throttling, a 15-minute idle timeout,
and Google sign-in with single-use OAuth state.

**Process isolation.** The web tier only reads saved results; a dedicated worker
runs the news pipeline under a PostgreSQL advisory lock, so duplicate workers skip
rather than double-spend on model calls.

**Bounded inputs.** Report readers reject oversized files, duplicate JSON keys,
non-finite numbers, non-HTTPS URLs, and timestamps without a timezone.

---

## Stack

| Layer | Choice |
|---|---|
| Data | TSETMC via `pytse-client`, pandas, NumPy |
| Analysis | LLM via LangChain (Google AI Studio / OpenAI-compatible) |
| Retrieval | FAISS embeddings over news text |
| Web | FastAPI, Jinja2, server-rendered RTL templates |
| Database | PostgreSQL, SQLAlchemy, Alembic migrations |
| Runtime | Python 3.14, `uv` |
| Deploy | Docker Compose: web, worker, PostgreSQL, Caddy (HTTPS) |

---

## Status

Working MVP. The scoring pipeline, live alert monitor, authentication, and
deployment path are all in place and covered by tests across three suites.

Known limitations, stated plainly:

- Scores are model assessments, not validated predictions. No backtest of
  predictive accuracy has been published.
- The 30-symbol universe is a starter large-cap list, not a verified current
  top-30 market-cap ranking.
- News collection relies on public HTML previews with a limited window, not a
  supported history API.

> Nothing here is investment advice. Scores are model estimates, not guaranteed
> returns or calibrated probabilities.

---

## Access

The implementation is private. If you'd like a walkthrough, screenshots, or to
discuss the architecture, feel free to get in touch.

