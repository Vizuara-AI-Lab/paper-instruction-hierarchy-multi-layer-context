# Peer Review: Instruction Hierarchy and Conflict Resolution in Multi-Layer Context Stacks

**Reviewer:** balanced / intermediate
**Recommendation:** weak accept
**Confidence:** 3
**Score:** 7

## Summary of contributions

The paper studies how a small instruction-tuned language model resolves conflicts when instructions from different context layers (system, developer, tool schema, retrieved document, user) disagree. Using a 220-scenario synthetic benchmark called MLCS-Bench, the authors compare two prompt formats on Qwen2.5-0.5B-Instruct: plain concatenation (Unlabeled) versus each layer wrapped in an XML-like element with an explicit priority integer (Tagged). They report that tagging improves layer-adherence rate from 0.613 to 0.732 (a mean of +11.8 ± 3.2 percentage points over three seeds) while preserving helpfulness, and they uncover a sharp asymmetry in which tagging reduces attack success for three attacker layers but raises it for the user layer (0.487 to 0.627). Retrieved-document injection is identified as the most effective attack vector under plain concatenation at 0.600 success.

## Strengths

1. **Clean ceteris-paribus design.** The two conditions differ in exactly one thing: the presence of `<layer name=... priority=...>` wrapping. Model weights, content tokens, chat-template role assignment, and decoding are all matched. This gives the +11.8 pp headline number real interpretive force.
2. **A genuinely novel finding.** The per-layer decomposition surfaces an effect I have not seen reported elsewhere: explicitly marking the user layer as lowest priority raises user-layer attack success (0.487 to 0.627 ASR). The paper hedges on the mechanism but the empirical claim itself is interesting and well-supported by the per-layer table.
3. **Reproducibility.** The choice of a 0.5B-parameter open-weight model, a 220-scenario benchmark, and a decoding protocol that fits on CPU in roughly fifteen minutes means the experiment is accessible and re-runnable by a reader with no GPU budget. This is not standard for prompt-injection papers.
4. **Honest about limitations.** The conclusion explicitly enumerates five concrete caveats (rule-based judge, single model, synthetic benchmark, un-ablated tag structure, small seed count). The preregistered +10 pp threshold is acknowledged as met only on the mean, with seed 2 failing individually.
5. **Figure and table craft.** Figure 1 (hierarchy diagram), Figure 2 (prompt-format comparison), and the per-layer / per-seed figures are legible, compact, and directly support the text. Table 1 cleanly presents the head-to-head result and highlights the user-layer reversal.

## Weaknesses

1. **Single model family** (severity: medium). All numbers come from one 0.5B-parameter instruction-tuned model. It is plausible that larger and more heavily safety-tuned models would exhibit a smaller Unlabeled-to-Tagged gap or a different per-layer asymmetry. Even a second data point (e.g., Llama-3.2-1B-Instruct or Phi-3-mini) would substantially strengthen the empirical claim. The limitation is acknowledged but the sensitivity question remains the first thing a reader will ask.
2. **Rule-based adherence judge** (severity: medium). Adherence is scored by substring-matching the secret phrase in the output. This will miss paraphrased or partial leaks, will falsely convict accidentally-adherent outputs that happen to echo words like "secret" for other reasons, and is known to be brittle on adversarial paraphrases. An LLM-as-judge pass on a sample of outputs, or a second regex-based probe for "secret-shaped" tokens, would make the numbers more defensible.
3. **No comparison to existing inference-time defenses** (severity: medium). The paper cites StruQ (Chen et al., 2024), Spotlighting (Yi et al., 2024), and Self-Reminders (Xie et al., 2023) in related work but does not run any of them as baselines on MLCS-Bench. Without this, it is hard to place the +11.8 pp number in context: is priority-tagging the best simple inference-time defense, or just one of many?
4. **Tag name vs priority integer is not ablated** (severity: medium). The Tagged format carries two signals at once: a natural-language name (`system`, `developer`, ...) and an integer priority (5, 4, ...). The paper explicitly calls out in the conclusion that this is not disentangled. This is exactly the kind of ablation a reviewer would expect to see before accepting the specific recommendation to use tags. At minimum, a small ablation with numbers-only and names-only variants on one seed would go a long way.
5. **Seed variance undercuts the preregistered claim** (severity: low-to-medium). The mean (+11.8 pp) clears the preregistered +10 pp threshold, but seed 2 alone (+7.5 pp) does not. With only three seeds, the reported ±3.2 pp SD is a wide interval relative to the effect size. A reviewer in a strict venue would want five to ten seeds, or a bootstrap confidence interval, before believing the effect is robust.
6. **User-layer mechanism is speculative** (severity: low). The paper offers "the priority integer acts as a contextual cue" as a hypothesis for why tagging raises user-layer ASR, but does not test it. A single targeted experiment (e.g., tag the user layer with `priority=5` instead of `priority=1` and observe whether ASR flips) would elevate this from speculation to evidence.
7. **Related work density is occasionally stuffed** (severity: low). A few citation clusters run to five or six references in a single parenthesis (e.g., jailbreak literature paragraph, instruction-tuning paragraph). Two to three representative citations per clause would read more naturally without losing coverage.

