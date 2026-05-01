# 📈 Search Ads ROI Experiment

## 📘 Overview
This project examines the effectiveness of running sponsored search ads on web browsers and assesses whether the reported ROI reflects true incremental value. The analysis is based on the Columbia Business School case about [RoI on sponsored search at Bazaar.com](https://caseworks.business.columbia.edu/caseworks/measuring-roi-sponsored-search-ads) and uses an experimental framework to compare treated and control groups and improve the reliability of the ROI estimate.

Treated unit: Google is the treated unit. It ran sponsored ads in Weeks 1–9 (pre-treatment period) and had its sponsored campaign go dark in Weeks 10–12 (post-treatment period).

Control unit: Yahoo, Bing, and Ask, which continue running sponsored ads throughout weeks 1–12.

---

## 🔑 Key Highlights
- The original ROI estimate of 320% was based on a limited view of ad performance.
- The refined approach uses a difference-in-differences (DiD) framework to better isolate the effect of the sponsored ads.


---

## 💰 ROI Calculation
We identified the following flaws in the previous ROI metric.

- The ROI estimate uses the probability that consumers purchase after landing on the page, but clicks do not always result in a page impression.
- It also relies on average margin per conversion, which may be skewed by extremely high-value items.
- Most critically, it calculates ROI using branded search, which may reflect organic behavior rather than incremental conversions driven by prompted ads.

The refined ROI is calculated using incremental lift, average margin per purchase, conversion rate, and weekly CPC:

```r
ROI = (Click Lift × Avg Margin × Conversion Rate - Weekly CPC) / Weekly CPC
```

---

## 📊 Results
The analysis shows that sponsored search ads appear to generate meaningful incremental traffic, and the refined ROI estimate of 579.8% suggests the campaign is highly valuable, though the result should be interpreted cautiously because the pre-treatment trends are not perfectly aligned.

## 🗂 Repository Structure
```bash
├── did_sponsored_ads.csv #data
├── Search_Ads_ROI_Experiment.Rmd #code with detailed analysis
└── README.md
```
