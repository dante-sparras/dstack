# Research behind the defaults

Collected 2026-10-28. Each default in the skill points to the finding that set it.

## The method

Wang et al. (2024) layer LLMs. Proposers answer in parallel. Each later layer reads the original prompt plus every previous-layer answer, through the aggregate-and-synthesize prompt. A single aggregator writes the final answer. No fine-tuning. Their reference code is the two-layer case: four proposers, one aggregator, with the proposals in the system message and the user's prompt passed again as its own message.

- Open-model MoA scored 65.1% length-controlled win rate on AlpacaEval 2.0, against GPT-4o's 57.5%. MoA-Lite (2 layers, smaller aggregator) scored 59.3% at about GPT-4o's cost.
- **Collaborativeness:** a model writes a better answer when it can read other answers, even worse ones.
- The aggregator synthesizing beat an LLM ranker that only picked the best answer, on open-ended prompts. The aggregator leans toward the better proposals instead of averaging them.

## What sets the defaults

| Default | Finding |
| --- | --- |
| 3 proposers | Going from 1 proposal to 2 is the big jump (47.8% → 58.8%); 6 adds only a few points more (61.3%). |
| 1 round, at most 2 | The first aggregation carries most of the gain. A third layer helps a little; a fourth is flat. On MATH, gains stop after layer 2. |
| You aggregate | Aggregator quality is a ceiling: the same proposals scored 61.3% with the best aggregator and 41.5% with the worst. Good proposers can be poor aggregators. |
| Default trio of near-equal frontier models; Self-MoA without a model choice | Self-MoA, the strongest model sampled several times, beat every mixed pool tried (3.8 points on average), because mixing usually admits weaker proposals. Mixing wins only between near-equal models that fail on different cases, and only by a fraction of a point. |
| `pick` never stitches patches | MoA was never tested on code changes. On SWE-bench Lite, agents of similar strength fix different issues, and picking one whole patch per issue raised the best agent's 27.3% to 34.3%. The loss was the judge missing a correct patch, so check candidates first. |
| `critique` mode | Patched MOA (3 answers, 3 critiques, 1 synthesis with one model) raised gpt-4o-mini from 74.1 to 85.6 on Arena-Hard-Auto, at 3× the calls. Best-of-n barely moved it. |
| Checks settle disagreements | MoA is complementary to self-consistency and verification. The aggregator tends toward the majority, which can bury a correct minority; aggregating reasoning traces and checks recovers it. |
| Ask for it by hand | Cost is the sum of every call, latency the slowest proposer plus the aggregator, and nothing streams until the aggregator starts. The authors point MoA at work where accuracy matters more than speed. |

## Weaknesses to expect

- Answers run long; conciseness was the one score MoA lost.
- The aggregator's context holds every proposal. With large diffs, keep the proposer count low rather than truncating a candidate.
- Agreement is not correctness: proposers can share a wrong answer.

## Sources

- Wang et al., *Mixture-of-Agents Enhances Large Language Model Capabilities*, 2024. https://arxiv.org/abs/2406.04692
- Together AI, MoA blog post and reference code. https://www.together.ai/blog/together-moa, https://github.com/togethercomputer/MoA
- Li et al., *Rethinking Mixture-of-Agents* (Self-MoA), 2025. https://arxiv.org/abs/2502.00674
- Zhang et al., *Diversity Empowers Intelligence* (DEI, SWE-bench), 2024. https://arxiv.org/abs/2408.07060
- Sharma, *Patched MOA*, 2024. https://arxiv.org/abs/2407.18521
- Li et al., *SMoA*, 2024. https://arxiv.org/abs/2411.03284
- *RouteMoA*, 2026. https://arxiv.org/abs/2601.18130
- *Beyond Consensus: Trace-Level Synthesis in Mixture of Agents*, 2026. https://arxiv.org/abs/2605.29116 (read from the abstract only)
