# Lesson: Transfer Learning — Frozen Backbone বনাম Fine-Tuning

> **Series:** Stereo Hs Exp V1 — Learning Notes
> **Context:** Stereo, radar আর monocular benchmark-এ ImageNet-pretrained model (ResNet-50, ViT-B/16) কীভাবে ব্যবহার হয়
> **Format:** Concept → Model-এর গঠন → Mathematics → Toy hand calculation → PyTorch code → আমাদের সিদ্ধান্ত → Quiz

---

## Table of Contents

1. [Transfer learning কী](#1-transfer-learning-কী)
2. [Model-এর দুটো অংশ: Backbone আর Head](#2-model-এর-দুটো-অংশ-backbone-আর-head)
3. [তিনটা কৌশল](#3-তিনটা-কৌশল)
4. [Mathematics: gradient কোথায় যায়](#4-mathematics-gradient-কোথায়-যায়)
5. [Toy Example: হাতে হিসাব](#5-toy-example-হাতে-হিসাব)
6. [PyTorch Code](#6-pytorch-code)
7. [Frozen backbone-এর একটা বড় সুবিধা: embedding একবার](#7-frozen-backbone-এর-একটা-বড়-সুবিধা-embedding-একবার)
8. [আমাদের Stereo experiment-এ কোনটা, আর কেন](#8-আমাদের-stereo-experiment-এ-কোনটা-আর-কেন)
9. [Paper-এ সঠিক শব্দ](#9-paper-এ-সঠিক-শব্দ)
10. [Quiz](#10-quiz)

---

## 1. Transfer learning কী

**Transfer learning** মানে: একটা বড় dataset-এ আগে থেকে train করা model নিয়ে নিজের (ছোট) dataset-এর কাজে ব্যবহার করা।

- ImageNet: ~12 লাখ ছবি, 1000টা class (কুকুর, গাড়ি, পাখি…)।
- ওখানে train হওয়া model আগে থেকেই শিখে ফেলেছে: edge, corner, texture, repeating pattern, shape।
- Sea surface image-এও edge (wave crest), texture (ripple) আর pattern (wave group) আছে। তাই ImageNet-এর শেখা জ্ঞান আমাদের কাজে লাগতে পারে।

**মূল কথা:** দুটো কৌশলেই **আমাদের নিজের data দিয়ে training হয়।** পার্থক্য শুধু **কোন weight বদলানোর অনুমতি আছে।**

```
                    Transfer Learning
                           │
          ┌────────────────┼────────────────────┐
          │                │                    │
 (A) Frozen backbone   (B) Full fine-tuning  (C) Two-stage
     শুধু head train       backbone + head       আগে A, তারপর
                           সব train              backbone-এর
                                                 শেষ অংশ খোলা
```

---

## 2. Model-এর দুটো অংশ: Backbone আর Head

```
Image ──► [ BACKBONE ] ──► feature vector ──► [ HEAD ] ──► Hs
          ResNet-50 conv layers  (2048 সংখ্যা)     regression layer
          ~23.5 million weight                     Linear: 2049 weight
          ImageNet থেকে শেখা                         নতুন, random শুরু
```

| অংশ | কাজ | Weight সংখ্যা (ResNet-50) | কোথা থেকে আসে |
|---|---|---|---|
| **Backbone** | ছবি থেকে feature বের করা | ~23.5 million | ImageNet pretrained |
| **Head** | Feature থেকে $H_s$ বের করা | Linear: 2048 + 1 bias = **2049** | Random, আমাদের data দিয়ে শিখতে হবে |

- ImageNet-এর আসল head ছিল 1000-class classifier। সেটা ফেলে দিয়ে আমরা **1টা output-এর regression head** বসাই।
- ViT-B/16-এ backbone ~86 million weight, আর feature vector 768-মাত্রার।

---

## 3. তিনটা কৌশল

| | (A) Frozen backbone | (B) Full fine-tuning | (C) Two-stage |
|---|---|---|---|
| Backbone-এর weight | **বদলায় না** | বদলায় (ছোট learning rate) | প্রথমে না, পরে শেষের কিছু layer |
| Head-এর weight | Train হয় | Train হয় | Train হয় |
| কতগুলো weight শেখে | ~2 হাজার | ~23.5 million | মাঝামাঝি |
| কম data-য় | ✅ নিরাপদ | ⚠️ overfitting ঝুঁকি বেশি | ⚠️ মাঝামাঝি |
| Compute | কম (CPU-তেও চলে) | বেশি (GPU লাগে) | বেশি |
| Backbone নতুন domain-এ খাপ খায়? | না | হ্যাঁ | আংশিক |
| অন্য নাম | Feature extraction, linear probing | Fine-tuning | Gradual unfreezing |

**সাধারণ নিয়ম:**
- Data অনেক আর domain ImageNet থেকে অনেক আলাদা হলে (B) বা (C) ভালো।
- Data কম হলে (A) নিরাপদ।

---

## 4. Mathematics: gradient কোথায় যায়

Model-কে দুটো function-এর composition হিসেবে লেখা যায়:

$$\hat{y} = h_{\phi}\big(g_{\theta}(x)\big)$$

- $g_\theta$ = backbone, parameter $\theta$
- $h_\phi$ = head, parameter $\phi$
- Loss: $L = (\hat y - y)^2$

**Chain rule দিয়ে gradient:**

$$\frac{\partial L}{\partial \phi} = \frac{\partial L}{\partial \hat y}\cdot\frac{\partial \hat y}{\partial \phi}$$

$$\frac{\partial L}{\partial \theta} = \frac{\partial L}{\partial \hat y}\cdot\frac{\partial \hat y}{\partial g}\cdot\frac{\partial g}{\partial \theta}$$

**Update rule** (gradient descent, learning rate $\eta$):

| | (A) Frozen | (B) Fine-tuning |
|---|---|---|
| Head | $\phi \leftarrow \phi - \eta\,\dfrac{\partial L}{\partial \phi}$ | $\phi \leftarrow \phi - \eta\,\dfrac{\partial L}{\partial \phi}$ |
| Backbone | $\theta \leftarrow \theta$ **(কোনো পরিবর্তন নেই)** | $\theta \leftarrow \theta - \eta_{\text{small}}\,\dfrac{\partial L}{\partial \theta}$ |

Frozen অবস্থায় backbone-এর gradient হিসাবই করা হয় না (`requires_grad = False`), তাই memory আর সময় দুটোই বাঁচে।

---

## 5. Toy Example: হাতে হিসাব

সবচেয়ে ছোট model: backbone-এ একটা weight $W_1$, head-এ একটা weight $w_2$।

$$z = W_1\,x \quad \text{(backbone)}, \qquad \hat y = w_2\,z \quad \text{(head)}$$

**শুরুর মান:**

| Symbol | মান | অর্থ |
|---|---|---|
| $x$ | 2 | Input (ছবির জায়গায় একটা সংখ্যা) |
| $W_1$ | 0.5 | Backbone weight ("ImageNet থেকে শেখা") |
| $w_2$ | 1 | Head weight (নতুন) |
| $y$ | 3 | Target ($H_s$) |
| $\eta$ | 0.1 | Learning rate |

### Step 1: Forward pass

$$z = W_1 x = 0.5 \times 2 = 1$$
$$\hat y = w_2 z = 1 \times 1 = 1$$
$$L = (\hat y - y)^2 = (1 - 3)^2 = 4$$

### Step 2: Gradient (chain rule)

$$\frac{\partial L}{\partial \hat y} = 2(\hat y - y) = 2(1 - 3) = -4$$

Head-এর জন্য:
$$\frac{\partial L}{\partial w_2} = \frac{\partial L}{\partial \hat y}\cdot z = (-4)(1) = -4$$

Backbone-এর জন্য:
$$\frac{\partial L}{\partial W_1} = \frac{\partial L}{\partial \hat y}\cdot w_2 \cdot x = (-4)(1)(2) = -8$$

### Step 3: একটা update step

নিয়ম: $\text{new} = \text{old} - \eta \times \text{gradient}$

| | (A) Frozen | (B) Fine-tuning |
|---|---|---|
| $w_2$ (head) | $1 - 0.1(-4) = \mathbf{1.4}$ | $1 - 0.1(-4) = \mathbf{1.4}$ |
| $W_1$ (backbone) | **0.5** (বদলায় না) | $0.5 - 0.1(-8) = \mathbf{1.3}$ |

### Step 4: নতুন prediction

| | (A) Frozen | (B) Fine-tuning |
|---|---|---|
| $z$ | $0.5 \times 2 = 1.0$ | $1.3 \times 2 = 2.6$ |
| $\hat y$ | $1.4 \times 1.0 = \mathbf{1.40}$ | $1.4 \times 2.6 = \mathbf{3.64}$ |
| নতুন Loss | $(1.40 - 3)^2 = 2.56$ | $(3.64 - 3)^2 = 0.41$ |

### এখান থেকে কী শিখলাম

- **Frozen:** শুধু head বদলেছে। Backbone ছবিকে আগের মতোই দেখে ($z$ এখনো 1)।
- **Fine-tuning:** backbone-ও বদলেছে। Loss অনেক দ্রুত কমেছে, এমনকি target ছাড়িয়েও গেছে (3.64 > 3)।
- **বাস্তবে:** ResNet-50-এ backbone-এ 23.5 million weight। মাত্র কয়েকটা আলাদা label থাকলে এতগুলো weight সহজেই training data **মুখস্থ** করে ফেলে (overfitting), কিন্তু নতুন data-য় খারাপ করে।

---

## 6. PyTorch Code

### (A) Frozen backbone

```python
import torch
import torch.nn as nn
import torchvision

# 1. Load ImageNet-pretrained ResNet-50
model = torchvision.models.resnet50(weights="IMAGENET1K_V2")

# 2. Freeze ALL backbone weights
for parameter in model.parameters():
    parameter.requires_grad = False

# 3. Replace the 1000-class head with a 1-output regression head
#    (a new layer is trainable by default)
model.fc = nn.Linear(2048, 1)

# 4. Give ONLY the head to the optimizer
optimizer = torch.optim.AdamW(model.fc.parameters(), lr=1e-3)

# 5. Count trainable weights
trainable = sum(p.numel() for p in model.parameters() if p.requires_grad)
total = sum(p.numel() for p in model.parameters())
print("trainable:", trainable)   # 2049
print("total    :", total)       # ~23.5 million
```

### (B) Full fine-tuning

```python
model = torchvision.models.resnet50(weights="IMAGENET1K_V2")
model.fc = nn.Linear(2048, 1)

# all weights trainable (default); backbone gets a smaller learning rate
backbone_params = [p for name, p in model.named_parameters() if not name.startswith("fc.")]
optimizer = torch.optim.AdamW([
    {"params": backbone_params, "lr": 1e-5},
    {"params": model.fc.parameters(), "lr": 1e-3},
])
```

### Code দেখে কীভাবে বুঝবে কোনটা ব্যবহার হয়েছে

| Code-এ যা আছে | মানে |
|---|---|
| `requires_grad = False` backbone-এ, আর training-এর শেষ পর্যন্ত তা-ই থাকে | **(A) Frozen** |
| কোথাও `requires_grad = False` নেই | **(B) Fine-tuning** |
| প্রথমে `False`, পরে কিছু layer-এ আবার `True` | **(C) Two-stage** |
| Optimizer-এ শুধু `model.fc.parameters()` | (A) |

---

## 7. Frozen backbone-এর একটা বড় সুবিধা: embedding একবার

Backbone না বদলালে একই ছবির feature vector সবসময় একই থাকে। তাই:

1. প্রতিটা ছবির embedding (ResNet-50: 2048 সংখ্যা) **একবারই** বের করে disk-এ রাখা যায়।
2. LORO-র প্রতিটা fold-এ শুধু ছোট head train করতে হয়, যা কয়েক সেকেন্ডের কাজ।

| | Fine-tuning | Frozen |
|---|---|---|
| Backbone কতবার চালাতে হয় | প্রতি fold × seed × epoch × ছবি | **প্রতি ছবিতে একবার** |
| ৮ fold × ৩ seed × ২০ epoch | ~480 বার পুরো dataset | 1 বার |
| GPU ছাড়া সম্ভব? | কার্যত না | ✅ হ্যাঁ (কয়েক ঘণ্টা) |

---

## 8. আমাদের Stereo experiment-এ কোনটা, আর কেন

**সিদ্ধান্ত: (A) Frozen backbone**

| কারণ | ব্যাখ্যা |
|---|---|
| **Label খুব কম** | Main set-এ মাত্র ৮টা আলাদা $H_s$। 23.5 million weight বদলাতে দিলে model সহজে "কোন station" মুখস্থ করবে |
| **তিন modality-তে একই setting** | Radar frozen, monocular frozen। Stereo-ও frozen হলে তুলনা ন্যায্য |
| **Compute** | Embedding একবার বের করলেই হয়, CPU-তেও সম্ভব |

**Stereo-তে embedding কীভাবে ব্যবহার হবে:**

- cam01 → $f_A$ (2048), cam02 → $f_B$ (2048)
- **L (monocular-সমতুল্য):** head-এর input = $f_A$
- **Stereo (order-invariant):** head-এর input = $\left[\dfrac{f_A + f_B}{2},\ |f_A - f_B|\right]$ (4096)
  - A আর B অদলবদল করলেও input একই থাকে, তাই left/right-এর অনিশ্চয়তা ফলাফলে প্রভাব ফেলে না।

---

## 9. Paper-এ সঠিক শব্দ

| ✅ ঠিক | ❌ বিভ্রান্তিকর |
|---|---|
| "frozen ImageNet-pretrained backbone with a trained regression head" | "fine-tuned ResNet-50" (যদি backbone frozen থাকে) |
| "used as a fixed feature extractor" | "transfer learning" (একা লিখলে অস্পষ্ট) |
| "linear probing" (head linear হলে) | |

**উদাহরণ বাক্য:**
> *"ImageNet-pretrained ResNet-50 and ViT-B/16 backbones are used as frozen feature extractors, and only a regression head is trained on the target data."*

---

## 10. Quiz

1. Frozen backbone-এ কি আমাদের data দিয়ে কোনো training হয়? হলে কোন অংশের?
2. ResNet-50-এর head যদি `nn.Linear(2048, 1)` হয়, তাহলে কয়টা trainable weight আছে? হিসাব দেখাও।
3. Toy example-এ $x = 1$, $W_1 = 2$, $w_2 = 0.5$, $y = 2$, $\eta = 0.1$ ধরে একটা update step করো। Frozen আর fine-tuning দুই ক্ষেত্রেই নতুন $\hat y$ বের করো।
4. আমাদের stereo experiment-এ মাত্র ৮টা record। Full fine-tuning করলে কী সমস্যা হতে পারে?
5. Frozen backbone-এ embedding একবার বের করে রাখা যায় কেন, কিন্তু fine-tuning-এ যায় না কেন?
6. কোনো code-এ প্রথম 5 epoch `requires_grad = False`, তারপর `layer4`-এর জন্য `True` করা হয়েছে। এটা কোন কৌশল? Paper-এ কী লিখবে?

<details>
<summary>Answers</summary>

1. হ্যাঁ। শুধু **head**-এর weight আমাদের data দিয়ে train হয়। Backbone-এর weight বদলায় না।

2. 2048টা input weight + 1টা bias = **2049**।

3. Forward: $z = 2 \times 1 = 2$, $\hat y = 0.5 \times 2 = 1$, $L = (1-2)^2 = 1$।
   $\partial L/\partial \hat y = 2(1-2) = -2$।
   $\partial L/\partial w_2 = -2 \times 2 = -4$; $\partial L/\partial W_1 = -2 \times 0.5 \times 1 = -1$।
   - Frozen: $w_2 = 0.5 + 0.4 = 0.9$, $W_1 = 2$ → $\hat y = 0.9 \times 2 = \mathbf{1.8}$।
   - Fine-tuning: $w_2 = 0.9$, $W_1 = 2 + 0.1 = 2.1$ → $\hat y = 0.9 \times 2.1 = \mathbf{1.89}$।

4. 23.5 million weight শেখার জন্য মাত্র ৮টা আলাদা label আছে। Model station-এর চেহারা (platform, আলো, camera angle) মুখস্থ করে ফেলতে পারে, আর অদেখা record-এ খারাপ করবে (overfitting / shortcut learning)।

5. Frozen অবস্থায় backbone-এর weight কখনো বদলায় না, তাই একই ছবির output সবসময় একই থাকে। Fine-tuning-এ প্রতিটা update-এর পর backbone বদলায়, ফলে একই ছবির embedding-ও বদলে যায়। তাই আগে থেকে হিসাব করে রাখা যায় না।

6. **Two-stage (gradual unfreezing)।** এটা fine-tuning-এর একটা রূপ, frozen না। Paper-এ লিখবে: *"The head was first trained with a frozen backbone for 5 epochs; layer4 was then unfrozen and fine-tuned with a reduced learning rate."*

</details>
