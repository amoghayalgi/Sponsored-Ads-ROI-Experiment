# 📈 Measuring ROI on Bazaar’s Sponsored Search Ads

**Project Description:** This project evaluates whether Bazaar’s reported 320% ROI from branded sponsored search ads is valid by using first-difference and difference-in-differences analysis to estimate incremental lift more accurately.

---

## 📘 Overview
Bazaar is a top online retailer in the US that uses display and search engine ads to drive click conversion. Search engine online advertising, specifically through Google and Bing, includes branded search (consumers’ search including the term “Bazaar”) and non-branded search. Bob, a member of the Marketing Analytics team, reported a 320% ROI through Google branded search ads. We examine whether this ROI measure calculation is valid and provide further recommendations.

---

## 🔑 Key Insights & Recommendations
- Bob’s ROI metric is defective: ROI is supposed to measure the incremental conversions resulting from prompted ads rather than organic consumer behaviors.
- We corrected this by adding incremental lift factor from the difference-in-differences estimate in the refined ROI formula.
- There is no data provided for non-branded search.
- We recommend the team look into non-branded search and evaluate whether it yields higher ROI.
- If positive, the team should switch to investing in non-branded search ads.

---

## ⚠️ Cautions
Although Google and the control platforms follow broadly similar patterns before week 10, the control group shows a stronger upward trend even in the pre-treatment period. This imperfect alignment means the parallel trends assumption may not fully hold, so the DiD estimate should be interpreted with caution. Some of the post-treatment difference may reflect pre-existing trend divergence rather than the treatment effect alone.

---

## 📊 Data Summary
```r
library(dplyr)
library(ggplot2)

bazaar <- read.csv("did_sponsored_ads.csv")
bazaar <- bazaar %>%
  mutate(avg_total_clicks = avg_spons + avg_org,
         treated = ifelse(platform == "goog", 1, 0),
         post = ifelse(week > 9, 1, 0))

bazaar %>%
  group_by(treated, post) %>%
  summarise(avg = mean(avg_total_clicks))
```

The average total clicks by group and period were:
- Control, pre: 5265
- Control, post: 13330
- Google, pre: 8390
- Google, post: 6544

A log transformation of `avg_total_clicks` was attempted, but it over-corrected the skew and produced a left-skewed distribution, so the original values were retained.

---

## 📉 First Difference
The unit of observation is a week capturing average total clicks on Google. The pre-period spans 9 weeks before the technical glitch, and the post-period considers clicks between weeks 10 and 12. A simple regression was run to evaluate how the estimated treatment effect changes when using `post` as the sole predictor.

```r
# first-difference: the % change in web traffic arriving from Google; (after – before) / before
before_mean <- mean(bazaar$avg_total_clicks[bazaar$platform == "goog" & bazaar$post == 0])
google_data <- bazaar %>% filter(platform == "goog")
first_diff <- lm(avg_total_clicks ~ post, data = google_data)
summary(first_diff)

post_effect <- coef(first_diff)['post'] / before_mean
post_effect
```

Results show that click rates in the post period decreased by 22% compared to the pre period, with `p-value = 0.576`. However, this is only a before-after comparison, not a causal estimate, because it attributes all change in the treated group to the treatment.

---

## 🔁 Difference in Differences
The unit of observation is a search engine-week cell: for each platform (Yahoo, Bing, Google, Ask) and each week (1–12), we observe average sponsored and organic traffic. Google is the treatment group. It ran sponsored ads in Weeks 1–9 and had its sponsored campaign go dark in Weeks 10–12. The control group is the three other platforms that do not have any interruption during the 12-week experiment.

```r
# difference-in-differences
did <- lm(avg_total_clicks ~ treated + post + treated:post, data = bazaar)
summary(did)

coef(did)['treated:post'] / before_mean
```

The interaction term `treated:post` is statistically significant, confirming a treatment effect. Removing Google’s sponsored ads led to a significant drop in traffic, about a 118% decrease compared to the pre-period baseline after accounting for the trend observed from the control platforms.

---

## 📈 Parallel Trends
```r
week_avg <- bazaar %>%
  group_by(week, treated) %>%
  summarise(avg = mean(avg_total_clicks))

ggplot(week_avg, aes(x = week, y = avg, color = factor(treated))) +
  geom_line() +
  geom_vline(xintercept = 9, linetype = "dotted") +
  theme_bw()
```

The parallel trends plot shows Google and the control platforms trending similarly in weeks 1–9, but diverging sharply after week 9, which is consistent with the treatment effect. However, the control group’s steep upward trend compared to Google’s flat or declining trend raises concerns about whether the parallel trends assumption fully holds.

---

## 💰 Fixed ROI Calculation
The current search ad engine ROI formula is calculated as:

```r
CPC = 0.6
avg_num_of_spons_ad <- mean(bazaar[bazaar$platform == 'goog' & bazaar$week <= 9, ]$avg_spons, na.rm = TRUE)
weekly_cpc <- CPC * avg_num_of_spons_ad
avg_margin_per_purchase <- 21
lift <- 9910.6
treated_rev <- lift * avg_margin_per_purchase
roi <- (treated_rev * .12 - weekly_cpc) / weekly_cpc
roi
```

We identified three flaws in Bob’s ROI metric:
- Bob uses the probability that consumers purchase once they land on the page, but clicks don’t always result in a page impression.
- He also used average margin per conversion, which may be skewed by extremely high-value items.
- Most critically, he calculated ROI using branded search, which may reflect organic behavior rather than incremental conversions from prompted ads.

Due to data limitations, we addressed only the third issue in the revised ROI formula. Using the information provided, the corrected ROI is `579.8%`, which exceeds Bob’s original estimate of `320%`. However, since only one flaw was addressed, the remaining two issues should be revisited before adopting this as the final ROI formula.

---

## 🗂 Repository Structure
```bash
├── data/
│   └── did_sponsored_ads.csv
├── notebooks/
│   └── HW3_Amogha_Rachel.Rmd
├── outputs/
│   └── figures/
├── README.md
```
