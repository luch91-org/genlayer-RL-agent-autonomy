# GenLayer RL Agent Autonomy

Umbrella vision and engineering standards for a family of independent repositories that train reinforcement learning agents against LLM-consensus reward functions on GenLayer.

GitHub organization: GenLayer RL Agent Autonomy
Org profile README lives in the org `.github` repo at `profile/README.md`.
This CLAUDE.md is the shared engineering spec. Keep it in the `.github` repo and reference it from every domain repo.

## 1. What this is

Traditional reinforcement learning needs a numeric reward function that someone writes by hand. That works for chess and Atari. It breaks the moment the thing you actually care about is subjective: "was that a good triage decision", "is this hypothesis genuinely novel", "did this compromise lower the temperature in the room".

GenLayer changes the input to that equation. Intelligent Contracts are Python contracts that call LLMs natively and reach consensus on non-deterministic outputs through Optimistic Democracy. That means the reward signal itself can be a judgment call, evaluated by a committee of validators running diverse models, and recorded immutably on-chain.

This project connects a normal off-chain RL agent to that on-chain judge.

The agent never sees the rubric. It acts, gets scored by the LLM-consensus reward, updates its policy, and repeats. Over enough rounds it converges on behavior that satisfies human-like judgment, not a hard-coded number. The reward logic lives in a deployed contract, so the agent cannot rewrite it. That immutability is the safety property that makes an open-ended "here are the controls, improve" loop tractable instead of reckless.

Framed honestly: this is subjective-reward RL on-chain (RLAIF, judged by validator consensus). It is not open-ended self-modification. The agent optimizes a fixed, external, immutable objective. That constraint is a feature.

## 2. Why GenLayer specifically

- Native LLM calls inside the contract. The reward function is a prompt, not a formula.
- Validator consensus on subjective outputs. A single model can be gamed or biased. A diverse committee reaching agreement under an equivalence principle is far harder to exploit.
- Immutability. Once deployed, the reward function cannot be edited by the agent. No wireheading the scorer.
- Native web access. Rewards can be grounded in live real-world data (news, arXiv, on-chain analytics) with no external oracle.

No other chain lets a reward function say "score this on diplomatic tact" and have that score be trustlessly agreed upon.

## 3. The four domains

Each is a fully independent repository. Build order is top to bottom. Crisis Negotiator ships first because its state space is small enough to watch the agent learn in real time, but rich enough to show LLM-based reward in action.

| Repository | Domain | The agent learns to |
|---|---|---|
| `genlayer-rl-crisis-negotiator` | Disaster response | Dispatch drones, ambulances, and supplies to critical zones without wasting capacity |
| `genlayer-rl-protocol-immunologist` | DAO treasury defense | Pause, rotate signers, and hedge to preserve capital, but only when a threat is actually trending |
| `genlayer-rl-scientific-heretic` | Hypothesis generation | Propose novel, falsifiable, plausible hypotheses that a peer reviewer would find interesting |
| `genlayer-rl-diplomatic-interpreter` | Cross-community mediation | Draft compromise text that lowers polarization and raises the odds of agreement on both sides |

## 4. The loop, in one picture

```
   off-chain (your machine)                  on-chain (GenLayer)
 ┌───────────────────────────┐        ┌──────────────────────────────┐
 │  RL agent                 │  read  │  Intelligent Contract         │
 │  - reads state            │ <───── │  - holds the environment state│
 │  - picks an action (ε-greedy)      │  - applies the action         │
 │  - updates Q-table        │  write │  - LEADER calls an LLM to     │
 │    from the reward        │ ─────> │    score the new state        │
 │                           │        │  - VALIDATORS agree on the    │
 │                           │ reward │    score via eq_principle     │
 │                           │ <───── │  - records reward on-chain    │
 └───────────────────────────┘        └──────────────────────────────┘
```

The contract is the scorekeeper. The agent is the student. The student keeps guessing, the scorekeeper grades, the student rewrites its notes.

## 5. End state we are building toward

For every one of the four repositories, "done" means all of the following are true.

1. Deployed Intelligent Contract that defines the environment state, the action space, and an LLM-consensus reward function. Runs on Localnet and on public testnet (Bradbury) / Studio.
2. Off-chain RL agent that queries state, selects actions with epsilon-greedy exploration, sends transactions, and updates a Q-table from the returned reward.
3. Measurable learning. Average reward climbs from roughly 3 to 8 or higher over training. This curve is saved and plotted, not just asserted.
4. Saved policy committed as a release artifact (`q_table.json`), so a reviewer can load a trained agent without retraining.
5. Documentation a beginner can follow end to end: setup, deploy, train, and read the results.
6. A short demo GIF or video of the reward line climbing.
7. Clean, lint-passing contracts using the current GenLayer SDK. `genvm-lint` passes with no warnings.

