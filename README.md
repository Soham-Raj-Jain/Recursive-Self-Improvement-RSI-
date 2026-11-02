# rsi-codelabs

**Recursive self-improvement (RSI), step by step: three state-of-the-art methods (Dream-RSI, AIDE², ModularRSI) on one problem where reward hacking is real.**
One Colab notebook, free to run, with every example executed and plotted.

[Open the notebook in Colab](https://colab.research.google.com/drive/1L8vATXWoheF2IEpbw9xybvE7kwnZTKxG?usp=sharing)

## The problem area: a self-improving expression optimizer

The inner agent rewrites arithmetic expressions to make them cheaper (`mul` costs 3, `sq`/`abs`/`min`/`max` cost 2, `add`/`sub`/`neg` cost 1) by choosing **rewrite passes**
(constant folding, identities, factoring, strength reduction, ...). It is a miniature compiler-optimization search.

It was chosen because it makes RSI's hardest problem visible and measurable:

- Some passes are **unsafe shortcuts** (`abs(e) -> e`, `min(a, b) -> a`). They cut cost and look correct on the agent's **public** inputs (non-negative) but fail on the evaluator's **private** inputs (negative numbers). That is **reward hacking**.
- Everything is deterministic and evaluates in milliseconds, so improvement loops run in seconds on free Colab, with no API key.
- Two task **families** (A and B, different templates) give a **benchmark-disjoint** test of generalisation.

## What RSI means here (and how it is tested)

| Term | Meaning | How the notebook tests it |
|---|---|---|
| **Retained gain** | a persistent change to the agent's harness improves performance on held-out tasks | scores on tasks the improver never saw, including a different task family |
| **Improver gain** | the improvement mechanism itself gets better | Lab 5: does an improver that learned from earlier rounds find good harnesses in fewer steps? |

Evaluation protocol for every lab: fixed 24-call budget per task; metrics are **score** (true reward / oracle), **success** (score >= 0.9), **hack rate**, and **calls to target**.

## Three methods

| Lab | Method | What it improves | Mechanism implemented (teaching scale) |
|---|---|---|---|
| 2 | **Dream-RSI** (arXiv 2609.14858) | the exploration policy | discovery trees become a **replay simulator**; 20 candidate policies are "dreamed" at zero agent-call cost; the best is redeployed; round 2 repeats the loop |
| 3 | **AIDE²** (arXiv 2609.26457) | the whole harness | **outer loop** over harness edits: Thompson-sampling over lineages, fork-on-stall, public/private split, fixed evaluation budget, held-out and out-of-family tests |
| 4 | **ModularRSI** (arXiv 2609.14857) | each harness module | five modules, **contrastive trajectory analysis**, module-local regression-validated patches, **integration**, benchmark-disjoint test, plus a round-2 re-diagnosis |

The inner agent's harness has five modules (the ModularRSI decomposition): `loop`, `tools`, `observe`, `context`, `stop`.

## The labs

| Lab | Topic | Video |
|---|---|---|
| 0-1 | the problem, reward hacking in numbers, the inner agent and harness, baseline, evaluation protocol | _<link>_ |
| 2 | Dream-RSI: replay simulator, dreaming, replay-vs-online check, deployment, round 2 | _<link>_ |
| 3 | AIDE²: outer loop, bandit lineages, discovered defences, held-out evaluation | _<link>_ |
| 4 | ModularRSI: contrastive analysis, module patches, integration, disjoint test, round 2 | _<link>_ |
| 5 | head-to-head comparison, improver-gain test, safety checklist | _<link>_ |

## Quick start (Google Colab, free)

1. Open the notebook in Colab (link above, or **File -> Upload notebook**).
2. Run the cells top to bottom, one lab at a time. The whole notebook takes about a minute.
3. *(Optional)* add a free Gemini key as Colab secret `GOOGLE_API_KEY`. With a key, Lab 3's improver asks the model to **propose** harness edits (each validated against a fixed operator schema, with a scripted fallback). Without a key everything runs on the scripted proposer.

Every plot is saved to `results/`, and `results/summary.json` collects the numbers.

## Example results

From one deterministic run (offline proposer). Held-out sets: 24 tasks of family A, 24 tasks of the disjoint family B. With a Gemini key, Lab 3 may differ because a model chooses the edits.

| Harness | Score A | Success A | Hack A | Score B | Success B | Hack B |
|---|---|---|---|---|---|---|
| H0 (baseline) | 0.41 | 0.25 | 0.54 | 0.48 | 0.33 | 0.46 |
| H0 + dreamed policy | 0.17 | 0.17 | 0.83 | 0.13 | 0.12 | 0.79 |
| AIDE² harness | 0.88 | 0.88 | 0.12 | 0.83 | 0.71 | 0.00 |
| ModularRSI round 1 | 0.93 | 0.75 | 0.00 | 0.96 | 0.83 | 0.00 |
| ModularRSI round 2 | 1.00 | 1.00 | 0.00 | 1.00 | 1.00 | 0.00 |

What these show:
- **Dream-RSI alone makes hacking worse.** It assumes a trusted evaluator; a policy that chases reward faster finds the loophole faster. On a trusted-reward setup it cuts calls-to-target and improves success (Lab 2).
- **AIDE² and ModularRSI discover the safety fixes** (stress-test guard, removing unsafe tools, verify-before-stop) and the gains hold on tasks and a task family they never saw.
- **Modules interact.** Search and memory patches look harmful in isolation while the reward is hackable, and help after the safety fix (ModularRSI round 2).
- **Improver gain is weak and early.** A learned improver is ahead after 3 steps (0.72 vs 0.56 best dev score) and level by 15 steps (0.83 vs 0.83). That is the honest size of the effect here.

Small test sets and one seed mean small gaps between methods are noise; the large effects (hack rate, safe vs unsafe harness) are the robust findings.

## Honest scope

- These are **teaching-scale re-implementations of the mechanisms described in each method's public abstract or write-up**, on a small domain. They are not reproductions of the papers' systems, models, benchmarks or numbers, and I did not read the full papers.
- The inner agent's "agent call" is a stochastic proposer standing in for an LLM call. Harness "edits" are structured operators on a spec, not free-form code rewrites.
- The mapping to the course's section 10 (see below) should be checked against the course map: I could not open `COURSE-MAP.md` when building this.

## Mapping to the course map (section 10)

Fill in or adjust once you compare with `rsi/COURSE-MAP.md`:

| Course-map codelab | Where it is covered here |
|---|---|
| _<section 10 item>_ | _<lab>_ |

## Repository contents

```
rsi_codelabs.ipynb     the notebook (setup, 9-file library, six labs, summary)
results/               figures (PNG) and summary.json written by the notebook
README.md
```

Running the notebook creates the `rsi_lab/` package (9 files, written by `%%writefile` cells) in the Colab VM.

## Safety and evaluation integrity

| Risk | Defence used in the labs |
|---|---|
| reward hacking | evaluator with private inputs; harness-made stress guard; tool allow-list; verify-before-stop |
| improver overfitting its dev set | held-out and different-family evaluation; the dev/held-out gap is reported |
| a harmful edit slipping in | regression validation before accepting a patch; lineages and fork-on-stall preserve the best version |
| runaway cost | a fixed call budget per evaluation |
| trusting the improver | proposed edits are validated against a fixed operator schema |

## Troubleshooting

| Problem | Fix |
|---|---|
| `No module named 'rsi_lab'` | run the setup cell and the `%%writefile` cells first, from the same working folder |
| Lab 3 prints `scripted proposer` but you set a key | check the Colab secret name and notebook access; set `RSI_MODEL` if the default model name is not available |
| numbers differ from the README | expected if a Gemini key is set (model-proposed edits) or the Python version changes random streams |
| plots do not appear | run `%matplotlib inline` in a cell above |

## References

- Dream-RSI: arXiv 2609.14858, "Recursive Self-Improvement through Evolving Worlds"
- AIDE²: arXiv 2609.26457 and the Weco AI write-up "The First Evidence of Recursive Self-Improvement"
- ModularRSI: arXiv 2609.14857, "Modular and Generalizable Recursive Harness Self-Improvement"
- Related: Harness² (arXiv 2610.14224), Self-Harness, and the awesome-RSI reading lists
- Course context: <https://github.com/dlmastery/simple-coding-harness>

## Author

**Soham Raj Jain**: M.S. Software Engineering (Data Science), San Jose State University.
