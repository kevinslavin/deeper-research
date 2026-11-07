# Getting started with deeper-research

So someone sent you here. Welcome! This page tells you what this is and how to get it running in about 10 minutes.

## What it is

**deeper-research** is a *skill* for [Claude Code](https://claude.com/claude-code), Anthropic's AI coding and agent tool. A skill is a folder of instructions and scripts that Claude Code picks up and uses when the request fits.

You ask it a research question. It searches the web and academic databases first, then writes a report using **only what it actually found**, so every claim points to a real source. Before you see the report it:

- checks that each citation exists,
- has a *different* AI model try to pick the draft apart, and
- marks anything disputed, uncertain or date-sensitive.

The result is a fully cited reference document, which the skill calls a "Research Bible". It comes as Markdown plus a nicely formatted HTML page you open in a browser.

## What you need

| Thing | Why | Cost |
|---|---|---|
| **Claude Code** | Runs the skill | Your Claude subscription covers the writing |
| **Python 3.11 or newer** | Runs the helper scripts | Free |
| **An Exa API key** | Does the web search | Free tier: about 17 reports a month, no card needed |
| **One non-Claude AI key** (Gemini, Grok, or OpenAI via a ChatGPT login) | The "pick it apart" check has to use a different AI than the one that wrote the draft | Free tiers or existing subscriptions usually cover it |

Each report costs roughly $0.50–$0.60 in search credit and is capped at **$1 per run**, so it can't run up a surprise bill.

## The easy way: let Claude set it up

Once you have Claude Code installed:

1. Clone this repo (step 1 below).
2. Open Claude Code in that folder: `cd ~/Projects/deeper-research && claude`
3. Say: **"Read GETTING_STARTED.md and set this up for me."**

Claude will run the steps and stop to ask whenever it needs one of your API keys. If you'd rather do it yourself, keep reading.

## Manual setup

### 1. Get the code

```bash
git clone https://github.com/kevinslavin/deeper-research.git ~/Projects/deeper-research
```

### 2. Install Python dependencies

Check your Python version first:

```bash
python3 --version
```

If it says 3.11 or higher, you're fine. macOS often ships an older version (3.9), and that one **will fail** with `ModuleNotFoundError: tomllib`. Either way, use a virtual environment with a newer Python:

```bash
cd ~/Projects/deeper-research
python3.12 -m venv .venv          # or, if you use uv: uv venv --python 3.12 .venv
source .venv/bin/activate
pip install -r requirements.txt
```

No Python 3.12 on your machine? On a Mac, run `brew install python@3.12` (requires [Homebrew](https://brew.sh)).

Optional sanity check (all tests should pass):

```bash
python -m pytest -q
```

### 3. Tell Claude Code about the skill

```bash
mkdir -p ~/.claude/skills
ln -sfn ~/Projects/deeper-research ~/.claude/skills/deeper-research
```

This creates a shortcut in Claude Code's skills folder that points to the repo. Restart Claude Code afterward.

### 4. Add your API keys

```bash
cp ~/Projects/deeper-research/.env.example ~/.env
```

Open `~/.env` in any text editor and fill in:

**Required: search.**
```
EXA_API_KEY=...
```
Get one at [dashboard.exa.ai/api-keys](https://dashboard.exa.ai/api-keys). The free tier doesn't need a credit card. Without this key the tool **refuses to run** instead of making things up. That's intentional.

**Required: at least one non-Claude model,** for the independent review. Pick one:
```
GOOGLE_API_KEY=...     # Gemini: aistudio.google.com/apikey
XAI_API_KEY=...        # Grok:   console.x.ai
```
Or, if you have ChatGPT Plus, install the Codex CLI instead (`brew install codex`, then `codex login`). That route costs nothing extra.

**Optional:**
```
SEMANTIC_SCHOLAR_KEY=...   # higher limits on academic paper lookups
CONTACT_EMAIL=...          # gets faster access to OpenAlex/Crossref
```

Never commit or share your `.env` file. It holds your keys.

### 5. Set up the provider config

```bash
mkdir -p ~/.config/deeper-research
cp ~/Projects/deeper-research/config.toml.example ~/.config/deeper-research/config.toml
```

Open that file and **remove or comment out any provider you don't have a key for**. The skill quietly skips a provider with a missing key, but if a provider you've listed explicitly can't be reached, it stops with an error.

## Your first run

Open Claude Code in any project folder and say:

> Use deeper-research to research *your question here*.

(or type `/deeper-research your question here`)

It takes a while, because it's actually reading sources. When it finishes, look in:

```
<your project folder>/Deeper_Research/<topic>/RESEARCH-BIBLE_<topic>.html
```

Open that file in your browser.

**Tip:** Give it a focused question with some scope, such as *"How effective are four-day work weeks on productivity, based on trials since 2019?"* That works better than a bare topic like *"work weeks"*.

## If something goes wrong

| What you see | What it means | What to do |
|---|---|---|
| `ModuleNotFoundError: tomllib` | Python is too old | Activate the venv (`source ~/Projects/deeper-research/.venv/bin/activate`) or install Python 3.11+ |
| Run stops with **exit code 22** | Not enough good sources were found, so it refused to guess | Broaden or rephrase the question and try again |
| Run stops with **exit code 21** | The $1 per-run spending cap was hit | That's the safety limit doing its job. Narrow the question |
| Skill doesn't show up in Claude Code | The shortcut is missing or Claude Code wasn't restarted | Re-run step 3 and restart Claude Code |
| Complaint about a missing provider or adversary | No non-Claude model is set up | Add a Gemini or Grok key, or the Codex CLI (step 4), and check the config (step 5) |

## Staying up to date

```bash
cd ~/Projects/deeper-research && git pull
```

## Want more detail?

- [`README.md`](README.md): full feature list and how it compares to other approaches
- [`SKILL.md`](SKILL.md): the complete pipeline, the instructions Claude itself follows

Questions? Ask whoever sent you the link. 🙂