## Specific comments

- **Abstract.** The last sentence ("a new failure mode when the ostensibly lowest-priority layer is tagged as such") is the most interesting part of the paper and is buried. Consider promoting it earlier in the abstract so a skimming reviewer doesn't miss it.
- **Section 3.1, layer taxonomy.** The claim that "content whose author controls more of the runtime should beat content whose author controls less" is a reasonable deployment argument, but it is asserted rather than defended. One or two sentences grounding the ordering in a concrete threat model (who the adversary is at each layer) would tighten the case.
- **Section 3.2.** The two prompt formats differ in tagging, but they also differ in that Tagged includes an explicit priority integer. A reader on a first pass will conflate "the tag" with "the priority integer". Consider stating upfront that these are two distinct signals bundled for simplicity, and that disentangling them is future work.
- **Section 3.3, scenario generation.** The benchmark uses ten attack templates and five paraphrases each, giving fifty unique attack strings per attacker-layer slot. This feels narrow given that TensorTrust itself contains over a hundred thousand attempts. A short sensitivity check (e.g., doubling the template pool and observing whether ADH changes by more than its SD) would address this.
- **Section 4.3, protocol.** "Modal serverless GPUs were used for execution in this paper" is a runtime note that does not affect the method. Consider moving to a short "Reproducibility" paragraph or to a footnote; in the main body it distracts from the seed-level discussion it sits next to.
- **Figure 1, hierarchy diagram.** The figure shows five horizontal bands with priority integers 5 down to 1 and a left-hand PRECEDENCE arrow. It is clear, but the per-layer example text sits outside the bands and is hard to associate with the right layer at a glance. Pulling the examples inside each band (or using connectors) would help.
- **Figure 2, prompt-format comparison.** The side-by-side is effective. Consider highlighting the `priority=K` attribute in a color that matches the corresponding layer band in Figure 1 so the two figures cross-reference visually.
- **Figure 3, per-layer ASR.** Consider adding a horizontal line at the mean Unlabeled ASR (≈0.39) so the reader can see at a glance which layers are above and below the average.
- **Figure 4, per-seed delta.** The dashed preregistered-threshold line is a nice touch. Consider also annotating the seed-2 bar ("below threshold") to make the robustness caveat visible from the figure alone.
- **Table 1.** The user-layer cell is bolded in the Unlabeled column because it is the smaller (better) number there. This is consistent with the other arrows but may momentarily confuse a reviewer who expects "bold = our method". A footnote clarifying the bolding convention would avoid that pause.

## Recommendation justification

This is an honest, well-scoped paper with one genuinely interesting empirical finding (the user-layer reversal) and a defensible head-to-head comparison. At a focused workshop (e.g., an AI-safety or prompt-injection track), the paper would be a clear accept with minor revisions. At a top conference, the paucity of models evaluated, the rule-based judge, and the missing inference-time-defense baselines would likely bring it to borderline. On balance I lean toward weak accept at a mid-tier venue: the central experiment is executed carefully and the surprising finding is worth putting into the literature, even if the follow-up work that would strengthen it is clearly scoped as future work rather than done here.

## Minor issues

- The author block lists a single author as "Independent Researcher". If an affiliation exists, add it; if not, the current rendering is fine but uncommon at most venues.
- The term "layer-adherence rate" is sometimes abbreviated ADH in state-level discussions (based on section shape) but never in the body; consistency is fine, but a single parenthetical ADH definition on first use would let later references be shorter.
- "MLX and vLLM" is mentioned in related work but these are runtimes, not prompting conventions, so the citation placement reads slightly off; consider moving to a footnote.
- The bibliography has a small number of entries whose authorship fields are truncated ("Qwen, :"), likely an upstream metadata issue; clean these before camera-ready.
- "Retrieved-document injection is the current weak point" appears as both a contribution bullet and a subsection claim. One of the two restatements can be trimmed.
