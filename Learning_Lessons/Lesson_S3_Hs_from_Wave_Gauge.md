# Lesson S3: Wave Gauge থেকে Significant Wave Height ($H_s$) Label

> **Series:** Stereo Hs Exp V1 — Learning Notes
> **Script:** `scripts/03_compute_labels.py`
> **Format:** Theory → Mathematics → Toy hand calculation → Code mapping → আমাদের result → Quiz

---

## Table of Contents

1. [Wave gauge কী মাপে](#1-wave-gauge-কী-মাপে)
2. [$H_s$-এর সংজ্ঞা: দুটো ঐতিহাসিক পথ](#2-h_s-এর-সংজ্ঞা-দুটো-ঐতিহাসিক-পথ)
3. [কেন ঠিক "4"? Rayleigh distribution থেকে derivation](#3-কেন-ঠিক-4-rayleigh-distribution-থেকে-derivation)
4. [Spectral পথ: $H_{m0} = 4\sqrt{m_0}$ আর Parseval](#4-spectral-পথ-h_m0--4sqrtm_0-আর-parseval)
5. [আমাদের Pipeline: ধাপে ধাপে](#5-আমাদের-pipeline-ধাপে-ধাপে)
6. [Toy Example: হাতে হিসাব](#6-toy-example-হাতে-হিসাব)
7. [Code-এর কোন line কোন ধাপ](#7-code-এর-কোন-line-কোন-ধাপ)
8. [আমাদের আসল Result](#8-আমাদের-আসল-result)
9. [Quiz](#9-quiz)

---

## 1. Wave gauge কী মাপে

- **Resistive wave gauge** হলো পানিতে খাড়া করে বসানো একটা তার (বা তারের জোড়া)। পানির উচ্চতা বাড়লে তারের যতটা অংশ ডুবে থাকে তা বাড়ে, ফলে electrical resistance বদলায়। এই resistance-কে calibration দিয়ে উচ্চতায় (meter) রূপান্তর করা হয়।
- Output হলো একটা নির্দিষ্ট বিন্দুতে **sea surface elevation-এর time series** $\eta(t)$।
- আমাদের BS data-য়:
  - ৬টা gauge, pentagon আকারে সাজানো (radius 25 cm)।
  - 2011 সালে 10 Hz, 2013 সালে **20 Hz** sampling। Script এটা timestamp থেকে নিজেই বের করে।
  - Unit meter, সময় MATLAB datenum (UTC)।

$$\eta(t_n), \quad t_n = t_0 + \frac{n}{f_s}, \quad n = 0, 1, \dots, N-1$$

---

## 2. $H_s$-এর সংজ্ঞা: দুটো ঐতিহাসিক পথ

### 2.1 Wave-by-wave পথ: $H_{1/3}$
1. Time series-কে zero-crossing দিয়ে আলাদা আলাদা wave-এ ভাগ করা হয়। প্রতিটা wave-এর height হলো crest থেকে trough পর্যন্ত দূরত্ব।
2. সব wave height বড় থেকে ছোট ক্রমে সাজানো হয়।
3. **সবচেয়ে বড় এক-তৃতীয়াংশ wave-এর গড়** হলো $H_{1/3}$।

এই সংজ্ঞা নাবিকদের চোখে দেখা "wave height"-এর সাথে ভালো মেলে।

### 2.2 Statistical পথ: $H_{m0}$
Sea surface-কে একটা **random process** হিসেবে দেখা হয়। তখন আর আলাদা wave গুনতে হয় না, শুধু variance লাগে:

$$H_{m0} = 4\sqrt{m_0} = 4\,\sigma_\eta$$

যেখানে $m_0 = \sigma_\eta^2$ হলো $\eta$-এর variance।

**আমরা এই দ্বিতীয় পথ ব্যবহার করছি:** $H_s = 4\,\mathrm{std}(\eta)$। এটা robust, zero-crossing-এর ঝামেলা নেই, আর আধুনিক oceanography-তে standard।

---

## 3. কেন ঠিক "4"? Rayleigh distribution থেকে derivation

**Assumption:** $\eta$ একটা zero-mean Gaussian process আর narrow-band। তাহলে wave height $H$ **Rayleigh distribution** মেনে চলে:

$$p(H) = \frac{H}{4\sigma^2}\exp\!\left(-\frac{H^2}{8\sigma^2}\right), \quad H \ge 0$$

**Step 1: Exceedance probability** (কোনো wave $h$-এর চেয়ে বড় হওয়ার সম্ভাবনা):

$$P(H > h) = \exp\!\left(-\frac{h^2}{8\sigma^2}\right)$$

**Step 2: Top 1/3-এর সীমা $H^*$ বের করা।** সবচেয়ে বড় এক-তৃতীয়াংশ wave $H^*$-এর উপরে থাকে, অর্থাৎ $P(H > H^*) = 1/3$:

$$\exp\!\left(-\frac{H^{*2}}{8\sigma^2}\right) = \frac{1}{3}
\;\Rightarrow\; \frac{H^{*2}}{8\sigma^2} = \ln 3
\;\Rightarrow\; H^* = \sigma\sqrt{8\ln 3} = 2.965\,\sigma$$

**Step 3: সেই top 1/3-এর গড়।** Conditional mean:

$$H_{1/3} = E[H \mid H > H^*] = \frac{1}{1/3}\int_{H^*}^{\infty} H\,p(H)\,dH = 3\int_{H^*}^{\infty} H\,p(H)\,dH$$

এই integral numerically করলে আসে:

$$H_{1/3} = 4.004\,\sigma \approx 4\,\sigma$$

**এজন্যই $H_s = 4\sigma$।** Gaussian আর narrow-band assumption-এর অধীনে $4\sigma$ আর $H_{1/3}$ প্রায় সমান। বাস্তব সমুদ্রে broad-band spectrum-এর কারণে $H_{1/3}$ সাধারণত $H_{m0}$-এর চেয়ে কয়েক শতাংশ কম হয়।

> **Oral exam line:** "$H_{m0} = 4\sqrt{m_0}$ follows from the Rayleigh distribution of wave heights under a narrow-band Gaussian sea, for which the mean of the highest one-third of waves equals $4.004\,\sigma_\eta$."

---

## 4. Spectral পথ: $H_{m0} = 4\sqrt{m_0}$ আর Parseval

### 4.1 Wave spectrum
$\eta(t)$-কে বিভিন্ন frequency-র sinusoid-এর যোগফল হিসেবে দেখা যায়। **Spectrum** $E(f)$ বলে প্রতিটা frequency-তে কতটা energy (variance) আছে, unit $\mathrm{m^2/Hz}$।

### 4.2 Spectral moment
$$m_n = \int_0^\infty f^n\,E(f)\,df \qquad \Rightarrow \qquad m_0 = \int_0^\infty E(f)\,df$$

### 4.3 Parseval's theorem
**Spectrum-এর নিচের মোট area = time series-এর variance:**

$$m_0 = \int_0^\infty E(f)\,df = \sigma_\eta^2$$

তাই দুটো পথ গাণিতিকভাবে একই:

$$H_{m0} = 4\sqrt{m_0} = 4\sigma_\eta$$

**পার্থক্য শুধু একটা:** spectral পথে frequency band বেছে নেওয়া যায় (আমরা 0.05–2.5 Hz নিয়েছি)। Std পথে সব frequency থাকে, যেমন খুব ধীর drift বা খুব দ্রুত noise। তাই দুটোর মান মিললে বোঝা যায় data-য় এমন অবাঞ্ছিত energy নেই।

### 4.4 Welch method
বাস্তব data-তে একটা লম্বা FFT নিলে spectrum খুব noisy হয়। Welch method যা করে:
1. Time series-কে অনেকগুলো overlapping segment-এ ভাগ করে (আমাদের `nperseg = 1024`)।
2. প্রতিটা segment-এ window লাগিয়ে FFT করে।
3. সব segment-এর spectrum-এর গড় নেয়, ফলে smooth estimate পাওয়া যায়।

---

## 5. আমাদের Pipeline: ধাপে ধাপে

```
.mat file
  │
  ├─(1) Time_UTC (datenum) → Python datetime
  ├─(2) Image record-এর সময়সীমার ভেতরের sample রাখা
  ├─(3) প্রতিটা gauge: linear detrend (mean + trend সরানো)
  ├─(4) প্রতিটা gauge: Hs_g = 4·std(η_g)
  ├─(5) Label: Hs = median(Hs_1, …, Hs_6)
  └─(6) Check: spectral Hm0, full-record Hs, gauge spread, NaN, spikes
```

| ধাপ | কেন |
|---|---|
| (2) Time cut | Image যে ১২ বা ৩০ মিনিটের sea দেখছে, label-কেও ঠিক সেই সময়ের হতে হবে |
| (3) Detrend | Tide, sensor zero-drift বা calibration offset wave না। এগুলো variance-কে কৃত্রিমভাবে বাড়িয়ে দেয় |
| (4) প্রতিটা gauge আলাদা | একটা gauge খারাপ হলে সেটা আলাদা করে চোখে পড়ে |
| (5) Median | Outlier-robust। ৬টার মধ্যে ১–২টা খারাপ হলেও label প্রায় বদলায় না। Mean-এ একটা খারাপ gauge পুরো ফল টেনে নেয় |

---

## 6. Toy Example: হাতে হিসাব

### 6.A MATLAB datenum → সময়

BS01-এর প্রথম timestamp: `734777.67916782`

| ধাপ | হিসাব | ফল |
|---|---|---|
| পূর্ণ সংখ্যা অংশ | 734777 | তারিখ (MATLAB day count) |
| ভগ্নাংশ × 24 | 0.67916782 × 24 | 16.30003 hour → **16 h** |
| বাকি × 60 | 0.30003 × 60 | 18.0017 min → **18 min** |
| বাকি × 60 | 0.0017 × 60 | **0.10 s** |

অর্থাৎ **16:18:00.1 UTC**। Image record শুরু হয়েছে 16:18:00-এ, তাই gauge আর image প্রায় একই সাথে শুরু হয়েছে ✅।

Python-এর `datetime.fromordinal()`-এর day 1 হলো 0001-01-01, আর MATLAB-এর day 1 হলো 0000-01-01। পার্থক্য **366 দিন**, তাই code-এ 366 বিয়োগ করা হয়।

---

### 6.B একটা toy wave থেকে $H_s$

ধরা যাক 8টা sample, একটা পূর্ণ wave period, amplitude $a = 0.2$ m। Sensor-এর zero ঠিক না থাকায় সবগুলোর সাথে 0.05 m offset যোগ হয়েছে:

| $n$ | Pure wave $0.2\sin(2\pi n/8)$ | Measured $\eta_n$ (offset +0.05) |
|---|---|---|
| 0 | 0.0000 | 0.0500 |
| 1 | 0.1414 | 0.1914 |
| 2 | 0.2000 | 0.2500 |
| 3 | 0.1414 | 0.1914 |
| 4 | 0.0000 | 0.0500 |
| 5 | −0.1414 | −0.0914 |
| 6 | −0.2000 | −0.1500 |
| 7 | −0.1414 | −0.0914 |

**Step 1: Mean**

$$\bar\eta = \frac{0.05 + 0.1914 + 0.25 + 0.1914 + 0.05 - 0.0914 - 0.15 - 0.0914}{8} = \frac{0.40}{8} = 0.05 \text{ m}$$

Offset-টাই mean হিসেবে বেরিয়ে এসেছে।

**Step 2: Mean বাদ দেওয়া** → $\eta'_n = \eta_n - 0.05$ = pure wave column।

**Step 3: Variance**

$$\sigma^2 = \frac{1}{N}\sum \eta_n'^2 = \frac{0^2 + 0.1414^2 + 0.2^2 + 0.1414^2 + 0^2 + 0.1414^2 + 0.2^2 + 0.1414^2}{8}$$

$$= \frac{4(0.02) + 2(0.04)}{8} = \frac{0.08 + 0.08}{8} = \frac{0.16}{8} = 0.02 \text{ m}^2$$

**Step 4: Standard deviation**

$$\sigma = \sqrt{0.02} = 0.1414 \text{ m}$$

**Step 5: $H_s$**

$$H_s = 4\sigma = 4 \times 0.1414 = \mathbf{0.566 \text{ m}}$$

**Offset বাদ না দিলে কী হতো?** Mean সরানো না হলে variance-এর বদলে $\frac{1}{N}\sum\eta_n^2$ নেওয়া হতো, যা $0.02 + 0.05^2 = 0.0225$। তখন $H_s = 4\sqrt{0.0225} = 0.60$ m, অর্থাৎ **৬% বেশি**। শুধু sensor-এর একটা offset থেকেই এই ভুল আসত।

> ⚠️ **Teaching point:** এখানে একটাই sinusoid আছে, যার wave height $H = 2a = 0.4$ m। অথচ $H_s = 0.566$ m, অর্থাৎ $H_s = 1.41H$। কারণ "4σ = $H_{1/3}$" সম্পর্কটা শুধু **random (Rayleigh) sea**-র জন্য সত্য, একটা regular wave-এর জন্য না। Regular wave-এ $\sigma = a/\sqrt2$, তাই $4\sigma = 2\sqrt2\,a$।

---

### 6.C Linear detrend (5-point toy)

ধরা যাক tide-এর কারণে পানি ধীরে ধীরে উঠছে:

| $t$ | 0 | 1 | 2 | 3 | 4 |
|---|---|---|---|---|---|
| $y$ (m) | 0.10 | 0.32 | 0.18 | 0.40 | 0.30 |

একটা সরলরেখা $\hat y = a + bt$ fit করব (least squares)।

**Step 1: Mean**

$$\bar t = \frac{0+1+2+3+4}{5} = 2, \qquad \bar y = \frac{0.10+0.32+0.18+0.40+0.30}{5} = \frac{1.30}{5} = 0.26$$

**Step 2: Deviation**

| $t$ | $t - \bar t$ | $y - \bar y$ | $(t-\bar t)(y-\bar y)$ | $(t-\bar t)^2$ |
|---|---|---|---|---|
| 0 | −2 | −0.16 | 0.32 | 4 |
| 1 | −1 | 0.06 | −0.06 | 1 |
| 2 | 0 | −0.08 | 0 | 0 |
| 3 | 1 | 0.14 | 0.14 | 1 |
| 4 | 2 | 0.04 | 0.08 | 4 |
| **Sum** | | | **0.48** | **10** |

**Step 3: Slope আর intercept**

$$b = \frac{\sum(t-\bar t)(y-\bar y)}{\sum(t-\bar t)^2} = \frac{0.48}{10} = 0.048 \text{ m/step}$$

$$a = \bar y - b\,\bar t = 0.26 - 0.048 \times 2 = 0.164 \text{ m}$$

**Step 4: Residual (detrended signal)** $= y - (0.164 + 0.048t)$

| $t$ | Trend $0.164 + 0.048t$ | Residual |
|---|---|---|
| 0 | 0.164 | −0.064 |
| 1 | 0.212 | 0.108 |
| 2 | 0.260 | −0.080 |
| 3 | 0.308 | 0.092 |
| 4 | 0.356 | −0.056 |

Residual-এর mean 0, আর ঊর্ধ্বমুখী trend চলে গেছে। $H_s$ এই residual থেকেই হিসাব হয়।

> **কেন 30 মিনিটে এটা নিরাপদ:** ৩০ মিনিটে শত শত wave থাকে, আর সেগুলোর উপরে-নিচে ওঠানামা মিলে প্রায় শূন্য slope দেয়। ফলে detrend শুধু ধীর tide বা drift সরায়, wave-কে প্রায় ছোঁয় না। কিন্তু 6.B-এর মতো মাত্র এক period-এর data-য় linear detrend নিজেই wave-এর একটা অংশ সরিয়ে ফেলত। ছোট data-য় detrend সাবধানে ব্যবহার করতে হয়।

---

### 6.D Spectral পথে একই $H_s$ (Parseval check)

6.B-এর pure wave-এর ($N = 8$) DFT করলে:

$$X_k = \sum_{n=0}^{7}\eta'_n\,e^{-i2\pi kn/8}$$

Sinusoid-এর energy শুধু $k = 1$ আর তার mirror $k = 7$-এ থাকে:

$$|X_1| = |X_7| = \frac{N a}{2} = \frac{8 \times 0.2}{2} = 0.8, \qquad \text{বাকি সব } |X_k| = 0$$

One-sided variance (Parseval):

$$m_0 = \frac{2|X_1|^2}{N^2} = \frac{2 \times 0.64}{64} = 0.02 \text{ m}^2$$

এটা 6.B-এর variance-এর সাথে **হুবহু মিলে গেছে।** তাই:

$$H_{m0} = 4\sqrt{0.02} = 0.566 \text{ m} = H_s \text{ (std পথ)} ✅$$

---

### 6.E ছয়টা gauge থেকে একটা label (BS01-এর আসল সংখ্যা)

| Gauge | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|
| $H_{s,g}$ (m) | 0.299 | 0.295 | 0.305 | 0.310 | 0.297 | 0.287 |

**Step 1: ছোট থেকে বড় সাজানো:** 0.287, 0.295, **0.297, 0.299**, 0.305, 0.310

**Step 2: সংখ্যা জোড় (6টা), তাই মাঝের দুটোর গড়:**

$$\text{median} = \frac{0.297 + 0.299}{2} = \mathbf{0.298 \text{ m}}$$

Script-এর label-ও 0.298 m ✅।

**Spread** = max − min = 0.310 − 0.287 = 0.023 m (প্রায় 8%)।

**Median কেন mean-এর চেয়ে ভালো:** ধরো gauge 6 খারাপ হয়ে 0.287-এর বদলে 0.150 দেখাচ্ছে।
- Mean: 0.299 থেকে 0.276-এ নেমে যেত।
- Median: (0.297 + 0.299)/2 = **0.298, একটুও বদলাত না।**

---

## 7. Code-এর কোন line কোন ধাপ

| ধাপ | `03_compute_labels.py`-এর code |
|---|---|
| datenum → datetime | `datetime.fromordinal(whole_days) + timedelta(days=fraction) - timedelta(days=366)` |
| Sampling rate | `fs = 1 / (np.median(np.diff(time_datenum)) * 86400)` (1 দিন = 86400 s) |
| Time cut | `inside = (time_seconds >= offset_s) & (time_seconds <= offset_s + image_duration_s)` |
| Detrend | `scipy.signal.detrend(eta_g, type="linear")` |
| $H_s$ (std) | `hs_std = 4.0 * np.std(eta_g)` |
| Welch spectrum | `scipy.signal.welch(eta_g, fs=fs, nperseg=1024)` |
| $m_0$ | `np.trapezoid(psd[band], freq[band])` (trapezoid rule দিয়ে area) |
| $H_{m0}$ | `4.0 * np.sqrt(m0)` |
| Label | `np.median(hs_std_list)` |
| Spike | Median থেকে $6 \times 1.4826 \times \mathrm{MAD}$-এর বেশি দূরের sample। 1.4826 গুণ করলে MAD Gaussian std-এর সমান হয় |

---

## 8. আমাদের আসল Result

| Record | Gauge $H_s$ (m) | Table 2 (m) | পার্থক্য | Gauge spread (m) |
|---|---|---|---|---|
| BS01 | 0.298 | 0.30 | −1% | 0.023 |
| BS02 | 0.394 | 0.36 | +9% | 0.032 |
| BS03 | 0.477 | 0.45 | +6% | 0.035 |
| BS04 | 0.514 | 0.55 | −7% | 0.040 |
| BS05 | 0.593 | 0.66 | −10% | **0.120** |
| BS06 | **0.958** | **0.41** | **+134%** | **0.157** |
| BS07 | 0.555 | 0.65 | −15% | 0.076 |

**যা ঠিক আছে:**
- Time alignment: gauge আর image-এর শুরুর সময়ের পার্থক্য মাত্র ~0.1 s।
- Std আর spectral $H_s$ সব record-এ ~1%-এর মধ্যে মেলে। অর্থাৎ drift বা noise-এর সমস্যা নেই।
- কোনো NaN নেই, spike প্রায় নেই।

**যা তদন্ত করতে হবে:**
- **BS06:** gauge বলছে 0.96 m, অথচ Table 2 আর paper-এর text বলছে ~0.4 m। পার্থক্য দ্বিগুণেরও বেশি।
- **2013 (BS05–07):** gauge-গুলোর মধ্যে spread 2011-এর চেয়ে 3–4 গুণ বেশি, যদিও gauge-গুলো মাত্র 25 cm দূরে দূরে বসানো।

পরের ধাপ: `03b_label_diagnostics.py`, যেটা $T_p$, 5-min stationarity, gauge correlation আর gain ratio দেখে।

---

## 9. Quiz

1. একটা 20 মিনিটের gauge record-এ $\sigma_\eta = 0.125$ m। $H_s$ কত?
2. একটা regular (monochromatic) wave-এর height $H = 1.0$ m। এর $H_{m0}$ কত, আর সেটা $H$-এর চেয়ে বড় কেন?
3. $P(H > H^*) = 1/10$ ধরে $H^*$-কে $\sigma$-এর হিসাবে বের করো। (এটা $H_{1/10}$-এর সীমা।)
4. ছয়টা gauge-এর $H_s$ = [0.50, 0.52, 0.51, 0.49, 0.95, 0.50]। Mean আর median বের করো। কোনটা label হিসেবে ভালো, আর কেন?
5. 2013 সালের gauge 20 Hz-এ record করেছে। Script যদি ভুল করে 10 Hz ধরে নিত, তাহলে time cut-এ কী ভুল হতো?
6. Std পথ আর spectral পথে $H_s$ প্রায় সমান এলে data সম্পর্কে কী বোঝা যায়?

<details>
<summary>Answers</summary>

1. $H_s = 4 \times 0.125 = 0.50$ m।
2. $a = 0.5$ m, $\sigma = a/\sqrt2 = 0.354$ m, তাই $H_{m0} = 1.414$ m। $4\sigma = H_{1/3}$ সম্পর্কটা Rayleigh-distributed random sea-র জন্য derive করা। Regular wave-এ সব wave সমান, তাই এই সম্পর্ক খাটে না।
3. $\exp(-H^{*2}/8\sigma^2) = 0.1 \Rightarrow H^* = \sigma\sqrt{8\ln 10} = 4.29\,\sigma$।
4. Mean = 3.47/6 = 0.578 m। Median: সাজালে 0.49, 0.50, 0.50, 0.51, 0.52, 0.95, তাই (0.50 + 0.51)/2 = 0.505 m। Median ভালো, কারণ 0.95 একটা outlier (সম্ভবত খারাপ gauge), আর median সেটার প্রভাব প্রায় সম্পূর্ণ বাদ দেয়।
5. প্রতিটা sample-এর সময় দ্বিগুণ ধরা হতো। তখন 30 min-এর image span-এ আসলে 15 min-এর gauge data পড়ত, আর বাকি অংশ ভুল সময়ের data হতো। আমাদের script `fs` timestamp থেকে বের করে, তাই এই ভুল হয়নি।
6. মোট variance-এর প্রায় সবটাই 0.05–2.5 Hz wave band-এর ভেতরে। অর্থাৎ ধীর drift বা উচ্চ-frequency noise উল্লেখযোগ্য নয়, আর detrend ঠিকভাবে কাজ করেছে।

</details>
