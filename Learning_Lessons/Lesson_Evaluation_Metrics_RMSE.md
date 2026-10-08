# Lesson: Evaluation Metrics — RMSE এবং বন্ধুরা (হাতে হিসাব)

> **Series:** Stereo Hs Exp V1 — Learning Notes
> **Context:** Stereo $H_s$ benchmark-এর table-এ RMSE, MAE, Bias, $R^2$, CC, SI, MAPE কীভাবে হিসাব হয়
> **Format:** Pipeline → সূত্র → Toy example → আসল data দিয়ে হাতে হিসাব → Code → Quiz

---

## Table of Contents

1. [Stereo-তে prediction কীভাবে তৈরি হয় (৩ ধাপ)](#1-stereo-তে-prediction-কীভাবে-তৈরি-হয়-৩-ধাপ)
2. [ধাপ ২: Frame → Record (median আর seed-গড়)](#2-ধাপ-২-frame--record-median-আর-seed-গড়)
3. [RMSE: সূত্র আর অর্থ](#3-rmse-সূত্র-আর-অর্থ)
4. [Toy example: ৩টা record](#4-toy-example-৩টা-record)
5. [আসল ফল দিয়ে হাতে হিসাব: ResNet-50 LR](#5-আসল-ফল-দিয়ে-হাতে-হিসাব-resnet-50-lr)
6. [বাকি metric একই table থেকে](#6-বাকি-metric-একই-table-থেকে)
7. [Record-level বনাম Frame-level](#7-record-level-বনাম-frame-level)
8. [ছোট n-এ RMSE পড়ার সতর্কতা](#8-ছোট-n-এ-rmse-পড়ার-সতর্কতা)
9. [Python code](#9-python-code)
10. [Quiz](#10-quiz)

---

## 1. Stereo-তে prediction কীভাবে তৈরি হয় (৩ ধাপ)

```
ধাপ ১: প্রতিটা frame (stereo pair)      → model → একটা Hs prediction
ধাপ ২: একটা record-এর ~950 frame       → median → seed-গড় → record-এর একটা prediction
ধাপ ৩: ৮টা record-এর (prediction − truth) → RMSE এবং অন্য metric
```

**কেন record-level?** একটা record-এর সব frame-এর label একটাই ($H_s$ একটা ৩০-মিনিটের statistic)। তাই আসলে আলাদা "প্রশ্ন" আছে মাত্র ৮টা: প্রতিটা record-এর $H_s$ কত।

---

## 2. ধাপ ২: Frame → Record (median আর seed-গড়)

ধরি একটা record-এর আসল $H_s$ = 0.40 m, আর সহজ করার জন্য ধরি ৫টা frame আছে।

**Seed 42-এর prediction:**

| Frame | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| $\hat H_s$ (m) | 0.42 | 0.55 | 0.47 | **0.90** | 0.50 |

**Median:** সাজালে 0.42, 0.47, **0.50**, 0.55, 0.90, তাই median = **0.50 m**।

**Mean নিলে কী হতো:**

$$\frac{0.42 + 0.55 + 0.47 + 0.90 + 0.50}{5} = \frac{2.84}{5} = 0.568 \text{ m}$$

Frame 4 (হয়তো whitecap বা glint) একাই mean-কে 0.07 m উপরে টেনে তুলত। **Median এই ধরনের outlier-কে উপেক্ষা করে।**

**তিনটা seed-এর গড়:** seed 42 → 0.50, seed 43 → 0.48, seed 44 → 0.52

$$\hat H_{s,\text{record}} = \frac{0.50 + 0.48 + 0.52}{3} = \mathbf{0.50 \text{ m}}$$

Seed-এর গড় নেওয়া হয় যাতে random initialization-এর ভাগ্যের উপর ফল নির্ভর না করে।

---

## 3. RMSE: সূত্র আর অর্থ

$$\text{RMSE} = \sqrt{\frac{1}{n}\sum_{i=1}^{n}\left(\hat y_i - y_i\right)^2}$$

**নামটা পেছন থেকে পড়লেই হিসাবের ক্রম:**

| অক্ষর | ধাপ | কাজ |
|---|---|---|
| **E** (Error) | 1 | $e_i = \hat y_i - y_i$ |
| **S** (Square) | 2 | $e_i^2$ |
| **M** (Mean) | 3 | $\frac{1}{n}\sum e_i^2$ |
| **R** (Root) | 4 | $\sqrt{\cdot}$ |

**কেন বর্গ নেওয়া হয়:**
1. চিহ্ন চলে যায়: +0.1 আর −0.1 একে অপরকে বাতিল করে না।
2. **বড় error বেশি শাস্তি পায়:** error 2 গুণ হলে বর্গ 4 গুণ হয়।

**কেন শেষে বর্গমূল:** unit আবার meter-এ ফিরে আসে, তাই RMSE-কে "গড়ে কত মিটার ভুল" হিসেবে পড়া যায় (বড় ভুলকে বেশি গুরুত্ব দিয়ে)।

---

## 4. Toy example: ৩টা record

| Record | True $y$ (m) | Pred $\hat y$ (m) | $e = \hat y - y$ | $e^2$ |
|---|---|---|---|---|
| A | 0.30 | 0.40 | +0.10 | 0.0100 |
| B | 0.50 | 0.45 | −0.05 | 0.0025 |
| C | 1.40 | 1.10 | −0.30 | 0.0900 |
| | | | **Sum** | **0.1025** |

$$\text{Mean} = \frac{0.1025}{3} = 0.03417, \qquad \text{RMSE} = \sqrt{0.03417} = \mathbf{0.185 \text{ m}}$$

**পর্যবেক্ষণ:** C-এর error B-এর 6 গুণ, কিন্তু $e^2$ 36 গুণ (0.09 বনাম 0.0025)। মোট squared error-এর 88% আসছে শুধু C থেকে।

---

## 5. আসল ফল দিয়ে হাতে হিসাব: ResNet-50 LR

(LORO, n = 8, seed-averaged record prediction)

| Record | True (m) | Pred (m) | $e$ | $e^2$ |
|---|---|---|---|---|
| AA01 | 1.360 | 1.267 | −0.093 | 0.0086 |
| AA02 | 1.990 | 1.391 | **−0.599** | **0.3588** |
| AA03 | 1.330 | 1.713 | +0.383 | 0.1467 |
| BS01 | 0.298 | 0.507 | +0.209 | 0.0437 |
| BS02 | 0.394 | 0.492 | +0.098 | 0.0096 |
| BS03 | 0.477 | 0.451 | −0.026 | 0.0007 |
| BS04 | 0.514 | 0.461 | −0.053 | 0.0028 |
| YS01 | 1.940 | 1.259 | **−0.681** | **0.4638** |
| | | | **Sum** | **1.0347** |

$$\text{MSE} = \frac{1.0347}{8} = 0.1293 \text{ m}^2, \qquad \text{RMSE} = \sqrt{0.1293} = \mathbf{0.360 \text{ m}} \ ✅$$

Script-এর output-ও 0.360।

**গুরুত্বপূর্ণ পর্যবেক্ষণ:** AA02 + YS01 = 0.3588 + 0.4638 = 0.8226, অর্থাৎ মোট squared error-এর **~80%** আসে মাত্র এই ২টা record থেকে। দুটোই সবচেয়ে বড় $H_s$-এর record, আর দুটোকেই model কম predict করেছে।

---

## 6. বাকি metric একই table থেকে

### 6.1 MAE (Mean Absolute Error)

$$\text{MAE} = \frac{1}{n}\sum |e_i|$$

$|e|$ = 0.093, 0.599, 0.383, 0.209, 0.098, 0.026, 0.053, 0.681, যার যোগফল 2.142।

$$\text{MAE} = \frac{2.142}{8} = \mathbf{0.268 \text{ m}}$$

বর্গ নেই বলে বড় error-কে আলাদা শাস্তি দেয় না। তাই সবসময় MAE ≤ RMSE।

### 6.2 Bias

$$\text{Bias} = \frac{1}{n}\sum e_i = \frac{-0.762}{8} = \mathbf{-0.095 \text{ m}}$$

Negative মানে গড়ে **underestimate**। চিহ্নের নিয়ম radar আর monocular table-এর মতোই: prediction − truth।

### 6.3 $R^2$ (coefficient of determination)

$$R^2 = 1 - \frac{\sum e_i^2}{\sum (y_i - \bar y)^2}$$

**Step 1:** $\bar y = \dfrac{8.303}{8} = 1.038$ m

**Step 2:** $(y_i - \bar y)^2$:

| Record | $y_i - \bar y$ | $(y_i - \bar y)^2$ |
|---|---|---|
| AA01 | 0.322 | 0.1038 |
| AA02 | 0.952 | 0.9065 |
| AA03 | 0.292 | 0.0853 |
| BS01 | −0.740 | 0.5474 |
| BS02 | −0.644 | 0.4146 |
| BS03 | −0.561 | 0.3146 |
| BS04 | −0.524 | 0.2744 |
| YS01 | 0.902 | 0.8138 |
| | **Sum** | **3.4605** |

**Step 3:**

$$R^2 = 1 - \frac{1.0347}{3.4605} = 1 - 0.299 = \mathbf{0.701}$$

**অর্থ:** "সবসময় গড় ($\bar y$) predict করি" এই নিয়মের তুলনায় model squared error-কে 70% কমিয়েছে। $R^2 < 0$ মানে model গড়ের চেয়েও খারাপ (যেমন Training mean baseline-এর LORO-তে −0.31)।

### 6.4 CC (Pearson correlation)

$$\text{CC} = \frac{\sum (y_i - \bar y)(\hat y_i - \bar{\hat y})}{\sqrt{\sum (y_i - \bar y)^2}\sqrt{\sum (\hat y_i - \bar{\hat y})^2}} = \mathbf{0.859}$$

Prediction আর truth একসাথে ওঠানামা করে কিনা মাপে, কিন্তু scale বা offset-এর ভুল ধরে না। যেমন সব prediction-কে দ্বিগুণ করলেও CC একই থাকে।

### 6.5 SI (Scatter Index)

$$\text{SI} = \frac{\text{RMSE}}{\bar y} = \frac{0.360}{1.038} = \mathbf{0.346}$$

গড় $H_s$-এর তুলনায় error কত বড়। Wave community-তে এটা প্রচলিত, কারণ এতে বিভিন্ন sea state-এর ফল তুলনা করা যায়।

### 6.6 MAPE

$$\text{MAPE} = \frac{100}{n}\sum \frac{|e_i|}{y_i}$$

| Record | AA01 | AA02 | AA03 | BS01 | BS02 | BS03 | BS04 | YS01 |
|---|---|---|---|---|---|---|---|---|
| $\lvert e\rvert / y$ (%) | 6.8 | 30.1 | 28.8 | **70.1** | 24.9 | 5.5 | 10.3 | 35.1 |

$$\text{MAPE} = \frac{211.6}{8} = \mathbf{26.5\%}$$

**লক্ষ্য করো:** RMSE-এ BS01-এর অবদান ছোট (0.0437), কিন্তু MAPE-এ সবচেয়ে বড় (70%)। কারণ ছোট $H_s$ (0.30 m)-এ 0.21 m ভুল আপেক্ষিকভাবে অনেক বড়। **RMSE বড় wave-এর ভুলে সংবেদনশীল, MAPE ছোট wave-এর ভুলে।**

---

## 7. Record-level বনাম Frame-level

| | Record-level | Frame-level |
|---|---|---|
| $n$ | 8 | ~7,700 |
| প্রতিটা $\hat y$ | Record-এর median prediction | প্রতিটা frame-এর prediction |
| প্রতিটা $y$ | Record-এর $H_s$ | একই (record-এর $H_s$) |
| ResNet-50 LR RMSE | 0.360 m | 0.377 m |
| কখন ব্যবহার | **মূল ফল (stereo)** | Radar আর monocular-এর sample-level table-এর সাথে তুলনা |

Frame-level RMSE সাধারণত একটু বেশি, কারণ median frame-গুলোর মধ্যকার ওঠানামা মুছে দেয়।

⚠️ **সতর্কতা:** একই record-এর frame-গুলো একে অপরের উপর নির্ভরশীল (একই sea state, একই label)। তাই $n$ = 7,700 দেখে confidence interval খুব সরু মনে হলেও আসল স্বাধীন তথ্য মাত্র ৮টা record-এর।

---

## 8. ছোট n-এ RMSE পড়ার সতর্কতা

1. **কয়েকটা record পুরো RMSE নির্ধারণ করে।** এখানে ২টা record দিয়েই 80%।
2. **Bootstrap CI দরকার:** ৮টা record থেকে বারবার with-replacement resample করে RMSE-এর অনিশ্চয়তা মাপা হয়।
3. **Baseline-এর সাথে তুলনা দরকার:** RMSE 0.360 ভালো নাকি খারাপ, সেটা বোঝা যায় শুধু Training mean (0.752) আর Setup-mean shortcut-এর (0.466) সাথে তুলনা করে।
4. **Overall RMSE একা বলে না model কী শিখেছে।** Station চিনে ফেলাও কম RMSE দিতে পারে। এজন্য within-setup Spearman আর permutation test লাগে।

---

## 9. Python code

```python
import numpy as np

true = np.array([1.360, 1.990, 1.330, 0.298, 0.394, 0.477, 0.514, 1.940])
pred = np.array([1.267, 1.391, 1.713, 0.507, 0.492, 0.451, 0.461, 1.259])

error = pred - true                       # step 1: error
squared = error ** 2                      # step 2: square
mse = np.mean(squared)                    # step 3: mean
rmse = np.sqrt(mse)                       # step 4: root

mae = np.mean(np.abs(error))
bias = np.mean(error)
r2 = 1 - np.sum(squared) / np.sum((true - true.mean()) ** 2)
cc = np.corrcoef(true, pred)[0, 1]
si = rmse / np.mean(true)
mape = 100 * np.mean(np.abs(error) / true)

print("RMSE", round(rmse, 3))    # 0.360
print("MAE ", round(mae, 3))     # 0.268
print("Bias", round(bias, 3))    # -0.095
print("R2  ", round(r2, 3))      # 0.701
print("CC  ", round(cc, 3))      # 0.859
print("SI  ", round(si, 3))      # 0.346
print("MAPE", round(mape, 1))    # 26.5
```

---

## 10. Quiz

1. True = [0.5, 1.0, 2.0], Pred = [0.6, 0.8, 1.5]। RMSE আর MAE হাতে বের করো। কোনটা বড়, আর কেন?
2. একটা model সব record-এর জন্য ঠিক $\bar y$ predict করে। তার $R^2$ কত?
3. সব prediction-এ +0.2 m যোগ করলে CC আর Bias কীভাবে বদলায়?
4. Section 5-এর table-এ AA02-এর prediction যদি 1.99 m (নিখুঁত) হতো, নতুন RMSE কত হতো?
5. RMSE ছোট হলেও কেন বলা যায় না যে model wave দেখে $H_s$ শিখেছে?

<details>
<summary>Answers</summary>

1. $e$ = [+0.1, −0.2, −0.5], $e^2$ = [0.01, 0.04, 0.25], sum = 0.30। RMSE = $\sqrt{0.30/3} = \sqrt{0.10}$ = **0.316 m**। MAE = (0.1 + 0.2 + 0.5)/3 = **0.267 m**। RMSE বড়, কারণ বর্গ নেওয়ায় বড় error (0.5) বেশি গুরুত্ব পায়।

2. $\sum e_i^2 = \sum (y_i - \bar y)^2$, তাই $R^2 = 1 - 1 = \mathbf{0}$।

3. **CC বদলায় না**, কারণ একটা ধ্রুবক যোগ করলে ওঠানামার ধরন একই থাকে। **Bias +0.2 m বাড়ে।**

4. AA02-এর $e^2$ 0.3588 থেকে 0 হয়। নতুন sum = 1.0347 − 0.3588 = 0.6759। RMSE = $\sqrt{0.6759/8} = \sqrt{0.0845}$ = **0.291 m**। একটা record ঠিক হলেই RMSE 0.360 থেকে 0.291-এ নেমে আসে, যা ছোট $n$-এ RMSE-এর সংবেদনশীলতা দেখায়।

5. কারণ ভিন্ন station-এর $H_s$ আলাদা। Model শুধু station চিনে সেই station-এর গড় $H_s$ বললেও RMSE কম আসতে পারে। সত্যিকারের wave-ভিত্তিক শেখা প্রমাণ করতে লাগে **within-setup ক্রম** (Spearman) আর **within-setup permutation test**।

</details>
