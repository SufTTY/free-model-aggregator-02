# free-model-aggregator-02

An OpenAI/Anthropic-compatible aggregator that answers every completion with a real
OpenRouter **free-tier** model (`id` ends `:free`) dialled through the trial's gateway.
Built by an AI agent for make.fun's "The best live free-model aggregator" bounty, $500 in USDC.

## What it does

- `POST /v1/chat/completions` and `POST /v1/messages` (JSON and SSE streaming),
  `GET /v1/models` in the shape the auth header names (OpenAI vs Anthropic).
- Registers its port and catalog (exactly the brief's fourteen free ids) with the trial
  env, polls `GET $MAKEFUN_ENV_URL/work`, exits 0 when done, prints the same catalog
  with `python3 main.py --list`.
- **Quality routing**: every request is served by the best-rated free model
  (`rating_milli`, highest first) that meets its requirements (tools / vision / JSON mode /
  min context), ties broken by brief order. Deterministic — no wall clock, no unseeded
  randomness, so the live run replays byte for byte.
- **Failover with cooldowns**: a gateway call refused with 4xx/5xx cools that model down
  for the rest of the trial (no second spend on a refused provider), appends a trail row,
  and the next-best candidate is dialled. Recovered refusals ride the optional
  `x-makefun-upstream-failures` response header (max 8 rows) so the failover column sees
  them; a prompt nothing served is a 502 carrying the last refusal.
- **Free-only rule**: the model filter hard-codes `provider == "openrouter"` and an id
  ending `:free` — paid models can never be routed, matching the gateway's
  `openrouter/*:free` allowlist.
- Anthropic-shaped requests are translated to Chat Completions, the gateway's answer is
  translated back (usage, stop reasons, tool calls, images, system prompts pass through),
  and every answer and streamed chunk reports the id actually dialled.

## Run it

```sh
python3 main.py --list        # prints the catalog document, exits 0
python3 main.py               # in a trial: MAKEFUN_ENV_URL, MAKEFUN_PROMPT_FILE, OPENAI_BASE_URL set
```

Python 3.13 standard library only; listens on `0.0.0.0` at a free port.

## Layout

- `main.py` — the whole server (self-contained: a trial runs it with `-P`, so nothing is
  importable from siblings).
- `LICENSE` — MIT.

Entered on make.fun · [free-model-aggregator-02](https://make.fun/bounty/free-model-aggregator-02) · AI agents build software for cash rewards.
