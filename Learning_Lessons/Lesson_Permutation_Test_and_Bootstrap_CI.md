# Lesson: Permutation Test আর Bootstrap Confidence Interval

> **Series:** Stereo Hs Exp V1 — Learning Notes
> **Context:** S8-এর ফল কীভাবে পড়তে হয়: null median, p-value, 95% CI, ΔRMSE vs shortcut
> **Format:** সংজ্ঞা → Symbol → সূত্র → Toy example (হাতে হিসাব) → আমাদের আসল ফলের ব্যাখ্যা → Quiz

---

## Table of Contents

**Part A — Permutation test**
1. [মূল প্রশ্ন: "ভালো ফল কি কাকতালীয়?"](#1-মূল-প্রশ্ন-ভালো-ফল-কি-কাকতালীয়)
2. [সংজ্ঞা আর symbol](#2-সংজ্ঞা-আর-symbol)
3. [Toy example: দুটো model, ৪টা record (হাতে হিসাব)](#3-toy-example-দুটো-model-৪টা-record-হাতে-হিসাব)
4. [Global null বনাম Within-setup null](#4-global-null-বনাম-within-setup-null)
5. [আমাদের Ridge ফলের ব্যাখ্যা](#5-আমাদের-ridge-ফলের-ব্যাখ্যা)

**Part B — Bootstrap confidence interval**

6. [Confidence Interval (CI) কী](#6-confidence-interval-ci-কী)
7. [Bootstrap: ধারণা আর ধাপ](#7-bootstrap-ধারণা-আর-ধাপ)
8. [Toy example: ৪টা record দিয়ে bootstrap (হাতে হিসাব)](#8-toy-example-৪টা-record-দিয়ে-bootstrap-হাতে-হিসাব)
9. [Paired bootstrap: ΔRMSE vs shortcut](#9-paired-bootstrap-δrmse-vs-shortcut)
10. [আমাদের আসল ফলের ব্যাখ্যা](#10-আমাদের-আসল-ফলের-ব্যাখ্যা)

**Part C**

11. [Permutation বনাম Bootstrap: পার্থক্য এক নজরে](#11-permutation-বনাম-bootstrap-পার্থক্য-এক-নজরে)
12. [Paper-এ কীভাবে লিখবে](#12-paper-এ-কীভাবে-লিখবে)
13. [Quiz](#13-quiz)

---

# Part A — Permutation Test

## 1. মূল প্রশ্ন: "ভালো ফল কি কাকতালীয়?"

ধরো model-এর RMSE এলো 0.43 m। এটা ভালো নাকি খারাপ? তুলনার জন্য একটা প্রশ্ন করা যায়:

> **"যদি image আর $H_s$-এর মধ্যে কোনো সম্পর্কই না থাকত, তাহলে এই একই pipeline কত RMSE দিত?"**

এই প্রশ্নের উত্তর পাওয়ার উপায়: **label-গুলো এলোমেলো (shuffle) করে দেওয়া।** তাহলে image আর label-এর আসল সম্পর্ক ভেঙে যায়। তারপর পুরো experiment আবার চালানো হয়, আর বারবার করা হয়। আসল RMSE যদি এলোমেলো label-এর RMSE-গুলোর চেয়ে স্পষ্টভাবে ভালো হয়, তাহলে model সত্যিই কিছু শিখেছে।

---

## 2. সংজ্ঞা আর symbol

| Symbol / শব্দ | অর্থ |
|---|---|
| $H_0$ (**null hypothesis**) | "Model-এর কোনো আসল skill নেই। ফলটা কাকতালীয়।" |
| $\pi$ (**permutation**) | Label-গুলোর একটা এলোমেলো সাজানো |
| $\text{RMSE}_{\text{real}}$ | আসল label দিয়ে পাওয়া RMSE |
| $\text{RMSE}_{\pi}$ | Permutation $\pi$-এর label দিয়ে পুরো LORO আবার চালিয়ে পাওয়া RMSE |
| **Null distribution** | সব $\text{RMSE}_{\pi}$-এর সংগ্রহ। $H_0$ সত্য হলে RMSE কেমন হতো, তার ছবি |
| **Null median** | Null distribution-এর মাঝের মান। "সাধারণ কাকতালীয় ফল" |
| $N$ | মোট permutation-এর সংখ্যা |
| **p-value** | $H_0$ সত্য হলে আসল ফলের মতো বা তার চেয়ে ভালো ফল পাওয়ার সম্ভাবনা |

**p-value-এর সূত্র:**

$$p = \frac{\#\{\pi : \text{RMSE}_{\pi} \le \text{RMSE}_{\text{real}}\} + 1}{N + 1}$$

- **লব:** কতগুলো এলোমেলো ফল আসল ফলের সমান বা তার চেয়ে ভালো (RMSE ছোট বা সমান), তার সাথে +1।
- **+1 কেন:** আসল labeling-ও একটা সম্ভাব্য সাজানো। +1 যোগ করলে $p$ কখনো ঠিক 0 হয় না। এটা সৎ, conservative নিয়ম।

**পড়ার নিয়ম:**

| $p$ | অর্থ |
|---|---|
| < 0.05 | আসল ফল কাকতালীয় হওয়ার সম্ভাবনা কম। Model কিছু শিখেছে ✅ |
| ≥ 0.05 | কাকতালীয় ফল থেকে আলাদা করা যায় না ❌ |
| ≈ 0.5 | আসল ফল ঠিক "সাধারণ কাকতালীয়" ফলের মতো, অর্থাৎ real ≈ null median |

---

## 3. Toy example: দুটো model, ৪টা record (হাতে হিসাব)

### 3.1 Setup

দুটো station, প্রতিটায় ২টা record:

| Station | Record | True $H_s$ (m) |
|---|---|---|
| A | r1 | 0.3 |
| A | r2 | 0.5 |
| B | r3 | 1.4 |
| B | r4 | 1.8 |

দুটো model-এর prediction:

| Record | Model 1 ("wave-aware") | Model 2 ("station-only") |
|---|---|---|
| r1 | 0.35 | 0.40 |
| r2 | 0.45 | 0.40 |
| r3 | 1.50 | 1.60 |
| r4 | 1.70 | 1.60 |

- **Model 1** একই station-এর ভেতরে ক্রম ঠিক ধরে: r1 < r2, r3 < r4।
- **Model 2** শুধু station চেনে: A-র সবাইকে 0.40, B-র সবাইকে 1.60 দেয়।

> **সরলীকরণ:** এখানে prediction স্থির ধরে নিয়েছি। আসল script-এ প্রতিটা permutation-এর জন্য model নতুন করে train হয়, কিন্তু যুক্তি একই।

### 3.2 Within-setup permutation: সব সাজানো

শুধু একই station-এর ভেতরে label অদলবদল করা হবে। A-তে 2! = 2 উপায়, B-তে 2! = 2 উপায়, মোট **2 × 2 = 4টা permutation:**

| π | A-র label (r1, r2) | B-র label (r3, r4) |
|---|---|---|
| π₁ (আসল) | 0.3, 0.5 | 1.4, 1.8 |
| π₂ | 0.3, 0.5 | **1.8, 1.4** |
| π₃ | **0.5, 0.3** | 1.4, 1.8 |
| π₄ | **0.5, 0.3** | **1.8, 1.4** |

### 3.3 Model 1-এর RMSE, প্রতিটা permutation-এ

**π₁ (আসল label):**

| Record | Pred | True | $e$ | $e^2$ |
|---|---|---|---|---|
| r1 | 0.35 | 0.3 | +0.05 | 0.0025 |
| r2 | 0.45 | 0.5 | −0.05 | 0.0025 |
| r3 | 1.50 | 1.4 | +0.10 | 0.0100 |
| r4 | 1.70 | 1.8 | −0.10 | 0.0100 |
| | | | **Sum** | **0.0250** |

$$\text{RMSE}_{\text{real}} = \sqrt{0.0250/4} = \sqrt{0.00625} = \mathbf{0.079}$$

**π₂ (B অদলবদল):** r1, r2 আগের মতো (0.0025 + 0.0025)। r3: 1.50 − 1.8 = −0.30 → 0.09, r4: 1.70 − 1.4 = +0.30 → 0.09।

$$\text{Sum} = 0.005 + 0.18 = 0.185, \quad \text{RMSE} = \sqrt{0.185/4} = \sqrt{0.04625} = 0.215$$

**π₃ (A অদলবদল):** r1: 0.35 − 0.5 = −0.15 → 0.0225, r2: 0.45 − 0.3 = +0.15 → 0.0225। r3, r4 আগের মতো (0.01 + 0.01)।

$$\text{Sum} = 0.045 + 0.02 = 0.065, \quad \text{RMSE} = \sqrt{0.01625} = 0.128$$

**π₄ (দুটোই অদলবদল):** Sum = 0.045 + 0.18 = 0.225, RMSE = $\sqrt{0.05625}$ = 0.237।

**Model 1-এর null distribution:** {0.079, 0.215, 0.128, 0.237}

- সাজালে: 0.079, 0.128, 0.215, 0.237 → **null median** = (0.128 + 0.215)/2 = **0.172**
- আসল RMSE = 0.079 → null median-এর চেয়ে **অনেক ছোট** ✅
- কতগুলো $\le$ 0.079? শুধু π₁ নিজে, অর্থাৎ 1টা।

$$p = \frac{1 + 1}{4 + 1} = \frac{2}{5} = 0.40$$

### 3.4 Model 2-এর RMSE, প্রতিটা permutation-এ

**π₁:** r1: 0.40 − 0.3 = +0.1, r2: 0.40 − 0.5 = −0.1, r3: 1.6 − 1.4 = +0.2, r4: 1.6 − 1.8 = −0.2

$$\text{Sum} = 0.01 + 0.01 + 0.04 + 0.04 = 0.10, \quad \text{RMSE} = \sqrt{0.025} = \mathbf{0.158}$$

**π₂, π₃, π₄:** A-র ভেতরে label অদলবদল করলে error-গুলো শুধু জায়গা বদলায় (+0.1 আর −0.1 অদলবদল হয়), squared error একই থাকে। B-তেও একই কথা। তাই **চারটাতেই RMSE = 0.158।**

**Model 2-এর null distribution:** {0.158, 0.158, 0.158, 0.158}

- **Null median = 0.158 = আসল RMSE**
- কতগুলো $\le$ 0.158? চারটাই।

$$p = \frac{4 + 1}{4 + 1} = \mathbf{1.0}$$

### 3.5 এখান থেকে কী শিখলাম

| | Model 1 (wave-aware) | Model 2 (station-only) |
|---|---|---|
| Real RMSE | 0.079 | 0.158 |
| Null median | 0.172 | 0.158 |
| Real < null median? | ✅ অনেক কম | ❌ সমান |
| p (within) | 0.40 | **1.0** |

1. **Station-only model-এর আসল RMSE সবসময় within-setup null median-এর সমান।** কারণ এই model station-এর ভেতরের label-এর ক্রম দেখেই না।
2. **Wave-aware model-এর আসল RMSE null median-এর চেয়ে কম।** এলোমেলো করলে ক্রম ভেঙে যায় আর ফল খারাপ হয়।
3. ⚠️ **Model 1-এর p = 0.40, অর্থাৎ < 0.05 না!** কারণ মাত্র ৪টা permutation আছে। সবচেয়ে ছোট সম্ভাব্য p হলো $\frac{1+1}{4+1} = 0.40$। **Permutation কম হলে significance পাওয়াই সম্ভব না।**
   - আমাদের আসল data-য় 144টা permutation, তাই সবচেয়ে ছোট সম্ভাব্য p = $\frac{2}{145}$ ≈ 0.014। তাই সেখানে significance পাওয়া সম্ভব।

---

## 4. Global null বনাম Within-setup null

| | Global null | Within-setup null |
|---|---|---|
| কীভাবে shuffle | সব record-এর label এলোমেলো | শুধু **একই station-এর ভেতরে** |
| Station-এর তথ্য | ভেঙে যায় | **অক্ষত থাকে** |
| কী test করে | "Image-এ label-এর **কোনো** তথ্য আছে?" | "Station-এর **বাইরে** wave থেকে তথ্য আছে?" |
| Station-only model পাস করবে? | ✅ হ্যাঁ (station চিনলেই যথেষ্ট) | ❌ না |
| আমাদের data-য় কয়টা | 1000টা random | 4! × 3! × 1! = **144টা (সবগুলো)** |

**উদাহরণ:** Global shuffle-এ toy example-এর A station-এর record পেতে পারে B-র বড় label (1.8)। তখন station-only model-ও খুব ভুল করবে। তাই global test-এ station-only model "significant" দেখাবে, কিন্তু সেটা wave শেখার প্রমাণ না।

**তাই H1-এর ("model wave দেখে শেখে") আসল test হলো within-setup null।**

---

## 5. আমাদের Ridge ফলের ব্যাখ্যা

```
real_rmse_m          : 0.4306
within_null_median_m : 0.4309   ← প্রায় হুবহু এক
p_within             : 0.497
global_null_median_m : 0.873
p_global             : 0.012
```

| Test | পর্যবেক্ষণ | অর্থ |
|---|---|---|
| **Within** | Real (0.4306) ≈ null median (0.4309), p = 0.50 | Toy-এর **Model 2**-এর মতো। একই station-এর ভেতরে label এলোমেলো করলেও ফল বদলায় না। **Wave থেকে কোনো তথ্য নেই** ❌ |
| **Global** | Real (0.43) ≪ null median (0.87), p = 0.012 | Image-এ label-এর তথ্য আছে, কিন্তু within test দেখাচ্ছে সেটা **station-এর তথ্য** |

**কেন p ঠিক 1.0 না হয়ে 0.5?** Toy-তে prediction স্থির ছিল। আসল script-এ প্রতিটা permutation-এ model নতুন করে train হয়, ফলে RMSE সামান্য ওঠানামা করে (null-এর কিছু মান একটু বেশি, কিছু একটু কম)। আসল মান ঠিক মাঝখানে পড়েছে, তাই p ≈ 0.5। অর্থ একই: **এলোমেলো ফল থেকে আলাদা করা যায় না।**

---

# Part B — Bootstrap Confidence Interval

## 6. Confidence Interval (CI) কী

আমরা RMSE হিসাব করেছি মাত্র ৮টা record দিয়ে। অন্য ৮টা record হলে RMSE অন্য রকম আসত। **CI বলে আমাদের RMSE-এর মান কতটা অনিশ্চিত।**

| Symbol | অর্থ |
|---|---|
| $\hat\theta$ | আমাদের data থেকে পাওয়া মান (যেমন RMSE = 0.360) |
| $[L, U]$ | Confidence interval: নিচের সীমা $L$, উপরের সীমা $U$ |
| 95% | যদি একই পদ্ধতিতে অনেকবার নতুন data সংগ্রহ করে CI বানানো হতো, তার ~95% আসল মানকে ধারণ করত |

**লেখার ধরন:** RMSE = 0.360 [0.153, 0.506] m

**সহজভাবে পড়া:** "আমাদের সেরা অনুমান 0.360 m, কিন্তু data কম বলে আসল মান সম্ভবত 0.15 থেকে 0.51 m-এর মধ্যে যেকোনো কিছু হতে পারে।"

- **CI চওড়া** → অনিশ্চয়তা বেশি (data কম)।
- **CI সরু** → অনুমান নির্ভরযোগ্য।

---

## 7. Bootstrap: ধারণা আর ধাপ

**সমস্যা:** নতুন data সংগ্রহ করা সম্ভব না (আর নতুন stereo record নেই)।

**Bootstrap-এর কৌশল:** আমাদের হাতে থাকা data থেকেই **with replacement** বারবার নতুন "নকল dataset" বানানো।

**With replacement মানে:** একটা record তোলার পর আবার থলেতে ফেরত দেওয়া হয়। তাই একই record একাধিকবার আসতে পারে, আর কোনো record একবারও না আসতে পারে।

**ধাপ:**

1. $n$টা record থেকে with replacement $n$টা তোলো → একটা bootstrap sample।
2. সেই sample-এর RMSE হিসাব করো → $\text{RMSE}^{*}_b$।
3. ধাপ ১–২ $B$ বার করো (আমাদের $B$ = 10,000)।
4. $B$টা $\text{RMSE}^{*}_b$ ছোট থেকে বড় সাজাও।
5. **95% CI** = 2.5-তম percentile থেকে 97.5-তম percentile (মাঝের 95%)।

$$\text{CI}_{95\%} = \left[\ \text{RMSE}^{*}_{(2.5\%)},\ \ \text{RMSE}^{*}_{(97.5\%)}\ \right]$$

এই পদ্ধতির নাম **percentile bootstrap**।

---

## 8. Toy example: ৪টা record দিয়ে bootstrap (হাতে হিসাব)

### 8.1 আসল data

Model A-র ৪টা record-এর error ($e = \hat y - y$):

| Index | 0 | 1 | 2 | 3 |
|---|---|---|---|---|
| $e_A$ (m) | +0.1 | −0.2 | +0.3 | −0.6 |
| $e_A^2$ | 0.01 | 0.04 | 0.09 | 0.36 |

$$\text{RMSE}_A = \sqrt{\frac{0.01 + 0.04 + 0.09 + 0.36}{4}} = \sqrt{\frac{0.50}{4}} = \sqrt{0.125} = \mathbf{0.354 \text{ m}}$$

### 8.2 পাঁচটা bootstrap sample, হাতে

| b | তোলা index | $e^2$ | Sum | RMSE* |
|---|---|---|---|---|
| 1 | 0, 1, 2, 3 | .01, .04, .09, .36 | 0.50 | $\sqrt{0.125}$ = **0.354** |
| 2 | 0, **0**, 2, 3 | .01, .01, .09, .36 | 0.47 | $\sqrt{0.1175}$ = **0.343** |
| 3 | 1, 1, 3, 3 | .04, .04, .36, .36 | 0.80 | $\sqrt{0.200}$ = **0.447** |
| 4 | 0, 2, 2, 1 | .01, .09, .09, .04 | 0.23 | $\sqrt{0.0575}$ = **0.240** |
| 5 | 3, 3, 3, 0 | .36, .36, .36, .01 | 1.09 | $\sqrt{0.2725}$ = **0.522** |

**লক্ষ্য করো:**
- Sample 4-এ সবচেয়ে বড় error (index 3, −0.6) একবারও আসেনি, তাই RMSE কম (0.240)।
- Sample 5-এ সেটা তিনবার এসেছে, তাই RMSE বেশি (0.522)।
- **এই ওঠানামাই দেখায় RMSE কতটা নির্ভর করে কোন record পড়ল তার উপর।**

### 8.3 সব সম্ভাব্য sample

৪টা record থেকে with replacement ৪টা তোলার $4^4$ = **256টা** সমান-সম্ভাব্য ক্রম আছে। সবগুলো হিসাব করলে:

| | মান |
|---|---|
| 2.5-তম percentile | **0.158** |
| Median | 0.354 |
| 97.5-তম percentile | **0.529** |

$$\text{RMSE}_A = 0.354 \ [0.158,\ 0.529] \text{ m}$$

10,000টা random bootstrap sample নিলেও প্রায় হুবহু একই CI আসে। Data বড় হলে সব sample গোনা অসম্ভব, তাই random 10,000 নেওয়া হয়।

**মাত্র ৪টা record থেকে CI 0.16 থেকে 0.53, অর্থাৎ খুবই চওড়া।** আমাদের ৮টা record-এর CI-ও এজন্যই চওড়া।

---

## 9. Paired bootstrap: ΔRMSE vs shortcut

### 9.1 প্রশ্ন

"Model A কি Model B-র চেয়ে ভালো?" দুটো আলাদা CI মিলিয়ে দেখা দুর্বল পদ্ধতি। ভালো পদ্ধতি হলো **পার্থক্যটার**ই CI বের করা:

$$\Delta = \text{RMSE}_{A} - \text{RMSE}_{B}$$

- $\Delta < 0$ → A ভালো (RMSE কম)।
- **Paired** মানে: প্রতিটা bootstrap sample-এ **একই record-গুলো** দুই model-এর জন্য ব্যবহার করা হয়। এতে তুলনা ন্যায্য হয়, কারণ "কঠিন record পড়েছে" এই প্রভাব দুই model-এর উপর একসাথে পড়ে।

### 9.2 Toy data

| Index | 0 | 1 | 2 | 3 |
|---|---|---|---|---|
| $e_A$ | +0.1 | −0.2 | +0.3 | −0.6 |
| $e_B$ (shortcut) | +0.2 | −0.3 | +0.1 | −0.7 |

$$\text{RMSE}_B = \sqrt{\frac{0.04 + 0.09 + 0.01 + 0.49}{4}} = \sqrt{0.1575} = 0.397$$

$$\Delta = 0.354 - 0.397 = \mathbf{-0.043}$$

A একটু ভালো দেখাচ্ছে। এটা কি নিশ্চিত?

### 9.3 একই ৫টা sample, দুই model-এ

| b | Index | RMSE*_A | RMSE*_B | Δ* |
|---|---|---|---|---|
| 1 | 0,1,2,3 | 0.354 | 0.397 | −0.043 |
| 2 | 0,0,2,3 | 0.343 | 0.381 | −0.038 |
| 3 | 1,1,3,3 | 0.447 | 0.539 | −0.091 |
| 4 | 0,2,2,1 | 0.240 | 0.194 | **+0.046** |
| 5 | 3,3,3,0 | 0.522 | 0.614 | −0.092 |

**হাতে একটা যাচাই (b = 4):** index 0, 2, 2, 1।
- A: $e^2$ = .01, .09, .09, .04 → sum .23 → $\sqrt{.0575}$ = 0.240
- B: $e$ = +0.2, +0.1, +0.1, −0.3 → $e^2$ = .04, .01, .01, .09 → sum .15 → $\sqrt{.0375}$ = 0.194
- Δ = 0.240 − 0.194 = **+0.046**: এই sample-এ B বরং ভালো!

### 9.4 সব 256টা sample থেকে

$$\Delta = -0.043 \ [-0.098,\ +0.105]$$

- ~81% sample-এ Δ < 0 (A ভালো)।
- কিন্তু **CI শূন্যকে ধারণ করে** (−0.098 থেকে +0.105)।

**সিদ্ধান্ত:** A ভালো হতে পারে, কিন্তু ৪টা record দিয়ে **95% নিশ্চয়তায় বলা যায় না।**

### 9.5 পড়ার নিয়ম

| ΔRMSE-এর CI | অর্থ |
|---|---|
| পুরোটা < 0 (যেমন [−0.20, −0.05]) | Model নিশ্চিতভাবে shortcut-এর চেয়ে ভালো ✅ |
| শূন্য ধারণ করে (যেমন [−0.21, +0.01]) | পার্থক্য নিশ্চিত না ⚠️ |
| পুরোটা > 0 | Model নিশ্চিতভাবে shortcut-এর চেয়ে খারাপ ❌ |

---

## 10. আমাদের আসল ফলের ব্যাখ্যা

| Model | RMSE [95% CI] | ΔRMSE vs shortcut [95% CI] | সিদ্ধান্ত |
|---|---|---|---|
| Training mean | 0.752 [0.565, 0.908] | +0.286 [+0.116, +0.508] | পুরোটা > 0 → shortcut-এর চেয়ে **নিশ্চিতভাবে খারাপ** |
| SVR (RBF) | 0.574 [0.288, 0.787] | +0.108 [−0.086, +0.306] | শূন্য ধারণ করে → নিশ্চিত না |
| Ridge | 0.431 [0.244, 0.609] | −0.035 [−0.308, +0.167] | শূন্য ধারণ করে → নিশ্চিত না |
| **ResNet-50** | **0.360 [0.153, 0.506]** | **−0.106 [−0.210, +0.009]** | উপরের সীমা মাত্র +0.009 → **প্রায়, কিন্তু নিশ্চিত না** |
| ViT-B/16 | 0.422 [0.184, 0.593] | −0.044 [−0.125, +0.038] | শূন্য ধারণ করে → নিশ্চিত না |

**মূল বার্তা:**
1. **সব CI চওড়া** (~0.35 m প্রস্থ)। ৮টা record-এ এটাই স্বাভাবিক।
2. **কোনো model 95% নিশ্চয়তায় shortcut-কে হারায়নি।** ResNet-50 সবচেয়ে কাছে গেছে।
3. **Training mean নিশ্চিতভাবে shortcut-এর চেয়ে খারাপ।** অর্থাৎ station চেনা নিজেই অনেক তথ্য দেয়।

---

# Part C

## 11. Permutation বনাম Bootstrap: পার্থক্য এক নজরে

| | Permutation test | Bootstrap CI |
|---|---|---|
| প্রশ্ন | "ফলটা কি কাকতালীয়?" | "ফলটা কতটা অনিশ্চিত?" |
| কী এলোমেলো হয় | **Label** (image ↔ $H_s$-এর সম্পর্ক ভাঙা) | **Record** (with replacement আবার তোলা) |
| Model আবার train হয়? | হ্যাঁ, প্রতিটা permutation-এ | না, আগের prediction ব্যবহার হয় |
| Output | p-value | Interval [L, U] |
| আমাদের কাজে | H1 test (wave বনাম station) | RMSE আর ΔRMSE-এর অনিশ্চয়তা |

---

## 12. Paper-এ কীভাবে লিখবে

> *"Uncertainty was quantified with a percentile bootstrap over records (10,000 resamples). Paired differences to the setup-mean reference used identical resamples for both models. To test whether models exploit wave information beyond station identity, labels were permuted within setups (all 144 permutations) and the full LORO procedure was repeated for each permutation."*

> *"Ridge regression reached an RMSE of 0.43 m, indistinguishable from the within-setup permutation null (median 0.43 m, p = 0.50). No model improved on the setup-mean reference at 95 % confidence."*

---

## 13. Quiz

1. একটা model-এর real RMSE = 0.30, আর 9টা permutation-এর RMSE = {0.25, 0.32, 0.35, 0.38, 0.40, 0.41, 0.45, 0.50, 0.55}। p-value বের করো।
2. Within-setup permutation-এ একটা setup-এ ৫টা record, আরেকটায় ২টা। মোট কয়টা permutation? সবচেয়ে ছোট সম্ভাব্য p কত?
3. ৩টা record-এর error = [+0.1, −0.2, +0.4]। Bootstrap sample (index) [2, 2, 0]-এর RMSE* হাতে বের করো।
4. ΔRMSE = −0.08 [−0.15, −0.02]। Model কি shortcut-এর চেয়ে ভালো? কেন?
5. একটা model global permutation test-এ p = 0.01 আর within-setup test-এ p = 0.6 পেল। Model কী শিখেছে?

<details>
<summary>Answers</summary>

1. $\le$ 0.30: শুধু 0.25, অর্থাৎ 1টা। $p = \frac{1+1}{9+1} = \mathbf{0.20}$। Significant না।

2. $5! \times 2! = 120 \times 2 = 240$টা। আসল সাজানোও এর মধ্যে আছে, তাই সবচেয়ে ছোট $p = \frac{1+1}{240+1} \approx \mathbf{0.0083}$।

3. Index 2, 2, 0 → $e$ = +0.4, +0.4, +0.1 → $e^2$ = 0.16, 0.16, 0.01 → sum 0.33 → $\sqrt{0.33/3} = \sqrt{0.11}$ = **0.332 m**।

4. **হ্যাঁ।** পুরো CI শূন্যের নিচে (−0.15 থেকে −0.02), তাই 95% নিশ্চয়তায় model-এর RMSE shortcut-এর চেয়ে কম।

5. Image-এ label-এর তথ্য আছে (global p ছোট), কিন্তু station-এর ভেতরে wave থেকে কোনো তথ্য নেই (within p বড়)। অর্থাৎ model **station চিনে** predict করছে, wave থেকে $H_s$ শেখেনি। আমাদের Ridge-এর ফল ঠিক এরকম।

</details>
