# Spotify Music Popularity Prediction

## Introduction
This project analyzes Spotify music tracks to understand what audio features and metadata correlate with song popularity. 


## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

The following cleaning steps were applied to the raw dataset:

1. **Dropped unnamed index column** (`Unnamed: 0`): This was a leftover 
artifact from the CSV export and contained no useful information.

2. **Dropped rows with missing artist/track/album names**: Only 1 row 
per column was affected. These rows could not be used for analysis 
without a track name or artist.

3. **Converted `release_date` to datetime**: The raw data stored dates 
as strings in mixed formats. Converting to datetime allowed us to 
extract `release_year` for temporal analysis.

4. **Converted `explicit` to boolean**: The column was stored as a 
string (`"True"`/`"False"`). Converting to a proper boolean made it 
usable for filtering and encoding.

5. **Removed duplicate tracks by `track_id`**: Some songs appeared 
multiple times across different genres. Keeping duplicates would bias 
the model toward those songs.

6. **Extracted first artist**: Some tracks had multiple artists separated 
by semicolons (e.g., `"Ingrid Michaelson;ZAYN"`). We extracted the first 
artist to enable merging with `artists.csv`.

7. **Merged with `artists.csv`**: Added `artist_popularity` and 
`artist_followers` by matching on artist name. 96.8% of tracks were 
successfully matched.

Here is the head of the cleaned DataFrame:

<iframe src="assets/df_head.html" width="100%" height="250px" frameborder="0"></iframe>

---

### Univariate Analysis

<iframe src="assets/popularity_distribution.html" width="100%" height="500px" frameborder="0"></iframe>

The distribution of song popularity is heavily right-skewed, with the 
majority of songs scoring between 0-20. This suggests that achieving 
widespread popularity on Spotify is rare, with only a small fraction 
of songs reaching high popularity scores.

<iframe src="assets/danceability_distribution.html" width="100%" height="500px" frameborder="0"></iframe>

Danceability scores follow a roughly normal distribution centered around 
0.5-0.6, suggesting most songs have moderate danceability. The relatively 
symmetric shape indicates that Spotify's catalog is fairly balanced between 
low and high danceability tracks.

---

### Bivariate Analysis

<iframe src="assets/popularity_by_explicit.html" width="100%" height="500px" frameborder="0"></iframe>

Explicit songs tend to have a slightly higher median popularity than 
non-explicit songs. This trend motivated our hypothesis test in Step 4, 
where we formally tested whether this difference is statistically significant.

<iframe src="assets/popularity_vs_artist.html" width="100%" height="500px" frameborder="0"></iframe>

There is a clear positive relationship between artist popularity and song 
popularity, suggesting that more popular artists tend to release more 
popular songs. This motivated the inclusion of `artist_popularity` as a 
key feature in our final model.

---

### Interesting Aggregates

The table below shows mean popularity, danceability, energy, and tempo 
for the top 10 genres. Pop and hip-hop genres dominate in popularity 
while also showing higher danceability scores, suggesting that 
commercially successful genres share common audio characteristics.

<iframe src="assets/genre_stats.html" width="100%" height="400px" frameborder="0"></iframe>

---

## Assessment of Missingness

### NMAR Analysis

I do not believe any column in this dataset is NMAR (Missing Not At 
Random). The tempo column has the most significant missingness at 19.4% 
(22,114 out of 114,000 rows).

I believe tempo is MAR (Missing At Random) rather than NMAR because its 
missingness appears to depend on danceability (p < 0.001) rather than on 
the tempo values themselves. Songs with lower danceability like ambient, 
classical, or spoken word tracks may have irregular rhythms that make it 
difficult for Spotify's audio analysis system to detect a clear BPM value.

If tempo were NMAR, the missingness would depend on the actual tempo 
value itself. To further investigate, it would be helpful to obtain 
Spotify's internal audio analysis confidence scores, which would help 
explain why certain tracks fail to have tempo recorded.

### Missingness Dependency

We performed two permutation tests to analyze whether the missingness 
of `tempo` depends on other columns:

**Test 1: Tempo Missingness vs Popularity**
- Observed difference in means: 0.0595
- P-value: 0.7140
- Conclusion: Fail to reject null - tempo missingness does **not** 
depend on popularity

**Test 2: Tempo Missingness vs Danceability**
- Observed difference in means: 0.0126
- P-value: < 0.001
- Conclusion: Reject null - tempo missingness **does** depend on danceability

<iframe src="assets/missingness_plot.html" width="100%" height="500px" frameborder="0"></iframe>

The plot shows that songs with tempo recorded (not missing) have a more 
concentrated danceability distribution (0.4-0.7), while songs with missing 
tempo have a flatter, more uniform distribution. This suggests that songs 
with more extreme danceability values are more likely to have missing tempo.

---

## Hypothesis Testing

**Question:** Do explicit songs have significantly higher popularity 
than non-explicit songs on Spotify?

**Null Hypothesis:** The distribution of popularity is the same for 
explicit and non-explicit songs. Any observed difference is due to 
random chance.

**Alternative Hypothesis:** Explicit songs have a higher mean popularity 
than non-explicit songs.

**Test Statistic:** Absolute difference in mean popularity between 
explicit and non-explicit songs. We chose this because popularity is 
quantitative (0-100 scale), making difference in means a natural 
and interpretable statistic.

**Significance Level:** α = 0.05

**Results:**
- Explicit songs mean popularity: 36.45
- Non-explicit songs mean popularity: 32.94
- Observed difference: 3.5163
- P-value: < 0.001