The larger deliverable across all four: the first open, forkable template for "RL against an on-chain LLM reward" on GenLayer. Other builders clone one repo, swap the domain, and have a working subjective-reward RL environment in an afternoon.

## 6. Design principles and known constraints

These are load-bearing. Every repo respects them.

1. On-chain reward is slow and costs gas. Each `take_action` triggers LLM inference across validators plus consensus. You do not develop RL logic by hammering the chain 500 times. Every repo ships a `MockEnv` (instant, local, free) as the default for development, CI, and hyperparameter tuning, and a `GenLayerEnv` you switch to for the real demo. Live training runs use a modest episode budget and are resumable.
2. LLM rewards are noisy and non-stationary. The same state will not always score identically. Agents use reward smoothing and treat the consensus score as ground truth for the demo. Do not expect textbook-clean convergence.
3. Subjective scores never match exactly across validators. Never use `strict_eq` for a reward score. Use a comparative equivalence principle.
4. Tabular Q-learning fits these small discrete state spaces. It is intentionally the simplest thing that works, so the learning is legible line by line. Function approximation is a documented future path, not a v1 requirement.
5. The reward function is immutable once deployed. The agent optimizes it, it cannot edit it. State this explicitly in every README as the safety property.

## 7. Repository standards (apply to all four)

Directory layout:

```
.
├── contracts/
│   └── <domain>.py            # Intelligent Contract: state, actions, LLM reward
├── agent/
│   ├── env.py                 # Env interface + MockEnv + GenLayerEnv
│   ├── agent.py               # Q-learning loop, epsilon-greedy, save/resume
│   ├── train.py               # entrypoint: python -m agent.train
│   ├── plot.py                # renders the learning curve
│   └── requirements.txt
├── tests/
│   ├── test_contract.py       # GenVM tests for state transitions + reward validation
│   └── test_agent.py          # mocked tests for action selection + Q-updates
├── docs/
│   ├── tutorial.md
│   └── learning_curve.png
├── logs/
│   └── training.txt
├── CLAUDE.md                  # domain-scoped build guide
├── README.md
├── LICENSE                    # MIT
└── .github/workflows/ci.yml
```

Conventions:

- Python style: Black + isort. Contracts additionally pass `genvm-lint`.
- Commits: Conventional Commits (`feat:`, `fix:`, `docs:`, `test:`, `chore:`).
- Branches: `main` stable, `develop` integration, feature branches per issue.
- Versioning: semantic. First tag `v0.1.0-alpha`. Trained `q_table.json` attached to the GitHub Release.
- Every repo enables Issues and Discussions and links back to the org profile.
- Off-chain agent transport is the first-party `genlayer-py` SDK (Python >= 3.12). No Node bridge required.

Runner pin: contracts declare the pinned GenVM runner for reproducibility:

```python
# { "Depends": "py-genlayer:1jb45aa8ynh2a9c9xn3b7qqh8sm5q93hwfp7jqmwsfhh8jpz09h6" }
```

## 8. GenLayer contract rules that are non-negotiable

The most common ways a GenLayer contract silently fails or refuses to lint. Baked into every contract in this project.

1. Non-deterministic calls (`gl.nondet.exec_prompt`, `gl.nondet.web.get`, `gl.nondet.web.render`) must live inside an inner function that is passed to an equivalence principle or `gl.vm.run_nondet`. They cannot sit directly in the method body.
2. You cannot access `self` inside that inner nondet function. Snapshot every value you need into local variables first, then reference the locals.
3. The leader function must actually perform the LLM call and return its result. A function that only builds a prompt string runs no inference.
4. Subjective reward scores use `gl.eq_principle.prompt_comparative(fn, principle)` with a stated tolerance, or `prompt_non_comparative`. Never `strict_eq` for a score. `strict_eq` is only for outputs that should be byte-identical across validators.
5. State variables are declared with type annotations at class level and initialized in `__init__`.
6. Current SDK namespaces: `gl.nondet.*`, `gl.eq_principle.*`, `gl.vm.*`. The old flat forms (`gl.exec_prompt`, `gl.get_webpage`, `gl.eq_principle_strict_eq`) are deprecated. Always confirm signatures against the live docs before finalizing.

Sources of truth: docs.genlayer.com and sdk.genlayer.com.

## 9. How to contribute

1. Start from the org profile, which indexes all four domain repositories.
2. Pick a domain repository. Read its `README.md` and `docs/tutorial.md`.
3. Open or claim an issue in that repo.
4. Fork, branch, commit under Conventional Commits, open a PR.
5. CI must pass, and PRs that touch the agent include a short training-log snippet showing the curve.

## 10. Vision statement

We are building the first open playground where agents do not just execute tasks, they learn contested, human-like judgment through subjective feedback that is agreed upon by a decentralized committee and written to a blockchain. Small, legible, forkable. A concrete step toward autonomous systems whose objectives are transparent, external, and impossible for the agent to quietly rewrite.
