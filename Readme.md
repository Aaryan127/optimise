The idea is to improve the RL based transformer policy - [LocoTransformer](https://rchalyang.github.io/LocoTransformer/)

## Core Highlights of the LocoTransformer:

- **Cross-Modal Tokenization:** Notice how the `LocoTransformerEncoder` treats the proprioceptive state as a single token while breaking visual feature maps into spatial tokens. This allows the transformer to learn dependencies between robot pose and specific environmental obstacles.
- **Actor-Critic Architecture:** The PPO implementation uses a shared encoder for both the Actor (Policy) and Critic (Value) networks, facilitating shared representation learning across the two heads.
- **Temporal & Delay Handling:** The MMDR (Multi-Modal Delay Randomization) logic is integrated to handle the asynchronous nature of visual and proprioceptive sensors, which is critical for real-world deployment.
- **Hierarchical Sensor Wrappers:** The codebase uses history-stacking wrappers to provide the networks with short-term temporal context, essential for estimating velocities and contact states from raw data.

---

## Your task with Locotransformer:
- [ ] Check if the implementation can run on your system as is.
      - [ ] Use `uv` and `micromamba` for package management and installations - should be fine.
- [ ] Make necessary changes so that it runs along with mujoco on your system - do not worry about performance at this stage, get it working first.
- [ ] Train the policy and evaluate the results.
- [ ] The policy will (mostly) perform very poorly, in case you much think of specific changes you will make to the architecture/training setup to improve it.
