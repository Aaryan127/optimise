
## 1. Multi-Head Attention Enhancement

### Objective

Increase the number of attention heads in the LocoTransformer architecture from 1 to 2 per
transformer layer.

### Files Modified

`config/rl/moving/locotransformer/thin.json`

### Original Configuration

```json
"transformer_params": [
    [1, 256],
    [1, 256]
]
```

### Proposed Configuration

```json
"transformer_params": [
    [2, 256],
    [2, 256]
]
```

### Motivation

The baseline LocoTransformer architecture uses a single attention head in each transformer
layer. While functional, a single attention head can only learn one dominant interaction
pattern between the robot state token and visual terrain tokens.

Multi-head attention allows the model to learn multiple independent relationships
simultaneously. Different heads may specialise in different aspects of locomotion, such as:

* Terrain assessment
* Foothold selection
* Robot posture estimation
* Obstacle awareness
* Velocity and balance regulation

This can potentially improve cross-modal fusion between proprioceptive state information and
visual observations.

### Dimensional Validity

`nn.TransformerEncoderLayer` requires `d_model % n_head == 0`. The `d_model` here is
`encoder.visual_dim = 64`. With `n_head = 2`: `64 % 2 = 0`. ✓

The `256` in `transformer_params` is `dim_feedforward`, which is independent of `n_head`
and imposes no constraint.

### Expected Benefits

* Richer token interactions
* Improved representation learning
* Better utilisation of visual terrain information
* Increased model expressiveness with minimal code modification

### Implementation Complexity

Low

### Code Changes Required

Only the transformer configuration values need to be changed. No modifications to the actor,
critic, PPO implementation, or transformer source code are required because the architecture
already supports arbitrary attention head counts through configuration parameters.

---

## 2. Additional Transformer Layer

### Objective

Increase transformer depth from two encoder layers to three encoder layers.

### Files Modified

`config/rl/moving/locotransformer/thin.json`

### Original Configuration

```json
"transformer_params": [
    [1, 256],
    [1, 256]
]
```

### Proposed Configuration

```json
"transformer_params": [
    [1, 256],
    [1, 256],
    [1, 256]
]
```

### Motivation

The baseline LocoTransformer architecture uses two transformer encoder layers to model
interactions between the robot state token and visual terrain tokens.

Increasing transformer depth allows information to be refined across additional stages of
self-attention. Earlier layers can learn low-level state-terrain relationships while deeper
layers can potentially capture more abstract locomotion patterns and environmental structure.

Since each transformer layer performs additional rounds of token interaction, increasing depth
may improve the quality of cross-modal feature fusion and provide richer internal
representations for downstream action generation.

### Expected Benefits

* Increased representational capacity
* Additional state-terrain interaction steps
* Improved hierarchical feature extraction
* Potentially stronger locomotion policy learning

### Expected Drawbacks

* Increased computational cost
* Slightly higher memory usage
* Longer training times
* Potential overfitting if model capacity becomes excessive

### Implementation Complexity

Low

### Code Changes Required

Only the transformer configuration requires modification. The architecture dynamically
constructs transformer layers from the `transformer_params` list, therefore no changes to the
actor, critic, PPO implementation, or transformer source code are required.

### Verification Audit

The transformer stack is constructed dynamically inside `torchrl/networks/nets.py`:

```python
for n_head, dim_feedforward in transformer_params:
    visual_att_layer = nn.TransformerEncoderLayer(...)
    self.visual_append_layers.append(visual_att_layer)
```

No assumptions regarding a fixed number of transformer layers were found during inspection.

---

## 3. Increased Transformer Feedforward Capacity

### Objective

Increase the feedforward network dimension inside each transformer encoder layer from 256
to 512.

### Files Modified

`config/rl/moving/locotransformer/thin.json`

### Original Configuration

```json
"transformer_params": [
    [1, 256],
    [1, 256]
]
```

### Proposed Configuration

```json
"transformer_params": [
    [1, 512],
    [1, 512]
]
```

### Motivation

Each transformer encoder layer contains an internal feedforward network that processes token
representations after the attention operation. In the baseline architecture, this feedforward
dimension is set to 256. Increasing it to 512 provides greater representational capacity and
allows the transformer to learn more complex state-terrain relationships.

While attention determines which tokens should interact, the feedforward network determines
how those interactions are transformed into useful features. Expanding this component may
improve the quality of learned locomotion representations.

### Dimensional Note

`dim_feedforward` is the internal FFN width within each `TransformerEncoderLayer`. It is
entirely independent of `d_model` (`encoder.visual_dim = 64`) and imposes no constraint on
other dimensions. Changing it from 256 to 512 has no effect on input or output shapes of
the transformer stack.

### Expected Benefits

* Increased transformer capacity
* Richer feature transformations
* Improved token representation learning
* Potentially stronger state-terrain reasoning

