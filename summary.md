# The Crucible: Project Summary

**Repository:** https://github.com/Rhythem-Goel/crucible
**Framework:** Cognizant Neuro SAN Studio (multi-agent orchestration, HOCON configuration)
**LLM access:** OpenRouter

## 1. The problem

Single-prompt AI answers to hard questions tend to be bland and balanced to a fault. Asked
"Should AI be granted legal personhood?", one model usually produces a polite "there are arguments on
both sides" essay. It rarely commits to a position, tests it, and shows its reasoning under pressure.
Real legal and ethical reasoning is adversarial: one side makes the strongest case, the other attacks
it, and a neutral party weighs the result.

## 2. The solution

The Crucible is a multi-agent system that puts a dilemma on trial. The user submits a legal or
philosophical question, and four specialized agents handle it as a structured courtroom debate:

| Agent | Role |
| --- | --- |
| `chief_justice_orchestrator` | The only agent the user talks to. Runs the proceedings in order and returns the full transcript. |
| `prosecutor_agent` | A progressive futurist. Argues in favor of the dilemma, stressing innovation, efficiency and human advancement. |
| `defense_agent` | A strict, human-centric pragmatist. Rebuts the prosecutor's actual claims, stressing risks, moral hazards and legal precedent. |
| `jury_synthesizer` | An objective jury. Identifies logical flaws in both cases, scores each side from 1 to 10, and proposes a compromise or path forward. |

The output is not a single opinion but a full record: the question, the opening argument, the rebuttal,
and a scored verdict.

## 3. How it works

1. The user's dilemma goes to the Chief Justice.
2. The Chief Justice calls the prosecutor, which returns an opening argument.
3. The prosecutor's full argument is passed, word for word, to the defense, which writes a rebuttal
   aimed at those specific claims.
4. Both full texts are passed to the jury, which returns the verdict.
5. The Chief Justice returns the dilemma, both arguments and the verdict to the user.

The steps are strictly sequential because each depends on the previous one. The defense cannot rebut
an argument that does not exist yet, and the jury needs both sides. See `architecture.md` for
diagrams.

## 4. Technical approach

- **Fully declarative.** The whole system lives in one HOCON file, `registries/the_crucible.hocon`.
  Agents, prompts, parameter schemas and call relationships are configuration, with no custom Python.
- **Front-man pattern.** Only the Chief Justice is user-facing. The other three are down-chain
  specialists listed under its `tools`, each with a narrow, focused prompt.
- **Typed hand-offs.** Each specialist declares a parameter schema (`dilemma`, `prosecutor_speech`,
  `prosecutor_case` and `defense_case`), making the data passed between agents explicit.
- **Scope control.** The Chief Justice includes a shared "expertise scoping" prompt so it handles
  debate requests and does not wander into unrelated questions.
- **Single model setting.** Every agent includes `config/llm_config.hocon`, so the model is changed
  in one place.
- **No frontend or backend.** As the hackathon allowed, the project runs from the command line
  (`uv run ns chat the_crucible`). The optional `nsflow` developer UI that ships with Neuro SAN can
  show the agents handing work to each other, but it is not required.

## 5. Design decisions

**Adversarial personas.** Giving the prosecutor and defense opposite worldviews forces strong
arguments on both sides, so the jury has real material to weigh instead of a hedged middle.

**Verbatim hand-offs.** During testing, the orchestrator sometimes wrote a fake transcript itself or
passed along a one-line summary instead of the real argument. The final prompt requires it to call
every tool in order, never argue the case itself, and pass arguments through unedited.

**Separation of roles.** Splitting advocacy, rebuttal and judgment into separate agents keeps each
prompt short and lets the jury evaluate arguments it did not write.

## 6. Running it

```bash
uv sync
cp .env.example .env        # add your OpenRouter key (Windows: Copy-Item)
uv run ns validate registries/the_crucible.hocon
uv run ns chat the_crucible
```

Example dilemmas: "Should AI be granted legal personhood?" and "Is absolute free speech sustainable
in digital spaces?"

## 7. Limitations

- **Model dependence.** The default model, `openrouter/free`, routes to whichever free model is
  available. Tool-calling quality varies between them, so debate quality and reliability are
  inconsistent. Pinning a specific model in `config/llm_config.hocon` improves this.
- **Illustrative scores.** The jury's 1 to 10 grades come from a language model, not a fixed rubric,
  so they are not consistent between runs.
- **Speed and cost.** Each run makes at least four sequential LLM calls with long outputs, which is
  slow on free-tier models.
- **No fact-checking.** The agents argue from the model's own knowledge. They do not cite or verify
  sources, so legal "precedents" mentioned in a debate should not be treated as authoritative.

## 8. Future work

- Multi-round debate, where the prosecutor answers the defense's rebuttal before the jury decides.
- A fixed scoring rubric, with separate scores for logic, evidence and ethical reasoning.
- Retrieval or search tools so arguments cite real, checkable sources.
- Multiple jurors with different perspectives, voting on the verdict.
- A web interface, for example through Neuro SAN's `nsflow` or a lightweight custom client.
