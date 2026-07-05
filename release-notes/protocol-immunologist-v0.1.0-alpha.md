<!-- GitHub Release draft for github.com/luch91-org/genlayer-rl-protocol-immunologist
     Tag: v0.1.0-alpha   Target: main (commit aad6ad8)
     Attach asset: agent/q_table.json  (rename to q_table.json when uploading) -->

# Protocol Immunologist v0.1.0-alpha - trained brain included

First alpha release. A DAO treasury-defense RL environment: an off-chain tabular Q-learning agent decides when to **pause, rotate signers, or hedge** as threat signals move, and an **LLM inside a GenLayer Intelligent Contract judges** whether the action preserved capital *and* was proportionate - decisive during a real red alert, restrained during noise. Validators reach consensus on that judgment; the agent learns from it without seeing the rubric.

**The reward function is immutable once deployed. The agent optimizes it; it cannot rewrite it.**

## Highlights

- **Intelligent Contract** with an LLM-consensus reward that rewards protecting the treasury when a threat is genuinely trending and penalizes over-reacting to noise. GenVM-only storage (no floats; scores ×100), self-contained single-file deploy, comparative equivalence principle on the score.
- **Tabular Q-learning agent** - ε-greedy, optimistic Q-init, save/resume - with a free `MockEnv` default and a `GenLayerEnv` for the on-chain demo. The state encodes the threat regime so the optimal action is genuinely state-dependent (act under red alert; hold under green).
- **Measurable learning:** 500 mock episodes climb from ~4 to **~8** per-step once converged. Curve saved to `docs/learning_curve.png`.
- **Verified live on studionet** - a real multi-episode on-chain run (~706 s over LLM-consensus transactions, 10 states seen) with the contract correctly rewarding "paused during a red alert"; log at `logs/training_live_studionet.txt`.

## Verified live deployment

- Network: hosted GenLayer Studio (`https://studio.genlayer.com/api`, chain `studionet`)
- Contract: **`0x4213C3915a314B7A4ef926895A08638F54aE55dd`**

## Using the trained agent

Download the attached **`q_table.json`**, place it in `agent/`, and load it instead of retraining:

```bash
pip install -r agent/requirements.txt
python -m agent.train --env mock --episodes 500        # reproduce the curve
```

## Notes

- Alpha: reward prompts and APIs may change. Studio is a shared sandbox that can be reset - redeploy with `python -m agent.deploy --chain studionet` if the address stops resolving.
- Requires Python ≥ 3.12.

Part of the [GenLayer RL Agent Autonomy](https://github.com/luch91-org/genlayer-RL-agent-autonomy) suite (2 of 4).