### Expected Drawbacks

* Increased parameter count
* Higher memory consumption
* Longer training time

### Implementation Complexity

Low

### Code Changes Required

Only the transformer configuration requires modification. No changes to the actor, critic,
PPO implementation, transformer source code, or encoder architecture are necessary.

### Verification Audit

The feedforward dimension is dynamically supplied to:

```python
nn.TransformerEncoderLayer(
    visual_append_input_shape,   # d_model = encoder.visual_dim = 64
    n_head,
    dim_feedforward              # this value, independent of d_model
)
```

inside `torchrl/networks/nets.py`. No hardcoded dependency on a feedforward size of 256 was
found during inspection.

---

## 4. Max Pooling Token Aggregation

### Objective

Replace mean pooling of transformer output tokens with max pooling.

### Files Modified

`config/rl/moving/locotransformer/thin.json`

### Original Configuration

```json
"net": {
    "transformer_params": [
        [1, 256],
        [1, 256]
    ],
    "append_hidden_shapes": [256, 256]
}
```

### Proposed Configuration

```json
"net": {
    "transformer_params": [
        [1, 256],
        [1, 256]
    ],
    "append_hidden_shapes": [256, 256],
    "max_pool": true
}
```

### Motivation

The baseline LocoTransformer aggregates visual transformer tokens using mean pooling, which
treats every visual token equally and averages their representations into a single feature
vector. However, during locomotion a small number of highly relevant terrain tokens may
contain critical information regarding obstacles, footholds, gaps, or unstable terrain.
Averaging all tokens may dilute these important signals. Max pooling preserves the strongest
activation across tokens, allowing the network to focus on the most salient terrain features.

### Expected Benefits

* Improved preservation of important terrain features
* Stronger obstacle-related representations
* Reduced dilution of critical visual signals
* Potentially improved terrain awareness

### Expected Drawbacks

* Loss of global averaging information
* Increased sensitivity to noisy activations
* Possible instability if individual tokens become overly dominant

### Implementation Complexity

Very Low

### Code Changes Required

No Python source code modifications are required. The implementation already exists inside
`torchrl/networks/nets.py` in `LocoTransformer.forward()`:

```python
if self.max_pool:
    out_first = out[1: 1 + self.per_modal_tokens, ...].max(dim=0)[0]
else:
    out_first = out[1: 1 + self.per_modal_tokens, ...].mean(dim=0)
```

The feature is activated by passing `"max_pool": true` through the network configuration.

### Verification Audit

The `LocoTransformer` constructor exposes `max_pool=False` and receives parameters through
`**params["net"]` from `starter/ppo_locotransformer.py`. The pooling slicing correctly
excludes the state token (index 0) and operates only over terrain tokens
`out[1: 1 + self.per_modal_tokens, ...]`. No additional modifications are necessary.

---

## 5. Token-Level Layer Normalisation

### Objective

Enable token normalisation before transformer processing using Layer Normalisation.

### Files Modified

`config/rl/moving/locotransformer/thin.json`

### Original Configuration

```json
"net": {
    "transformer_params": [
        [1, 256],
        [1, 256]
    ],
    "append_hidden_shapes": [256, 256]
}
```

### Proposed Configuration

```json
"net": {
    "transformer_params": [
        [1, 256],
        [1, 256]
    ],
    "append_hidden_shapes": [256, 256],
    "token_norm": true
}
```

### Motivation

Transformer architectures frequently benefit from feature normalisation before attention
operations. The LocoTransformer implementation already contains a token normalisation pathway
based on Layer Normalisation but leaves it disabled by default. Enabling it standardises
token feature distributions before attention processing, which may improve optimisation
stability and reduce sensitivity to feature magnitude variations. This is particularly useful
when combining heterogeneous information sources such as proprioceptive state features and
visual terrain features.

### Known Limitation

`LocoTransformer.__init__()` creates both `self.token_ln` and `self.state_token_ln` when
`token_norm=True`, but `forward()` only applies `self.token_ln` to the full token sequence.
`self.state_token_ln` is defined but never called. As a result, the state token is not
independently normalised even when this flag is enabled. This is a latent incompleteness in
the original codebase and will not cause any runtime error, but it means the state token
bypasses the normalisation that `self.state_token_ln` was presumably intended to provide.

### Expected Benefits

* Improved training stability
* Better-conditioned token representations
* Reduced feature-scale sensitivity
* Potentially improved attention behaviour

### Expected Drawbacks

* Slight computational overhead
* Possible reduction in useful feature magnitude information

### Implementation Complexity

Very Low

### Code Changes Required

No Python source code modifications are required to enable this feature. The functionality
is already implemented inside `torchrl/networks/nets.py`:

