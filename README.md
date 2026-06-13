# Toolkit for Vision-Guided Quadrupedal Locomotion Research (Pybullet)


Official implementations of

**Learning Vision-Guided Quadrupedal Locomotion End-to-End with Cross-Modal Transformers** (LocoTransformer)<br/>
[Ruihan Yang*](https://rchalyang.github.io/), [Minghao Zhang*](https://www.minghaozhang.com), [Nicklas Hansen](https://nicklashansen.github.io/), [Huazhe Xu](http://hxu.rocks/), [Xiaolong Wang](https://xiaolonw.github.io)

[[Arxiv]](https://arxiv.org/abs/2107.03996) [[Webpage]](https://rchalyang.github.io/LocoTransformer/) [[ICLR Paper]](https://openreview.net/forum?id=nhnJ3oo6AB)

and

**Vision-Guided Quadrupedal Locomotion in the Wild with Multi-Modal Delay Randomization** (MMDR)<br/>
[Chieko Sarah Imai*](https://www.linkedin.com/in/csimai/), [Minghao Zhang*](https://www.minghaozhang.com), [Yuchen Zhang*](https://github.com/infinity1096), [Marcin Kierebiński](https://mrkiereb.com/), [Ruihan Yang](https://rchalyang.github.io/), [Yuzhe Qin](https://yzqin.github.io/), [Xiaolong Wang](https://xiaolonw.github.io)

[[Arxiv]](https://arxiv.org/abs/2109.14549) [[Webpage]](https://mehooz.github.io/mmdr-wild/)


<!-- ![delay](figures/real_robot_test.png) -->

Our repository contains the necessary functions to train the policy in simulation and deploy the learned policy on the real robot in the real world. Directly testes in the real robot, we can see the A1 robot traverses in versatile real-world scenarios:

![diverse_scenarios](figures/diverse_scenarios.png)

## Simulation Environments

<!-- Our repository contains the following environments in simulation:

![environment samples](figures/env_samples.png)

including uneven terrains, random-shaped obstacles, and various other environments. -->

With the depth maps as policy input, RL agents can learn to navigate through our simulated environments. We mainly use the following scenarios (tasks in LocoTransformer):

![environment samples](figures/teaser-long.png)

The following visualization results show the attention mechanism of the LocoTransformer at the different regions of the image:

![visualization](figures/visualization.png)

In MMDR, we also simulate multiple sim-to-real gap. For example, we simulate the "blinding spot" in the RealSense when sampling depth images:

![simulate_realsense](figures/simulate_realsense.png)


We also provide the functions to simulate the multi-modal latencies. 

![delay](figures/delay.png)


Please refer to our [paper](https://arxiv.org/abs/2109.14549) for more details.


## Setup

We assume that you have access to a GPU with CUDA >=9.2 support. All dependencies can then be installed with the following commands:

```
pip install -e .
```

## Configuration for enviornments

#### RL training
We use config files in folder `config` to configure the parameters of training and enviornment.
In `config/rl/`, there are three types of config files:
- `static`: Train in basic plane ground or uneven terrain with static obstacles.
- `moving`: Train in basic plane ground or uneven terrain with moving obstacles.
- `challenge`: Some challenging scenarios like mountain, hill, and so on.

In each folder, we have subfolders for different algorithms:
- `naive_baseline`: Train a naive baseline policy. No frame-extraction, no delay randomization.
- `frame_extract4`: Train a policy with frame-extraction (k=4 in the MMDR paper). No delay randomization.
- `frame_extract4_fixed_delay`: Train a policy with frame-extraction (k=4 in the MMDR paper) and fixed delay in all episodes.
- `frame_extract4_random_delay`: Train a policy with frame-extraction (k=4 in the MMDR paper) and random delay in each episodes.
- `locotransformer`: Train a LocoTransformer policy.
- `locotransformer_random_delay`: Train a LocoTransformer policy with random delay in each episodes.

In challeging scenarios, we only put the `baseline` and `locotransformer` folder as config files, since we didn't conduct MMDR experiements on these scenarios.

#### Visual-MPC Training
While RL can navigate from proprioception and vision by outputing the joint angles directly, we also provide a visual-MPC training to output the control command (linear and angular velocity). See the comparison in Locotransformer paper.

To reproduce, we use the following config folders:
- `mpc`: Train a visual-MPC policy with both proprioception and vision as input.
- `mpc_vision_only`: Train a visual-MPC policy with only vision as input.

Besides, You can also configurate the network architecture in `config/`. We only use PPO for all the experiments, so we didn't put the configuration of other algorithms. But users can still use them in `torchrl` library.

## Training & Evaluation

The `starter` directory contains training and evaluation scripts for all the included algorithms. The `config` directory contains training configuration files for all the experiments. You can use the python scripts, e.g. for training call

```
python starter/ppo_locotransformer.py \
  --config config/rl/static/locotransformer/thin-goal.json \
  --seed 0 \
  --log_dir {YOUR_LOG_DIR} \
  --id {YOUR_ID}
```

to run PPO+LocoTransformer on the environment, `thin-goal`. And you can use

```
python starter/locotransformer_viewer.py \
  --seed 0 \
  --log_dir {YOUR_LOG_DIR} \
  --id {YOUR_ID} \
  --env_name A1MoveGround
```
to test the trained model on the same environment.

## Real Robot Deployment

### Set up robot interface

Since our robot interface based on  [Motion Imitation](https://github.com/erwincoumans/motion_imitation). We provide the simplified interface setup instruction here. For detailed version please check [Motion Imitation](https://github.com/erwincoumans/motion_imitation)

build the python interface by running the following:
```bash
cd third_party/unitree_legged_sdk
mkdir build
cd build
cmake ..
make
```
Then copy the built `robot_interface.XXX.so` file to the main directory (where you can see this README.md file).

To deploy visual-policy, we should also set up Intel RealSense interface. Please check detailed instruction on [Librealsense](https://github.com/IntelRealSense/librealsense)

### (Optional) Convert pytorch policy to tensorrt policy
```
# Current 
bash a1_hardware/convert_tensor_rt/convert_trt.sh $EXP_ID $SEED $LOG_ROOT_PATH
```



### Run trained policy on the robot.
```
# For Tensor RT Version
python a1_hardware/execute_locotransformer_trt.py

# For Pytorch Version
python a1_hardware/execute_locotransformer.py
```
**We use joint control for A1, the default action and action scale are predefined. Normalization information for depth input is also predefined.**

**Some system path and configuration may varies, please modify accordingly.**

## Results

See [LocoTransformer](https://arxiv.org/abs/2107.03996) and [MMDR](https://arxiv.org/abs/2109.14549) for results.


## Citation
<a name="citation"></a>
If you find our code useful in your research, please consider citing our work as follows:

for LocoTransformer:
```
@inproceedings{
    yang2022learning,
    title={Learning Vision-Guided Quadrupedal Locomotion End-to-End with Cross-Modal Transformers},
    author={Ruihan Yang and Minghao Zhang and Nicklas Hansen and Huazhe Xu and Xiaolong Wang},
    booktitle={International Conference on Learning Representations},
    year={2022},
    url={https://openreview.net/forum?id=nhnJ3oo6AB}
}
```

for MMDR:
```

@inproceedings{Imai2021VisionGuidedQL, 
   title={Vision-Guided Quadrupedal Locomotion in the Wild with Multi-Modal Delay Randomization}, 
   author={Chieko Imai and Minghao Zhang and Yuchen Zhang and Marcin Kierebinski and Ruihan Yang and Yuzhe Qin and Xiaolong Wang}, 
   booktitle={2022 IEEE/RSJ international conference on intelligent robots and systems (IROS)}, 
 } 
```

## Acknowledgements

This repository is a product of our work on [LocoTransformer](https://rchalyang.github.io/LocoTransformer/) and [MMDR](https://mehooz.github.io/mmdr-wild/). Our RL implementation is based on [TorchRL](https://github.com/RchalYang/torchrl), and the environment implementation is based on [google motion-imitation](https://github.com/google-research/motion_imitation). The python interface for real robot is also based on [google motion-imitation](https://github.com/google-research/motion_imitation). 






## 1. Multi-Head Attention Enhancement

### Objective

Increase the number of attention heads in the LocoTransformer architecture from 1 to 2 per transformer layer.

### Files Modified

`config/rl/moving/locotransformer/thin.json`

### Original Configuration

```json
"transformer_params": [
    [1,256],
    [1,256]
]
```

### Proposed Configuration

```json
"transformer_params": [
    [2,256],
    [2,256]
]
```

### Motivation

The baseline LocoTransformer architecture uses a single attention head in each transformer layer. While functional, a single attention head can only learn one dominant interaction pattern between the robot state token and visual terrain tokens.

Multi-head attention allows the model to learn multiple independent relationships simultaneously. Different heads may specialize in different aspects of locomotion, such as:

* Terrain assessment
* Foothold selection
* Robot posture estimation
* Obstacle awareness
* Velocity and balance regulation

This can potentially improve cross-modal fusion between proprioceptive state information and visual observations.

### Expected Benefits

* Richer token interactions
* Improved representation learning
* Better utilization of visual terrain information
* Increased model expressiveness with minimal code modification

### Implementation Complexity

Low

### Code Changes Required

Only the transformer configuration values need to be changed. No modifications to the actor, critic, PPO implementation, or transformer source code are required because the architecture already supports arbitrary attention head counts through configuration parameters.



## 2. Additional Transformer Layer

### Objective

Increase transformer depth from two encoder layers to three encoder layers.

### Files Modified

`config/rl/moving/locotransformer/thin.json`

### Original Configuration

```json
"transformer_params": [
    [1,256],
    [1,256]
]
```

### Proposed Configuration

```json
"transformer_params": [
    [1,256],
    [1,256],
    [1,256]
]
```

### Motivation

The baseline LocoTransformer architecture uses two transformer encoder layers to model interactions between the robot state token and visual terrain tokens.

Increasing transformer depth allows information to be refined across additional stages of self-attention. Earlier layers can learn low-level state-terrain relationships while deeper layers can potentially capture more abstract locomotion patterns and environmental structure.

Since each transformer layer performs additional rounds of token interaction, increasing depth may improve the quality of cross-modal feature fusion and provide richer internal representations for downstream action generation.

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

Only the transformer configuration requires modification. The architecture dynamically constructs transformer layers from the `transformer_params` list, therefore no changes to the actor, critic, PPO implementation, or transformer source code are required.

### Verification Audit

The transformer stack is constructed dynamically inside:

`torchrl/networks/nets.py`

using:

```python
for n_head, dim_feedforward in transformer_params:
    visual_att_layer = nn.TransformerEncoderLayer(...)
    self.visual_append_layers.append(visual_att_layer)
```

No assumptions regarding a fixed number of transformer layers were found during inspection.



## 3. Increased Transformer Feedforward Capacity

### Objective

Increase the feedforward network dimension inside each transformer encoder layer from 256 to 512.

### Files Modified

`config/rl/moving/locotransformer/thin.json`

### Original Configuration

```json
"transformer_params": [
    [1,256],
    [1,256]
]
```

### Proposed Configuration

```json
"transformer_params": [
    [1,512],
    [1,512]
]
```

### Motivation

Each transformer encoder layer contains an internal feedforward network that processes token representations after the attention operation.

In the baseline architecture, the feedforward dimension is set to 256. Increasing this dimension to 512 provides greater representational capacity and allows the transformer to learn more complex state-terrain relationships.

While attention determines which tokens should interact, the feedforward network determines how those interactions are transformed into useful features. Expanding this component may improve the quality of learned locomotion representations.

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

Only the transformer configuration requires modification.

No changes to the actor, critic, PPO implementation, transformer source code, or encoder architecture are necessary.

### Verification Audit

The feedforward dimension is dynamically supplied to:

```python
nn.TransformerEncoderLayer(
    visual_append_input_shape,
    n_head,
    dim_feedforward
)
```

inside:

`torchrl/networks/nets.py`

No hardcoded dependency on a feedforward size of 256 was found during inspection.



## 4. Max Pooling Token Aggregation

### Objective

Replace mean pooling of transformer output tokens with max pooling.

### Files Modified

`config/rl/moving/locotransformer/thin.json`

### Original Configuration

```json
"net": {
    "transformer_params": [
        [1,256],
        [1,256]
    ],
    "append_hidden_shapes": [
        256,
        256
    ]
}
```

### Proposed Configuration

```json
"net": {
    "transformer_params": [
        [1,256],
        [1,256]
    ],
    "append_hidden_shapes": [
        256,
        256
    ],
    "max_pool": true
}
```

### Motivation

The baseline LocoTransformer aggregates visual transformer tokens using mean pooling.

Mean pooling treats every visual token equally and averages their representations into a single feature vector.

However, during locomotion, a small number of highly relevant terrain tokens may contain critical information regarding obstacles, footholds, gaps, or unstable terrain. Averaging all tokens may dilute these important signals.

Max pooling preserves the strongest activation across tokens and allows the network to focus on the most salient terrain features.

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

No Python source code modifications are required.

The implementation already exists inside:

`torchrl/networks/nets.py`

through:

```python
if self.max_pool:
    out_first = ...
else:
    out_first = ...
```

The feature is activated by passing:

```json
"max_pool": true
```

through the network configuration.

### Verification Audit

The `LocoTransformer` and `Transformer` constructors both expose:

```python
max_pool=False
```

and receive parameters through:

```python
**params["net"]
```

from:

`starter/ppo_locotransformer.py`

No additional modifications were found to be necessary.



## 5. Token-Level Layer Normalization

### Objective

Enable token normalization before transformer processing using Layer Normalization.

### Files Modified

`config/rl/moving/locotransformer/thin.json`

### Original Configuration

```json
"net": {
    "transformer_params": [
        [1,256],
        [1,256]
    ],
    "append_hidden_shapes": [
        256,
        256
    ]
}
```

### Proposed Configuration

```json
"net": {
    "transformer_params": [
        [1,256],
        [1,256]
    ],
    "append_hidden_shapes": [
        256,
        256
    ],
    "token_norm": true
}
```

### Motivation

Transformer architectures frequently benefit from feature normalization before attention operations.

The LocoTransformer implementation already contains a token normalization pathway based on Layer Normalization but leaves it disabled by default.

Enabling token normalization standardizes token feature distributions before attention processing, which may improve optimization stability and reduce sensitivity to feature magnitude variations.

This can be particularly useful when combining heterogeneous information sources such as proprioceptive state features and visual terrain features.

### Expected Benefits

* Improved training stability
* Better-conditioned token representations
* Reduced feature-scale sensitivity
* Potentially improved attention behavior

### Expected Drawbacks

* Slight computational overhead
* Possible reduction in useful feature magnitude information

### Implementation Complexity

Very Low

### Code Changes Required

No Python source code modifications are required.

The functionality is already implemented inside:

`torchrl/networks/nets.py`

through:

```python
if self.token_norm:
    self.token_ln = nn.LayerNorm(self.encoder.visual_dim)
```

and

```python
if self.token_norm:
    out = self.token_ln(out)
```

The feature can be enabled entirely through configuration.

### Verification Audit

The `token_norm` parameter is exposed in both:

```python
class Transformer(...)
class LocoTransformer(...)
```

and is forwarded automatically through:

```python
**params["net"]
```

from:

`starter/ppo_locotransformer.py`

No additional code modifications were found to be necessary.



## 6. Post-Transformer Layer Normalization

### Objective

Enable Layer Normalization within the post-transformer MLP head.

### Files Modified

`config/rl/moving/locotransformer/thin.json`

### Original Configuration

```json
"net": {
    "transformer_params": [
        [1,256],
        [1,256]
    ],
    "append_hidden_shapes": [
        256,
        256
    ]
}
```

### Proposed Configuration

```json
"net": {
    "transformer_params": [
        [1,256],
        [1,256]
    ],
    "append_hidden_shapes": [
        256,
        256
    ],
    "add_ln": true
}
```

### Motivation

After transformer processing and token aggregation, the resulting feature vector is passed through a multi-layer perceptron before generating actions and value estimates.

The implementation already contains optional Layer Normalization blocks that can be inserted after each hidden layer of this MLP.

Layer Normalization can stabilize feature distributions during training, improve optimization dynamics, and reduce sensitivity to feature magnitude variations.

Unlike token normalization, which operates on transformer tokens, this modification normalizes features inside the final decision-making MLP.

### Expected Benefits

* More stable hidden activations
* Improved optimization behavior
* Better-conditioned feature representations
* Potentially smoother PPO training

### Expected Drawbacks

* Slight computational overhead
* Possible suppression of useful feature magnitude information

### Implementation Complexity

Very Low

### Code Changes Required

No Python source code modifications are required.

The implementation already exists inside:

`torchrl/networks/nets.py`

through:

```python
if self.add_ln:
    self.visual_append_fcs.append(
        nn.LayerNorm(next_shape)
    )
```

The feature can be enabled entirely through configuration.

### Verification Audit

Inspection confirmed that Layer Normalization is inserted only within the post-transformer MLP stack and does not modify:

* Transformer attention layers
* Visual encoder
* State tokenizer
* PPO implementation

Activation is achieved through:

```json
"add_ln": true
```

within the network configuration.



## 7. Mean + Max Pool Fusion

### Objective

Replace single mean-pooled terrain aggregation with a combined mean-pooled and max-pooled terrain representation.

### Files Modified

`torchrl/networks/nets.py`

### Original Architecture

```text
Terrain Tokens
↓
Mean Pool
↓
State Token + Mean Terrain Token
↓
MLP
↓
Action / Value Output
```

### Proposed Architecture

```text
Terrain Tokens
↓
Mean Pool + Max Pool
↓
State Token + Mean Terrain Token + Max Terrain Token
↓
MLP
↓
Action / Value Output
```

### Motivation

The baseline LocoTransformer aggregates terrain information exclusively through mean pooling. While this captures overall terrain structure, it may dilute highly informative terrain features such as obstacles, edges, gaps, foothold candidates, or abrupt height changes.

By combining mean pooling and max pooling, the network receives both:

* Global terrain information (mean pooling)
* Strongest terrain activations (max pooling)

This allows the policy to simultaneously reason about overall terrain layout and the most critical local terrain features.

The approach is inspired by successful aggregation strategies used in:

* PointNet
* PointNet++
* Point Transformer
* Set Transformer

### Expected Benefits

* Improved obstacle awareness
* Better preservation of salient terrain features
* Richer terrain representations
* Stronger state-terrain feature fusion
* Increased feature diversity entering the policy network

### Expected Drawbacks

* Larger feature vector entering the MLP
* Increased parameter count
* Slightly higher memory usage
* Slightly longer training time

### Implementation Complexity

Medium

### Code Changes Required

The terrain token aggregation logic must be modified to compute both mean-pooled and max-pooled terrain features.

The downstream MLP input dimension must also be adjusted to account for the additional pooled feature vector.

### Verification Audit

Inspection of `torchrl/networks/nets.py` confirmed that:

```python
out_first = out[1: 1 + self.per_modal_tokens, ...].mean(dim=0)
```

is currently the sole terrain aggregation operation.

The resulting terrain representation is concatenated with the state token before entering the final MLP.

Introducing both mean-pooled and max-pooled terrain representations increases the concatenated feature dimension from:

```text
256 (state)
+ 256 (mean terrain)
= 512
```

to:

```text
256 (state)
+ 256 (mean terrain)
+ 256 (max terrain)
= 768
```

and therefore requires corresponding adjustments to the downstream MLP input dimensions.

### Research Basis

Mean-max fusion is widely used in token-set and point-cloud architectures where important local features can be diluted by averaging operations. The technique has demonstrated effectiveness in preserving both global context and highly informative local features.



## 8. Learnable Positional Embeddings

### Objective

Introduce learnable positional embeddings to provide explicit spatial information to transformer tokens.

### Files Modified

`torchrl/networks/base.py`

### Original Architecture

```text
State Token
+
Terrain Tokens
↓
Transformer
```

### Proposed Architecture

```text
State Token
+
Terrain Tokens
+
Learnable Positional Embeddings
↓
Transformer
```

### Motivation

The baseline LocoTransformer architecture constructs a sequence consisting of one state token and multiple terrain tokens before passing them into the transformer encoder.

During inspection, no positional embeddings or positional encoding mechanisms were found within the tokenization or transformer pipeline.

As a result, the transformer receives token content but lacks explicit information regarding the spatial location of terrain patches.

For locomotion tasks, spatial location is highly relevant because the robot must distinguish between terrain features located in different regions of the visual field.

Learnable positional embeddings allow the transformer to associate each token with a unique spatial position, improving its ability to reason about terrain layout and obstacle location.

### Expected Benefits

* Improved spatial awareness
* Better terrain understanding
* More informative attention patterns
* Enhanced state-terrain interaction

### Expected Drawbacks

* Additional trainable parameters
* Slightly increased model complexity

### Implementation Complexity

Medium

### Code Changes Required

A learnable positional embedding tensor is introduced inside the token encoder and added to the token sequence prior to transformer processing.

### Verification Audit

Inspection of `torchrl/networks/base.py` confirmed that:

```python
visual_out = torch.cat(out_list, dim=0)
```

directly constructs the transformer input sequence.

No positional embeddings, positional encodings, or learnable position parameters were found anywhere in the tokenization pipeline.

The transformer therefore receives token content without explicit positional information.

### Research Basis

Positional embeddings are a foundational component of modern transformer architectures, including:

* Transformer (Vaswani et al.)
* Vision Transformer (ViT)
* DeiT
* BEiT
* MAE
* DINO

These methods rely on positional information to allow attention mechanisms to reason about spatial structure and token location.

