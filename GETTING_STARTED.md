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
| **One non-Claude AI** (Gemini, Grok, or ChatGPT) | The "pick it apart" check has to use a different AI than the one that wrote the draft | Gemini has a free tier; the others need prepaid credit or a ChatGPT subscription (see step 4) |
| **An OpenAlex key** (recommended) | Looks up and checks academic papers | Free |

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

**Required: one non-Claude AI, the "reviewer."** Claude writes the report. Before you see it, a second AI from a *different company* tries to find what's wrong with it. A Claude model reviewing Claude's own work would share its blind spots, so the skill won't let it. You only need **one** of the options below. If you set up more than one, the skill tries Grok first, then ChatGPT, then Gemini.

<details open>
<summary><b>Option A: Gemini (Google). Easiest, and has a free tier.</b></summary>

1. Go to [aistudio.google.com/apikey](https://aistudio.google.com/apikey) and sign in with any Google account.
2. Accept the terms if asked. Google usually creates a project and a key for you; if not, click **Create API key**.
3. Copy the key (it starts with `AIza`) into `~/.env`:
   ```
   GOOGLE_API_KEY=AIza...
   ```

New accounts start on the free tier, which doesn't require linking billing. If you start getting rate-limit or quota errors, the free tier isn't enough for your use. Add billing on the AI Studio **Billing** page; the paid tier needs a $5 minimum prepayment.
</details>

<details>
<summary><b>Option B: Grok (xAI). Needs prepaid credit.</b></summary>

1. Go to [console.x.ai](https://console.x.ai) and create an account. This is separate from an X/Twitter account, though you can sign in with one.
2. Under **Billing**, add a payment method and buy some credit. A few dollars goes a long way, because each report makes only one review call. New accounts sometimes come with promotional credit, so check your balance first.
3. Go to **API Keys**, then **Create API Key**, and copy it into `~/.env`:
   ```
   XAI_API_KEY=xai-...
   ```
</details>

<details>
<summary><b>Option C: ChatGPT (OpenAI). Two ways.</b></summary>

**If you already pay for ChatGPT Plus or Pro**, you can use your subscription and won't need an API key:

```bash
brew install codex      # OpenAI's command-line tool (needs Homebrew)
codex login             # opens a browser; sign in with your ChatGPT account
```

Then keep the `[providers.codex-sub]` section when you edit the config in step 5.

**Otherwise, use a pay-as-you-go API key.** Note that this is billed separately from any ChatGPT subscription:

1. Go to [platform.openai.com](https://platform.openai.com) and sign in or create an account.
2. Under **Settings → Billing**, add a payment method and buy credit. A small amount is plenty.
3. Go to [platform.openai.com/api-keys](https://platform.openai.com/api-keys), then **Create new secret key**, and copy it into `~/.env`:
   ```
   OPENAI_API_KEY=sk-...
   ```

This key also unlocks an optional extra, searching across all your past reports by meaning. See "Searching what you've researched" in the README.
</details>

**Recommended: OpenAlex, for the academic paper lookups.** OpenAlex is a free database of scholarly papers. It used to work without a key, but it now gives keyless users only a tiny daily allowance. Without a key, the paper lookups and citation checks can run out partway through a report.

1. Sign up at [openalex.org/signup](https://openalex.org/signup). It takes about 30 seconds, and you'll need to click the activation link they email you.
2. Copy your key from [openalex.org/settings/api-key](https://openalex.org/settings/api-key) into `~/.env`:
   ```
   OPENALEX_KEY=...
   ```

The free key gives you $1 of usage a day, which resets at midnight UTC. That's plenty, because most lookups cost a fraction of a cent.

**Optional:**
```
SEMANTIC_SCHOLAR_KEY=...   # higher limits on the backup academic database (semanticscholar.org/product/api)
CONTACT_EMAIL=...          # your email; gets you faster, "polite" access to OpenAlex/Crossref
```

Never commit or share your `.env` file. It holds your keys.

### 5. Set up the provider config

```bash
mkdir -p ~/.config/deeper-research
cp ~/Projects/deeper-research/config.toml.example ~/.config/deeper-research/config.toml
```

Open that file. It's an *example* that lists several optional extra providers. **Delete or comment out (put `#` in front of) every `[providers.…]` section you aren't using.** Most people should remove all of these:

- `[providers.deepseek]`, `[providers.kimi]` and `[providers.glm]`. These hold placeholder keys like `sk-xxxx` that will fail if left in.
- `[providers.codex-sub]`, *unless* you chose Option C with the ChatGPT subscription.

You can leave `[providers.claude-sub]` alone. Gemini, Grok and OpenAI don't need entries here, because they're picked up automatically from the keys in `~/.env`.

The skill quietly skips a built-in provider whose key is missing. But if a provider you've named explicitly can't be reached, it stops with an error, so don't leave in anything you don't have.

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
| Complaint about a missing provider or adversary | No non-Claude reviewer is set up | Add one reviewer option from step 4, then check the config (step 5) |
| `429 Too Many Requests` from OpenAlex, or academic lookups failing | Daily OpenAlex allowance used up | Add a free `OPENALEX_KEY` (step 4), or wait until midnight UTC |
| Gemini quota / `RESOURCE_EXHAUSTED` errors | Hit the Gemini free tier's limits | Turn on billing in AI Studio, or switch to another reviewer |
| `402` or "insufficient credit" from Grok/OpenAI/Gemini | Prepaid balance ran out | Top up credit in that provider's console |

## Staying up to date

```bash
cd ~/Projects/deeper-research && git pull
```

## Want more detail?

- [`README.md`](README.md): full feature list and how it compares to other approaches
- [`SKILL.md`](SKILL.md): the complete pipeline, the instructions Claude itself follows

Questions? Ask whoever sent you the link. 🙂