```python
if self.token_norm:
    self.token_ln = nn.LayerNorm(self.encoder.visual_dim)
    self.state_token_ln = nn.LayerNorm(self.encoder.visual_dim)  # defined but not applied
```

and in `forward()`:

```python
if self.token_norm:
    out = self.token_ln(out)
```

The feature can be enabled entirely through configuration.

---

## 6. Post-Transformer Layer Normalisation

### Objective

Enable Layer Normalisation within the post-transformer MLP head.

### Files Modified

`config/rl/moving/locotransformer/thin.json`

### Original Configuration

```json
"net": {
    "transformer_params": [
        [1, 256],
        [1, 256]
    ],
    "append_hidden_shapes": [256, 256]
}
```

### Proposed Configuration

```json
"net": {
    "transformer_params": [
        [1, 256],
        [1, 256]
    ],
    "append_hidden_shapes": [256, 256],
    "add_ln": true
}
```

### Motivation

After transformer processing and token aggregation, the resulting feature vector is passed
through a multi-layer perceptron before generating actions and value estimates. The
implementation already contains optional Layer Normalisation blocks that can be inserted
after each hidden layer of this MLP. Layer Normalisation can stabilise feature distributions
during training, improve optimisation dynamics, and reduce sensitivity to feature magnitude
variations. Unlike token normalisation (Proposal 5), which operates on transformer tokens,
this modification normalises features inside the final decision-making MLP.

### Expected Benefits

* More stable hidden activations
* Improved optimisation behaviour
* Better-conditioned feature representations
* Potentially smoother PPO training

### Expected Drawbacks

* Slight computational overhead
* Possible suppression of useful feature magnitude information

### Implementation Complexity

Very Low

### Code Changes Required

No Python source code modifications are required. The implementation already exists inside
`torchrl/networks/nets.py`:

```python
if self.add_ln:
    self.visual_append_fcs.append(
        nn.LayerNorm(next_shape)
    )
```

Inspection confirmed that Layer Normalisation is inserted only within the post-transformer
MLP stack and does not modify the transformer attention layers, visual encoder, state
tokeniser, or PPO implementation.

---

## 7. Mean + Max Pool Fusion

### Objective

Replace single mean-pooled terrain aggregation with a combined mean-pooled and max-pooled
terrain representation.

### Files Modified

`torchrl/networks/nets.py`

### Original Architecture

```
Terrain Tokens
↓
Mean Pool  →  64-dim vector
↓
[state_token (64) | mean_terrain (64)]  →  128-dim MLP input
↓
MLP
↓
Action / Value Output
```

### Proposed Architecture

```
Terrain Tokens
↓
Mean Pool  →  64-dim vector
Max Pool   →  64-dim vector
↓
[state_token (64) | mean_terrain (64) | max_terrain (64)]  →  192-dim MLP input
↓
MLP
↓
Action / Value Output
```

### Motivation

The baseline LocoTransformer aggregates terrain information exclusively through mean pooling.
While this captures overall terrain structure, it may dilute highly informative terrain
features such as obstacles, edges, gaps, foothold candidates, or abrupt height changes.

By combining mean pooling and max pooling, the network receives both global terrain
information and the strongest terrain activations simultaneously. This allows the policy to
reason about overall terrain layout and the most critical local terrain features at the same
time.

The approach is inspired by aggregation strategies used in PointNet, PointNet++, Point
Transformer, and Set Transformer, where mean-max fusion has demonstrated effectiveness in
preserving both global context and highly informative local features.

### Dimension Analysis

With `encoder.visual_dim = 64` (i.e. `token_dim = 64`):

| Component | Dimension |
|-----------|-----------|
| `out_state` (state token) | 64 |
| `out_mean` (mean-pooled terrain) | 64 |
| `out_max` (max-pooled terrain) | 64 |
| **Total MLP input** | **192** |

Current MLP input is `64 * 2 = 128` (state + mean). The proposed change increases this to
`64 * 3 = 192` (state + mean + max), requiring the `* 2` multiplier in `__init__` to
become `* 3`.

If `in_channels == 16` (RGB + Depth, i.e. `self.second = True`), the same dual-pool logic
applies to the second modality as well, and the additional `encoder.visual_dim` added for
`self.second` must also be duplicated.

### Expected Benefits

* Improved obstacle awareness
* Better preservation of salient terrain features
* Richer terrain representations
* Stronger state-terrain feature fusion
* Increased feature diversity entering the policy network

### Expected Drawbacks

* Larger feature vector entering the MLP (128 → 192)
* Increased parameter count in the first MLP layer
* Slightly higher memory usage
* Slightly longer training time

### Implementation Complexity

Medium

### Code Changes Required

Two locations in `torchrl/networks/nets.py` must be modified.

**Change 1 — `LocoTransformer.__init__()`, line 974**

Update the MLP input dimension calculation to account for three concatenated vectors instead
of two:

