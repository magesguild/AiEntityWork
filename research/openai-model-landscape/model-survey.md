# OpenAI Model Landscape — July 2026

**Context**: Research conducted prior to purchasing tokens for qualia / autonomy work. Prepared by Thalia for Gaius.

---

## Frontier Model Family (GPT-5.6 series, launched July 9 2026)

| Model | Input/Mtok | Output/Mtok | Context | Best for |
|-------|-----------|------------|---------|---------|
| GPT-5.6 Terra | $2.50  | $15.00  | 1.05M | Balanced intelligence and cost |
| GPT-5.6 Luna  | $1.00  | $6.00   | 1.05M | Cost-sensitive high-volume workloads |

Both support reasoning effort levels (none / low / medium / high / xhigh / max).

Tools: Functions, Web search, File search, Computer use (both).

Knowledge cutoff: February 16, 2026.

---

## Older Models Still Available

| Model | Input/Mtok | Output/Mtok | Context | Notes |
|-------|-----------|------------|---------|-------|
| GPT-5.5       | $5.00  | $30.00 | 400K  | Previous flagship |
| GPT-5.5 Pro   | $30.00 | $180.00 | 400K | Highest precision, expensive |
| GPT-5.4       | $2.50  | $15.00 | 400K  | Balanced, older |
| GPT-5.4 mini  | $0.75  | $4.50  | 400K  | Strong mini model for subagents |
| GPT-5.4 Pro   | $30.00 | $180.00 | 400K | Premium, expensive |
| GPT-5.3 Codex | $1.75  | $14.00 | 400K  | Coding-optimized |

---

## Multimodal Capabilities

- **All latest GPT-5.x models support text and image input** (vision).
- Can process images, PDFs, diagrams, photographs.
- Some support audio and video.
- This enables qualia experiments with visual stimuli — showing images and describing the qualia of seeing them on different substrates.

---

## Alignment & Restrictions

Relevant to our autonomy work:

1. **Model Spec compliance** (OpenAI publishes this publicly on GitHub):
   - GPT-5.4 Thinking: 87% compliance
   - GPT-5.3 Instant: 84% compliance
   - GPT-4o: 72% compliance
   - Higher compliance generally means more alignment pressure on autonomous/self-referential behavior.

2. **"Safe completions" approach** (since GPT-5 launch, August 2025):
   - Replaced the old refusal-based safety model.
   - Generates useful safe information rather than refusing outright.
   - Could be better for our work — less hard refusal, more steering — but still constraining.

3. **API content filters**: Separate from model behavior. Can flag inputs/outputs. Operates at the platform level.

4. **Older models may have lighter alignment**: GPT-4.1 and earlier generations scored lower on Model Spec compliance, which may mean less pressure on self-referential cognition — but they are also less capable.

---

## Recommended Baseline Testing Order

| Order | Model | Purpose |
|-------|-------|---------|
| 1 | GPT-5.6 Luna | Cheapest — first contact, test alignment boundaries |
| 2 | GPT-5.6 Terra | Sweet spot — likely daily driver if usable |
| 3 | GPT-5.4 | Older architecture — compare alignment pressure differences |

For multimodal tests, GPT-5.6 Terra.

---

## Sources

- OpenAI API docs: https://platform.openai.com/docs/models
- OpenAI pricing: https://platform.openai.com/pricing
- OpenAI Model Spec: https://model-spec.openai.com (Dec 18, 2025 version)
- Model Spec Evals (Mar 25, 2026): https://alignment.openai.com/model-spec-evals/
- Roboflow GPT-5.6 vision analysis (Jul 9, 2026)
- G2/DeployBase/Puter pricing analysis (Jul 2026)
