---
name: laya
description: Make typed decisions (choice, score, noul) over text, email, tickets or JSON with the Laya non-autoregressive decision engine, in one forward pass, instead of TypeSafe Jev. Use when the user asks to classify, triage, route, score urgency, or answer yes/no questions about a piece of text in a fast, calibrated way, or to port a Jev `/v1/systemone` integration to Laya.
---

# Laya: typed decisions without generation

Laya answers typed questions about a state (text, email, ticket, or JSON) in a single
forward pass. It does not generate text, so there is nothing to parse. It is the
alternative to TypeSafe Jev, and its HTTP mode speaks the Jev `/v1/systemone` protocol.

## Setup

```bash
python -m pip install laya            # Python 3.10+
python -m pip install 'laya[structured]'   # optional: pydantic schemas via decide()
```

The first call downloads a checkpoint from Hugging Face. Use `Router(preload=True)` to load all
three up front. If the environment blocks Hugging Face, say so rather than guessing answers.

## Question types

| type | answer key | use for |
|---|---|---|
| `choice` | `choice` (argmax), `probabilities` | pick one of named options; `criteria` maps option to description |
| `score` | `score` (expected level index) | ordinal level; `criteria` is an ordered list of level texts |
| `noul` | `noul` (probability the answer is yes) | yes/no questions |

## Usage

```python
from laya import Router

router = Router()
state = "Hi, we were billed twice for March. Refund the duplicate today or we cancel."
questions = {
    "department": {"type": "choice", "instructions": "Which department should handle this?",
                   "criteria": {"billing": "invoices, payments, refunds",
                                "technical": "bugs, outages, system errors",
                                "other": "everything else"}},
    "urgency": {"type": "score", "instructions": "How urgent is this?",
                "criteria": ["not urgent", "soon", "blocking"]},
    "churn_risk": {"type": "noul", "instructions": "Does the user threaten to cancel or leave?"},
}
result = router.predict(state, questions)
result["answers"]["department"]["choice"]   # e.g. "billing"
result["answers"]["churn_risk"]["noul"]     # probability of yes
result["routing"]["model"]                  # which checkpoint answered
```

- `Router` detects language and routes non-English text to `laya-multilingual`.
- Long documents: pass `model="multilingual", max_len=8192` to `predict`. Accuracy on long
  text varies, so check it on the user's own data.
- Batches: `router.predict_batch(states, questions)`.
- CLI: `laya "My payment failed twice" --preset triage`.
- Schema-driven output: `laya.structured.decide` with a pydantic model.

## Serving over HTTP (Jev-compatible)

```bash
python -m pip install 'laya[serve]'
laya-serve            # POST /v1/systemone, same request and response shape as Jev
```

Point a Jev client's `baseUrl` at it. Set `LAYA_JEV_STRICT=1` if the client validates
responses against the strict Jev contract.

## Migrating from TypeSafe Jev: confidence conflict

Laya's `confidence` is normalized entropy. Jev's `confidence` is a different formula. A
threshold carried over from Jev gates about twice as much on Laya's `confidence`. When porting,
compare the Jev threshold against `x_jev_confidence` (added on `choice` and `score` answers by
`laya-serve`), not `confidence`. Never mix the two in one threshold.

## Limits to state to the user

- Accuracy is checkpoint- and domain-dependent. The shipped checkpoints are zero-shot.
  Fine-tuning on the user's own decisions improves accuracy (see `docs/finetune.md`).
- Do not use Laya output as the sole basis for consequential decisions without a review step.