```python
# ORIGINAL (line 974)
visual_append_input_shape = visual_append_input_shape * 3 * 2

# PROPOSED
visual_append_input_shape = visual_append_input_shape * 3
```

If `self.second` is True (RGB+Depth mode), the `self.second` branch below this line adds
one extra `encoder.visual_dim`. For full consistency in dual-modality mode this should also
be doubled, but for the standard single-modality (`in_channels=4`, depth-only) setup used
by `thin.json`, only the `* 3` change is required.

**Change 2 — `LocoTransformer.forward()`, lines 1015-1022**

Replace the existing conditional pooling block with unconditional dual pooling:

```python
# ORIGINAL
out_state = out[0, ...]
if self.max_pool:
    out_first = out[1: 1 + self.per_modal_tokens, ...].max(dim=0)[0]
else:
    out_first = out[1: 1 + self.per_modal_tokens, ...].mean(dim=0)
out_list = [out_state, out_first]

# PROPOSED
out_state = out[0, ...]
out_mean  = out[1: 1 + self.per_modal_tokens, ...].mean(dim=0)
out_max   = out[1: 1 + self.per_modal_tokens, ...].max(dim=0)[0]
out_list  = [out_state, out_mean, out_max]
```

With this change the `max_pool` config flag becomes redundant for this class (both pools are
always computed). It can be left absent from the config or ignored.

---

## 8. Learnable Positional Embeddings

### Objective

Introduce learnable positional embeddings to provide explicit spatial information to
transformer tokens.

### Files Modified

`torchrl/networks/base.py`

### Original Architecture

```
State Token (1)
+
Terrain Tokens (per_modal_tokens = 16)
↓
No positional information
↓
Transformer
```

### Proposed Architecture

```
State Token (1)
+
Terrain Tokens (per_modal_tokens = 16)
+
Learnable Positional Embeddings (17 × 1 × 64)
↓
Transformer
```

### Motivation

The baseline LocoTransformer constructs a sequence consisting of one state token and multiple
terrain tokens before passing them into the transformer encoder. Inspection of the codebase
confirmed that no positional embeddings or positional encoding mechanisms exist anywhere in
the tokenisation or transformer pipeline. The transformer therefore receives token content
but has no explicit information about the spatial location of terrain patches.

For locomotion tasks, spatial location is highly relevant because the robot must distinguish
between terrain features in different regions of the visual field. Learnable positional
embeddings allow the transformer to associate each token with a unique spatial position,
improving its ability to reason about terrain layout and obstacle location.

Positional embeddings are a foundational component of modern transformer architectures
including the original Transformer (Vaswani et al.), ViT, DeiT, BEiT, MAE, and DINO.

### Token Count

With `two_by_two=False` (default), `per_modal_tokens = 16`. For `in_channels=4` (depth
only, as configured by `thin.json`):

```
total_tokens = 1 (state) + 16 (depth terrain) = 17
```

For `in_channels=16` (RGB+Depth):

```
total_tokens = 1 (state) + 16 (depth) + 16 (rgb) = 33
```

### Expected Benefits

* Improved spatial awareness
* Better terrain layout understanding
* More informative attention patterns
* Enhanced state-terrain interaction

### Expected Drawbacks

* Additional trainable parameters (`total_tokens × token_dim = 17 × 64 = 1088` parameters
  for the standard depth-only setup)
* Slightly increased model complexity

### Implementation Complexity

Medium

### Code Changes Required

Two locations in `torchrl/networks/base.py` must be modified within `LocoTransformerEncoder`.

**Change 1 — `LocoTransformerEncoder.__init__()`, after line 548 (`self.flatten_layer = Flatten()`)**

Add the positional embedding parameter:

```python
# After: self.flatten_layer = Flatten()

# Positional embeddings: one per token in the transformer input sequence
if self.in_channels == 16:
    total_tokens = 1 + 2 * self.per_modal_tokens
else:
    total_tokens = 1 + self.per_modal_tokens
self.pos_embedding = nn.Parameter(
    torch.zeros(total_tokens, 1, token_dim)
)
nn.init.trunc_normal_(self.pos_embedding, std=0.02)
```

**Change 2 — `LocoTransformerEncoder.forward()`, line 622, replace:**

```python
# ORIGINAL
visual_out = torch.cat(out_list, dim=0)

# PROPOSED
visual_out = torch.cat(out_list, dim=0)
visual_out = visual_out + self.pos_embedding
```

The positional embedding tensor has shape `(total_tokens, 1, token_dim)`. The `1` in the
middle broadcasts correctly across the batch dimension of `visual_out`, which has shape
`(total_tokens, batch_size, token_dim)`.

                                                                                                                  DONE WITH THE HELP OF CLAUDE OPUS AND CHATGPT 5
