# The Crucible

A multi-agent simulation that stress-tests a hard question by putting it on trial.
Give it a legal or philosophical dilemma and four AI agents argue it out: one for, one against,
one to judge, and a Chief Justice to run the courtroom.

Built with Cognizant's [Neuro SAN Studio](https://github.com/cognizant-ai-lab/neuro-san-studio)
multi-agent framework. The whole system is declared in a single HOCON file, with no frontend or
backend code.

---

## How it works

```
                      user dilemma
                           |
                           v
              +-----------------------------+
              |  chief_justice_orchestrator |   (front man)
              +-----------------------------+
                 |1          |2          |3
                 v           v           v
          prosecutor_   defense_     jury_
          agent         agent        synthesizer
         (argues FOR)  (rebuts)     (scores + verdict)
```

| Agent | Role |
| --- | --- |
| `chief_justice_orchestrator` | Front man. Takes the dilemma, runs the three steps in order, and returns the full transcript plus verdict. |
| `prosecutor_agent` | Progressive futurist. Argues in favor of the dilemma: innovation, efficiency, human advancement. |
| `defense_agent` | Human-centric pragmatist. Attacks the prosecutor's claims: risks, moral hazards, legal precedent. |
| `jury_synthesizer` | Objective jury. Finds logical flaws in both cases, grades each side 1-10, and proposes a compromise or path forward. |

See [architecture.md](architecture.md) for the call sequence and design notes, and [summary.md](summary.md) for a project overview.

## Example dilemmas

- `Should AI be granted legal personhood?`
- `Is absolute free speech sustainable in digital spaces?`

## Prerequisites

- Python 3.12 or newer (Neuro SAN Studio requires it)
- An [OpenRouter API key](https://openrouter.ai/keys)

## Setup

```bash
git clone https://github.com/Rhythem-Goel/crucible.git
cd crucible

# create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate        # macOS / Linux
# .venv\Scripts\activate         # Windows

# install dependencies
pip install -r requirements.txt
```

Create your `.env` from the template and add your key:

```bash
# macOS / Linux
cp .env.example .env

# Windows (PowerShell)
Copy-Item .env.example .env
```

Then edit `.env`:

```
OPENROUTER_API_KEY=your-openrouter-api-key-here
```

The model is set in one place, `config/llm_config.hocon` (currently OpenRouter's `openrouter/free`).
To use a different model, change it there. Every agent picks it up.

## Run

Check that your key and config work:

```bash
ns check-llm-keys
ns check-config
ns validate registries/the_crucible.hocon
```

Chat with the network straight from the terminal (no server and no UI needed):

```bash
ns chat the_crucible
```

Run `ns chat --help` for other options, such as one-shot mode.

Optional: start the Neuro SAN server on its own (it listens on `localhost:8080`):

```bash
ns run --server-only
```

`ns run` without flags also launches the `nsflow` developer UI at http://localhost:4173/,
which is handy for watching the agents hand work to each other, but it is not required.

## Project structure

```
crucible/
├── config/
│   ├── llm_config.hocon     # Shared LLM settings (one place to change the model)
│   └── plugins.hocon        # Neuro SAN plugins (logging bridge, optional Phoenix tracing)
├── registries/
│   ├── manifest.hocon       # Tells the server which networks to serve
│   ├── the_crucible.hocon   # The four-agent network definition
│   └── expertise_scoping_instructions.hocon  # Shared prompt that keeps the Chief Justice on topic
├── .env.example             # Template for your API key
├── requirements.txt         # Python dependencies
├── architecture.md          # Call sequence and design notes
├── summary.md               # Project summary
└── README.md                # This file
```

## Troubleshooting

**`OPENROUTER_API_KEY` not found.** Make sure `.env` exists in the project root and contains the key.
Run `ns check-llm-keys` to verify.

**`No module named 'langchain_openrouter'` (shown in the chat or logs).** The OpenRouter integration
is missing from your environment. Run `pip install -r requirements.txt` again, or
`pip install langchain-openrouter`.

**`cannot import name 'eager_task_factory' from 'asyncio'`.** You are on Python 3.11 or older.
Use Python 3.12+.

**`UnicodeEncodeError: 'charmap' codec can't encode character` in the logs (Windows).**
A logging message contains an emoji your console encoding cannot print. It is harmless, and
setting `PYTHONUTF8=1` before running removes it.

**Odd or inconsistent debates.** `openrouter/free` routes to whichever free model is available,
and their tool-calling quality varies. For more reliable runs, set a specific model in
`config/llm_config.hocon`.

## Acknowledgements

Built on [Neuro SAN](https://github.com/cognizant-ai-lab/neuro-san) and
[Neuro SAN Studio](https://github.com/cognizant-ai-lab/neuro-san-studio), both Apache-2.0 licensed.
