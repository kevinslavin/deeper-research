# deeper-research — setup for Fairfield Bio

Retrieval-first deep research as a Claude Code skill. It fetches a real document
corpus first, then writes only from what it fetched — so every claim traces to a
source that actually exists. Output is a fully-cited "Research Bible" (Markdown +
HTML) plus a bibliography and a machine-readable claims file.

Budget: **about $0.50–$0.60 of search per report**, hard-capped at $1 per run.
The AI reasoning itself is $0 if you use your Claude Code subscription.

Setup is about ten minutes. Claude can run all of it for you — just open this
file in Claude Code and say "set this up for me."

---

## 1. Clone the repo

```bash
git clone https://github.com/kevinslavin/deeper-research.git ~/Projects/deeper-research
```

## 2. Install with a modern Python

The scripts need **Python 3.11+**. macOS still ships 3.9, which will fail with
`ModuleNotFoundError: tomllib`. Check with `python3 -VV`. If it's older than 3.11,
build a virtualenv and activate it before every run:

```bash
cd ~/Projects/deeper-research
uv venv --python 3.12 .venv        # or: python3.12 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Confirm it works — you should see 662 tests pass:

```bash
python -m pytest -q
```

## 3. Register it as a skill

```bash
ln -sfn ~/Projects/deeper-research ~/.claude/skills/deeper-research
```

Restart Claude Code. It should now offer a `deeper-research` skill.

## 4. Keys

Put these in `~/.env` (create it if missing; never commit it).

**Required — the search layer:**

```
EXA_API_KEY=...
```

Get one at [exa.ai](https://exa.ai). This is non-optional: the pipeline refuses to
write anything until it has fetched a real corpus, so with no Exa key it stops
cold rather than guessing. That refusal is deliberate — it's what stops the tool
inventing citations. **Ask Kevin whether Fairfield has a shared team key before
buying your own.**

**Required — at least one non-Anthropic model.** Your Claude Code subscription
covers the main writing at $0 and is detected automatically. But the pipeline runs
an independent "adversary" pass that tries to refute the draft, and it must use a
*different* model family than the one that wrote it. With only Claude available,
that check cannot run. So add one of:

```
GOOGLE_API_KEY=...     # Gemini
XAI_API_KEY=...        # Grok
```

or install the Codex CLI (`brew install codex` + `codex login`) if you have a
ChatGPT Plus subscription — that route is $0.

**Optional:**

```
SEMANTIC_SCHOLAR_KEY=...   # higher rate limits on the free academic lookup
CONTACT_EMAIL=...          # faster, politer access to OpenAlex/Crossref
```

## 5. Provider config

Copy Kevin's working config as a starting point:

```bash
mkdir -p ~/.config/deeper-research
cp ~/Projects/deeper-research/config.toml.example ~/.config/deeper-research/config.toml
```

Then edit it to list only the providers whose keys you actually set. A provider
whose key is missing is skipped; a provider you name explicitly but can't reach is
a hard error, so don't reference what you don't have.

Verify what resolved:

```bash
cd ~/Projects/deeper-research && source .venv/bin/activate
python -c "
import os, config
t=[os.path.expanduser('~/.config/deeper-research/config.toml')]
p,_=config.load_config(t, dict(os.environ))
print('providers:', sorted(p))
rc=config.load_run_config(t, dict(os.environ))
print('writer:', rc.synthesizer, '| adversary:', rc.adversary)
print('warning:', rc.adversary_warning)
"
```

You want two different names for writer and adversary, and `warning: None`.

## 6. First run

In Claude Code, from whatever project you want the research filed under:

> Use deeper-research to research <your question>.

Results land in `<your project>/Deeper_Research/<slug>/`. Read
`RESEARCH-BIBLE_<slug>.html` in a browser.

---

## Two things worth knowing

**It will refuse thin topics.** If the search doesn't return enough distinct
sources, the run stops with exit code 22 instead of padding a report. That's
working as designed — it means the evidence isn't there, and a confident-sounding
answer would have been fabricated. Broaden the question and retry.

**Costs are pre-charged and capped.** Each search is billed against a per-run
ledger with a $1 default ceiling. Hit the cap and it exits 21 and tells you,
rather than quietly spending more.

Questions: Kevin.
