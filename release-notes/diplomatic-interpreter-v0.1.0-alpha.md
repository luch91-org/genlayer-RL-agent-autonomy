<!-- GitHub Release draft for github.com/luch91-org/genlayer-rl-diplomatic-interpreter
     Tag: v0.1.0-alpha   Target: main (commit e6c5ce6)
     Attach asset: agent/q_table.json  (rename to q_table.json when uploading) -->

# Diplomatic Interpreter v0.1.0-alpha - trained brain included

First alpha release. A cross-community mediation RL environment: an off-chain tabular Q-learning agent drafts compromises in a fixed dispute (a riverside parcel - public park vs. housing), and an **LLM mediator inside a GenLayer Intelligent Contract estimates, on one anchored 0-10 scale, how likely both communities are to accept** the draft. Validators reach consensus on that number; it is the reward, and a single **polarization index** then cools **deterministically** from it. The agent learns diplomatic tact - and to match the move to the temperature of the room - without seeing the rubric.

**The reward function is immutable once deployed. The agent optimizes it; it cannot rewrite it.**

## Highlights

- **Intelligent Contract** whose single acceptance number folds *fairness* and *fit-to-the-moment* together (kept to one number so validators stay clustered), with a deterministic polarization update. GenVM-only storage (polarization 0-100 integer, scores ×100, no floats), self-contained single-file deploy, comparative equivalence principle on the score.
- **Tabular Q-learning agent** - ε-greedy, optimistic Q-init, save/resume, **random-start training** so every polarization bucket is learned on-policy. State = polarization bucket; the optimal action is genuinely state-dependent: a **detailed** compromise while the dispute is hot, a **concise** consolidation once it has cooled.
- **Measurable learning:** 500 mock episodes climb from ~3.5 to **~8.8** per-step once converged. Curve saved to `docs/learning_curve.png`.
- **Verified live on studionet** - a full trained-agent episode, 5 on-chain LLM-consensus steps, **per-step average 7.60**, with the dispute **cooling monotonically from 0.80 to 0.23** (a 57% drop) and **no `NO_MAJORITY`**. The state-dependence is real on-chain: the concise consolidation scores ~6 while polarization is high (dismissive of live grievances) and ~8 once settled. Log at `logs/training_live_studionet.txt`.

### Two bugs live testing caught (and offline testing structurally could not)

1. The mock-optimal path skipped the `medium` polarization bucket, so it went untrained - and the live LLM's exact score dropped the dispute right onto it, where the agent then picked a garbage action. Fixed by training from random start temperatures.
2. On a slow validator set a pending `take_action` outlived the receipt wait and a naive retry double-submitted - the round counter jumped by two. Fixed by giving a submitted transaction a grace window to land before ever resubmitting.

## Verified live deployment

- Network: hosted GenLayer Studio (`https://studio.genlayer.com/api`, chain `studionet`)
- Contract: **`0xA5cf174b2fDC77058C181435040121711312EE15`**

## Using the trained agent

Download the attached **`q_table.json`**, place it in `agent/`, and load it instead of retraining:

```bash
pip install -r agent/requirements.txt
python -m agent.train --env mock --episodes 500        # reproduce the curve
```

## Notes

- Alpha: reward prompts and APIs may change. Studio is a shared sandbox that can be reset - redeploy with `python -m agent.deploy --chain studionet` if the address stops resolving.
- Requires Python ≥ 3.12.

Part of the [GenLayer RL Agent Autonomy](https://github.com/luch91-org/genlayer-RL-agent-autonomy) suite (4 of 4).
