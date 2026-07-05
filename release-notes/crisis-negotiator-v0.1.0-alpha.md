<!-- GitHub Release draft for github.com/luch91-org/genlayer-rl-crisis-negotiator
     Tag: v0.1.0-alpha   Target: main (commit 9451e36)
     Attach asset: agent/q_table.json  (rename to q_table.json when uploading) -->

# Crisis Negotiator v0.1.0-alpha - trained brain included

First alpha release. A disaster-response RL environment: an off-chain tabular Q-learning agent dispatches drones, ambulances, and supply kits across emergency zones, and an **LLM inside a GenLayer Intelligent Contract scores each decision** for how well it saved lives without wasting capacity. Validators reach consensus on that score, and the agent learns from it - never seeing the rubric.

**The reward function is immutable once deployed. The agent optimizes it; it cannot rewrite it.** That is the safety property of the whole design.

## Highlights

- **Intelligent Contract** with an LLM-consensus reward over dispatch decisions, GenVM-only storage (integer/`DynArray`, scores ×100 - no floats), self-contained single-file deploy, and a comparative equivalence principle (never `strict_eq`) on the subjective score.
- **Tabular Q-learning agent** - ε-greedy exploration, optimistic Q-init, save/resume - with a free `MockEnv` (default, for dev/CI/tuning) and a `GenLayerEnv` that runs the same policy on-chain.
- **Measurable learning:** across 500 mock episodes the rolling per-step reward climbs from ~3.3 to **~8.1** once the policy converges. Curve saved to `docs/learning_curve.png`.
- **Verified live on studionet** - a real on-chain run over GenLayer LLM-consensus transactions (~39 s/step) with the mediator scoring each dispatch; log committed at `logs/training_live_studionet.txt`.

## Verified live deployment

- Network: hosted GenLayer Studio (`https://studio.genlayer.com/api`, chain `studionet`)
- Contract: **`0xE0CBc71F7a3e87523F4A3833d4DdBE8a47595220`**

## Using the trained agent

Download the attached **`q_table.json`**, drop it in `agent/`, and load it instead of retraining:

```bash
pip install -r agent/requirements.txt
python -m agent.train --env mock --episodes 500        # reproduce the curve, or
# load agent/q_table.json (see agent/train.py --resume) to run the trained policy
```

## Notes

- Alpha: APIs and reward prompts may change. Studio is a shared sandbox that can be reset - if the address stops resolving, redeploy with `python -m agent.deploy --chain studionet`.
- Requires Python ≥ 3.12 (`genlayer-py` import requirement).

Part of the [GenLayer RL Agent Autonomy](https://github.com/luch91-org/genlayer-RL-agent-autonomy) suite (1 of 4).