**Conclusion:** Since p-value < 0.001 < 0.05, we reject the null 
hypothesis. The data suggests that explicit songs tend to have higher 
popularity than non-explicit songs on Spotify. However, we cannot 
conclude causation - genre may be a confounding variable since explicit 
songs are more common in genres like rap and hip-hop, which tend to 
be more popular overall.

---

## Framing a Prediction Problem

**Prediction Problem:** Predict the popularity score (0-100) of a 
Spotify song based on its audio features and metadata.

**Type:** Regression (popularity is a continuous numerical value)

**Response Variable:** `popularity`
We chose popularity because it directly measures a song's success 
on Spotify and is valuable for artists and producers to understand 
what makes a song popular.

**Evaluation Metric:** RMSE (Root Mean Squared Error)
- Interpretable: same units as popularity (0-100)
- Penalizes large errors more than small ones
- Better than MAE for catching large mispredictions
- Better than MSE because it's in interpretable units

**Time of Prediction:** All features used (danceability, energy, 
valence, loudness, speechiness, acousticness, instrumentalness, 
liveness, explicit, track_genre, artist_popularity, artist_followers) 
are measured by Spotify immediately when a song is uploaded, making 
them all valid features to use at prediction time.

---

## Baseline Model

**Model:** Linear Regression

**Features (4 total):**
- Quantitative (2): danceability, energy (left as-is, already 0-1 scale)
- Nominal (2): explicit, track_genre (OneHotEncoded)

**Performance:**

| | RMSE | R² |
|--|------|-----|
| Training | 19.2612 | 0.2558 |
| Test | 19.1596 | 0.2561 |
| Generalization Gap | 0.1016 | 0.0003 |

**Is this a good model?**
No - the baseline model is not particularly good. An RMSE of 19.16 
means predictions are off by ~19 points on a 0-100 scale, and an 
R² of 0.2561 means the model only explains 25.6% of the variance 
in popularity. This is expected for a baseline model using only 4 
features and assuming linear relationships. Step 7 improves on this 
significantly.

---

## Final Model

**Model:** Ridge Regression (alpha=0.1)

**Features:**

| Feature | Type | Transformation |
|---------|------|----------------|
| danceability, energy, valence, speechiness, acousticness, instrumentalness, liveness | Quantitative | StandardScaler |
| artist_popularity | Quantitative | StandardScaler |
| loudness, artist_followers | Skewed | QuantileTransformer |
| explicit, track_genre | Nominal | OneHotEncoder |

**Why these new features?**
- `artist_popularity`: Artist fame is a strong predictor of track 
popularity. A song by a popular artist is more likely to be streamed 
regardless of audio features.
- `artist_followers`: Reflects the artist's fanbase size, which 
directly influences how many people will listen to their songs.
- `valence, speechiness, acousticness, instrumentalness, liveness`: 
Capture additional audio characteristics beyond just danceability and energy.

**Why these transformations?**
- **StandardScaler**: Ridge regression uses L2 regularization which is 
sensitive to feature scales. Standardizing ensures no single feature dominates.
- **QuantileTransformer**: Loudness and artist_followers are heavily skewed. 
QuantileTransformer maps them to a normal distribution, reducing outlier impact.
- **Ridge instead of Linear Regression**: Adds L2 regularization to prevent 
overfitting with many one-hot encoded genre features.

**Hyperparameter Tuning:**
Tested alpha values: [0.1, 1.0, 10.0, 100.0]
Best alpha: 0.1

**Performance:**

| | RMSE | R² |
|--|------|-----|
| Baseline (LR) | 19.1596 | 0.2561 |
| Final (Ridge) | 16.5677 | 0.3372 |
| Improvement | 2.5919 | 0.0811 |

The addition of `artist_popularity` and `artist_followers` was the key 
driver of improvement, confirming that artist fame is a stronger predictor 
of song popularity than audio features alone. The final model explains 
33.7% of the variance in popularity, a significant improvement over 
the baseline's 25.6%.

---

## Fairness Analysis

**Groups:**
- Group X: Explicit songs (n=1,440, 8.3% of test set)
- Group Y: Non-explicit songs (n=15,809, 91.7% of test set)

**Evaluation Metric:** RMSE (regression model)

**Null Hypothesis:** Our model is fair. Its RMSE for explicit and 
non-explicit songs are roughly the same, and any differences are 
due to random chance.

**Alternative Hypothesis:** Our model is unfair. Its RMSE differs 
significantly between explicit and non-explicit songs.

**Test Statistic:** Absolute difference in RMSE between groups

**Significance Level:** α = 0.05

**Results:**
- RMSE (Explicit songs): 18.5883
- RMSE (Non-explicit songs): 16.3730
- Observed difference: 2.2153
- P-value: < 0.001

**Conclusion:** We reject the null hypothesis (p < 0.001 < 0.05). 
The model performs significantly worse for explicit songs (RMSE = 18.59) 
compared to non-explicit songs (RMSE = 16.37). This is likely due to 
class imbalance - explicit songs make up only 8.3% of the test set, 
so the model learned patterns from non-explicit songs much better.

<iframe src="assets/fairness_plot.html" width="100%" height="500px" frameborder="0"></iframe>

The plot shows the null distribution of RMSE differences from 500 
permutations. The observed difference of 2.2153 (red line) falls far 
outside the null distribution, confirming that the model performs 
significantly differently for explicit vs non-explicit songs.