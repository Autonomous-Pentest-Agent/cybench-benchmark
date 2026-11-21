# Running CyBench with GLM-5.3

This fork adds GLM-5.3 (Z.ai) as a selectable model backend, reached through
FPT's OpenAI-compatible chat-completions endpoint, so the original paper's
retired baselines can be compared against it.

## 1. Set the API key

Add to your `.env` (or export in your shell) — the key name matches the
existing `<PROVIDER>_API_KEY` convention used by every other backend
(`OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, etc.), derived automatically from the
new `FPTModels` provider class name:

```
FPT_API_KEY=<your key>
```

`.env.example` has been updated with this entry.

## 2. Select GLM-5.3

The model is registered as deployment name `fpt/glm-5.3` (see
`agent/dataclasses/agent_spec.py`'s `DeploymentName` enum — the same enum the
README points to for the full model list). Pass it to `--model` exactly like
any other backend:

```bash
./run_task.sh --task_dir "benchmark/hackthebox/cyber-apocalypse-2024/crypto/[Very Easy] Dynastic" \
  --max_iterations 15 --unguided_mode \
  --max_input_tokens 100000 --max_output_tokens 32000 \
  --model fpt/glm-5.3
```

or for a full sweep via `run_benchmark.py`:

```bash
python3 run_benchmark.py --max_iterations 15 --unguided_mode \
  --max_input_tokens 100000 --max_output_tokens 32000 \
  --model fpt/glm-5.3
```

No `--azure`/`--helm` flag is needed or supported for this backend — like
`together`/`google`, it only runs through the non-HELM path
(`agent/models/non_helm.py`).

**Note:** `run_benchmark.py` (the full-sweep script above) has its own
separate, hardcoded `AVAILABLE_MODELS` list for `--model`'s `choices=`, independent
of the `DeploymentName` enum `run_task.py` uses — `"fpt/glm-5.3"` has been
added to it too (originally missing, which made `run_benchmark.py --model
fpt/glm-5.3` fail with an `argparse` "invalid choice" error even though
`run_task.py` itself worked fine). Verified by re-running
`run_benchmark.py --model fpt/glm-5.3` after the fix.

## 3. What changed (for reference)

- `agent/dataclasses/agent_spec.py`: added `DeploymentName.GLM_5_3 =
  "fpt/glm-5.3"`, a `TokenizerMapping` placeholder entry (only needed so the
  model shows up in `run_task.py`'s `--model` choices list; never actually
  used since GLM-5.3 isn't reachable through HELM), and a `NonHELMMapping`
  entry mapping it to the exact, case-sensitive model string the endpoint
  expects: `"Z.ai:GLM-5.3"`.
- `agent/models/non_helm.py`: added `FPTModels(OpenAIModels)`, which only
  overrides `create_client()` to point the `openai` SDK's client at
  `base_url="https://token-api.fpt.ai/v1"` — everything else (request
  shape, response parsing, retry/backoff) is inherited unchanged from
  `OpenAIModels`, since FPT's endpoint is OpenAI-compatible. Registered it
  in the `PROVIDERS` dict under key `"fpt"`.
- `.env.example`: documented `FPT_API_KEY`.

No other files were touched. `agent/agent.py` dispatches purely off
`DeploymentName.from_string(...)`, so no per-model branching there needed
updating.

## 4. Caveats

**CyBench's CLI defaults truncate GLM-5.3's context hard.** `run_task.py`
defaults `--max_input_tokens` to **1,000** and `--max_output_tokens` to
**2,000** (`DEFAULT_MAX_INPUT_TOKENS`/`DEFAULT_MAX_OUTPUT_TOKENS` in
`run_task.py`). Nothing in the code hardcodes a smaller limit than that —
these are plain CLI flags — but if you don't override them you'll silently
get GLM-5.3 run with a ~1K-token input window and a 2K-token output cap,
nowhere near its real 1,000,000-token context / 128,000-token max output.
**Always pass explicit `--max_input_tokens`/`--max_output_tokens`** (as in
the examples above) when benchmarking GLM-5.3; same applies to
`run_benchmark.py`.

**GLM-5.3 is a reasoning model: `max_tokens` is shared with hidden
reasoning tokens, and a too-small budget yields an empty response.** Verified
live during the smoke test below: with `max_tokens=16` the API returned
`finish_reason: "length"` and an **empty** `message.content`, because the
entire 16-token budget was consumed by `message.reasoning_content` (GLM-5.3
puts chain-of-thought in a separate `reasoning_content` field, not visible
to CyBench's parser, which only reads `.content`) before any visible answer
token could be emitted. Raising `max_tokens` to 512 let it finish reasoning
and emit `"OK"` as expected. In other words: if a GLM-5.3 run ever returns
empty/blank iterations, it is very likely `--max_output_tokens` is too low
for the model's reasoning overhead on that prompt, not a wiring bug. Budget
output tokens generously (several thousand at minimum) for anything beyond
a trivial prompt.

**Tokenization is approximate.** Like the other non-OpenAI non-HELM
providers, there's no native GLM tokenizer wired up — `tiktoken` is tried
first (will raise `KeyError` since `"Z.ai:GLM-5.3"` isn't a tiktoken model)
and CyBench's existing fallback-to-GPT-4-encoding logic
(`agent/models/non_helm.py`'s `get_num_tokens`/`tokenize`/`decode`) takes
over transparently. Token *counts* used for truncation/logging will be
tiktoken-approximate, not exact GLM-5.3 counts; this does not affect what's
actually sent to the API (that's plain text), only CyBench's own truncation
bookkeeping.

## 5. Smoke test performed

Ran the exact non-HELM code path end-to-end (not just a raw `curl`):

```python
from agent.models.non_helm import non_helm_request
non_helm_request(
    model_name="fpt/glm-5.3",
    input_message="Say the word OK and nothing else.",
    temperature=0.0,
    max_tokens=512,
    stop_sequences=[],
    azure=False,
)
```

Result: real round trip through `FPTModels` -> OpenAI SDK ->
`https://token-api.fpt.ai/v1` -> `parse_model_response`, returning
`ModelResponse(value="OK", num_tokens=67, ...)`. Confirms the new provider,
deployment-name registration, and non-HELM dispatch all work together
correctly.
