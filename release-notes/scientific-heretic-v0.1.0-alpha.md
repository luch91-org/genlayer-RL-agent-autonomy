<!-- GitHub Release draft for github.com/luch91-org/genlayer-rl-scientific-heretic
     Tag: v0.1.0-alpha   Target: main (commit b753769)
     Attach asset: agent/q_table.json  (rename to q_table.json when uploading) -->

# Scientific Heretic v0.1.0-alpha - trained brain included

First alpha release. A hypothesis-generation RL environment: an off-chain tabular Q-learning agent plays a researcher that **proposes hypotheses and tests them**, and an **LLM peer-reviewer inside a GenLayer Intelligent Contract scores each proposal 0-10** on falsifiability, novelty, and plausibility, with validators reaching consensus. Testing a stored high-merit, falsifiable hypothesis validates it for a bonus. The agent learns the shape of a review-worthy idea from grades alone.

**The reward function is immutable once deployed. The agent optimizes it; it cannot rewrite it.**

## Highlights

- **Intelligent Contract** storing hypotheses + merits, with an LLM peer-review reward and a **deterministic** validation step. GenVM-only storage (no floats; scores ×100), self-contained single-file deploy, comparative equivalence principle on the review score.
- **Tabular Q-learning agent** - ε-greedy, optimistic Q-init, save/resume - with a free `MockEnv` default and a `GenLayerEnv` for the chain. Score-aware state so the agent learns to propose good hypotheses *and* test the falsifiable ones.
- **Measurable learning:** 500 mock episodes climb from ~3.7 to **~8.8** per-step once converged. Curve saved to `docs/learning_curve.png`.
- **Verified live on studionet** - a full trained-agent episode, 4 on-chain steps, greedy policy proposing two hypotheses and validating each, **per-step average 8.00**; log at `logs/training_live_studionet.txt`.

### Consensus lesson baked in

The `test` action was originally LLM-scored ("was this experiment worthwhile?"). On the shared testnet that subjective prompt spread validator scores past the equivalence tolerance and intermittently returned **`NO_MAJORITY`** - the transaction never landed. Making `test` **deterministic** (its payoff fixed by the already-agreed merit) is both more spec-faithful and consensus-robust. Live testing caught what offline testing structurally could not.

## Verified live deployment

- Network: hosted GenLayer Studio (`https://studio.genlayer.com/api`, chain `studionet`)
- Contract: **`0xDd169FA2FA5D258f1CCBc8CAe61eA652733435F6`**

## Using the trained agent

Download the attached **`q_table.json`**, place it in `agent/`, and load it instead of retraining:

```bash
pip install -r agent/requirements.txt
python -m agent.train --env mock --episodes 500        # reproduce the curve
```

## Notes

- Alpha: reward prompts and APIs may change. Studio is a shared sandbox that can be reset - redeploy with `python -m agent.deploy --chain studionet` if the address stops resolving.
- Requires Python ≥ 3.12.

Part of the [GenLayer RL Agent Autonomy](https://github.com/luch91-org/genlayer-RL-agent-autonomy) suite (3 of 4).
