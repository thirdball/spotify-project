# Spotify Music Popularity Prediction

## Introduction
This project analyzes Spotify music tracks to understand what audio features and metadata correlate with song popularity. 


## 4: Hypothesis Testing

Question:
Do explicit songs have significantly higher popularity 
than non-explicit songs on Spotify?

Null Hypothesis:
The distribution of popularity is the same for explicit 
and non-explicit songs. Any observed difference is due 
to random chance.

Alternative Hypothesis:
Explicit songs have a higher mean popularity than 
non-explicit songs.

Test Statistic: Absolute difference in mean popularity
WHY: Popularity is quantitative (0-100), so difference 
in means captures the direction and magnitude of 
any effect. Absolute value makes it two-sided.

Significance Level: α = 0.05

Results:
- Explicit songs mean:     36.45
- Non-explicit songs mean: 32.94
- Observed difference:     3.5163
- P-value:                 < 0.001

Conclusion:
Since p-value < 0.001 < 0.05, we reject the null 
hypothesis. The data suggests that explicit songs 
tend to be more popular than non-explicit songs on 
Spotify (36.45 vs 32.94 average popularity).

Note: We cannot conclude causation - genre may be 
a confounding variable since explicit songs are more 
common in rap/hip-hop, which tends to be more popular.


Step 8: Fairness Analysis

Groups:
- Group X: Explicit songs     (n=1,440)
- Group Y: Non-explicit songs (n=15,809)

WHY these groups?
We chose explicit vs non-explicit songs because our 
hypothesis test (Step 4) showed these groups have 
significantly different popularity distributions. 
This raises the question of whether our model 
predicts popularity equally well for both groups.

Evaluation Metric: RMSE
WHY: We built a regression model, so we must use 
RMSE rather than classification metrics like 
precision or recall.

Null Hypothesis:
Our model is fair. Its RMSE for explicit and 
non-explicit songs are roughly the same, and any 
differences are due to random chance.

Alternative Hypothesis:
Our model is unfair. Its RMSE differs significantly 
between explicit and non-explicit songs.

Test Statistic: 
Absolute difference in RMSE between explicit and 
non-explicit songs.

Significance Level: α = 0.05

Results:
- RMSE (Explicit songs):     18.5883
- RMSE (Non-explicit songs): 16.3730
- Observed difference:       2.2153
- P-value:                   < 0.001

Conclusion:
We reject the null hypothesis (p < 0.001 < 0.05). 
The model performs significantly worse for explicit 
songs (RMSE = 18.59) compared to non-explicit songs 
(RMSE = 16.37), a difference of 2.22 points.

This suggests our model is unfair with respect to 
explicit content - it is less accurate at predicting 
the popularity of explicit songs than non-explicit songs.
