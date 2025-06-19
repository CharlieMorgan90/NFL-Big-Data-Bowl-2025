# NFL-Big-Data-Bowl-2025
I developed a data-driven Confusion Score to quantify pre-snap defensive disorganization in the NFL using tracking data, engineered features, and a logistic regression model.

**Problem Statement:**

**Pre-snap defensive confusion** occurs when defenders exhibit signs of miscommunication, hesitation, or last-second adjustments before the snap, often leading to coverage breakdowns or poor positioning. **My goal** was to identify and quantify these moments using tracking data. I analyzed defensive player movements for erratic behavior, such as frequent direction changes, crossing paths, late alignments, and sudden shifts in response to offensive motion.

Using these features, I trained a logistic regression model to predict whether a play was "confused" or "not confused." Plays were labeled based on statistical thresholds derived from my engineered features. I then used the resulting model coefficients to construct a **Confusion Score**—a numerical representation of how disorganized a defense appears pre-snap, based solely on defender movement. This was a complex challenge, as not all movement equates to confusion.

Once I had a reliable Confusion Score, I investigated which features most strongly correlated with defensive confusion, which correlated with defensive success, and which showed no strong relationship. I also analyzed the types of plays or offensive behaviors that were most likely to trigger confusion in the defense.

**Engineered Features:** _(Please see the document titled feature_analysis for detailed methodology)_

- maxDistance: Maximum distance between a defender and their assigned offensive player
- averageDistance: Average distance between defenders and receivers
- total_nearest_defender_distance: Sum of all nearest defender-to-receiver distances
- num_defenders_moving: Total number of defenders moving before the snap
- average_snap_speed: Average speed of defenders at the moment of the snap
- max_speed_in_window: Maximum speed reached by a defender during the pre-snap window
- segment_switch_count: Number of times defenders switched between who was following the offensive player in motion
- num_defenders_react: Number of defenders who reacted to the offensive motion
- avg_sim_score: Average similarity score between defender movement vector and offensive movement vector
- max_sim_score: Maximum similarity score between any defender and offender movement vector
- var_sim_score: Variance in defender movement similarity scores
- total_motion_duration: Total duration of defender motion during the pre-snap window
- teamTotalDistance: Total combined distance traveled by all defenders
- teamAverageSpeed: Average speed across all defenders during the pre-snap period

**Exploratory Feature Analysis:** _(This is a high-level summary. See the feature_analysis document for full details)_

Features Positively Correlated With Confusion

- totalMotionDuration, segmentSwitchCount, numDefendersReact, avgSimilarityScore, maxSimilarityScore, varSimilarityScore
- Ex. The top 100 plays with the highest tracking variance had an average EPA of 0.441 and average 8.96 yards gained per play. Massive offensive success.

Features Negatively Correlated With Confusion

- teamTotalDistance, teamTotalSpeed
- Suggests that defenders moving just before the snap are likely recognizing and reacting correctly to the offensive play.

Features With No Correlation To Confusion

- maxDistance, averageDistance, numDefendersMoving, maxSpeedThroughSnap

Labeling Dataset:

A subset of 1,040 plays was created and labeled specifically for training a logistic regression model using the engineered features we calculated to generate weights for a confusion score. The dataset was perfectly balanced, consisting of an equal number of plays labeled as Confused and Not Confused to avoid introducing any bias into the model.

Plays were labeled using logical rules grounded in earlier feature analysis, which examined the relationship between our features and offensive success metrics such as yards gained and expected points added (EPA). For example, plays with a high value of total_motion_duration — representing how long defenders were in motion before the snap — were labeled as Confused, since prior analysis showed that longer durations often correlated with greater offensive success. This same reasoning was applied across all selected features to ensure consistent and meaningful labeling.

This approach ensured that the model learned from the same relationships identified during our exploratory analysis, making the final confusion score both interpretable and grounded in real gameplay patterns.

For each of the selected plays, the dataset included one row consisting of the assigned confusion label (Confused or Not Confused) along with the full set of extracted features for that play.

**Model + Evaluation:**

I selected logistic regression for its interpretability and simplicity, which aligned with my goal: to extract weights for the confusion score, not necessarily to build the most predictive model. More complex models (e.g., XGBoost, Random Forest) might overfit or obscure the relationships between features and confusion, and wouldn't provide direct, usable coefficients.

### Although prediction performance was not the core goal, I still evaluated the model to understand its behavior

- **Accuracy**: 0.639
- **F1 Score**: 0.631
- **Feature Weights:**

Before training, I standardized the features using StandardScaler to ensure all inputs were on the same scale, critical for logistic regression to produce meaningful, comparable weights. The same scaler was reused when applying the model to the full dataset of 16,000 plays to ensure consistency.

**Evaluation Confusion Score:**

To test the effectiveness of the confusion score in identifying defensive confusion, I compared its values against offensive performance metrics like yards gained and expected points added (EPA).

Positive vs. Negative Confusion Scores:

Plays with confusion_score > 0:

- Avg Yards Gained: 6.37
- Avg EPA: 0.06

Plays with confusion_score < 0:

- Avg Yards Gained: 5.41
- Avg EPA: −0.04

This suggests that higher confusion scores are associated with better offensive outcomes.

Top vs. Bottom 500 Confusion Scores:

Top 500 most confused plays:

- Avg Yards Gained: 6.89
- Avg EPA: 0.15

Bottom 500 least confused plays:

- Avg Yards Gained: 4.94
- Avg EPA: −0.07

This further suggests that higher confusion scores are associated with better offensive outcomes.

Full Confusion Spectrum:

I created bar plots showing how offensive performance changes across the full range of confusion scores:

- **Plays sorted by confusion_score (lowest to highest)**
- Each bar represents average Yards Gained or average EPA
- Bars are color-coded:
  - Red: Negative Confusion Score
  - Green: Positive Confusion Score

These visualizations reveal an upward trend in offensive success as the confusion score increases supporting the validity of the model-generated score.

![](Visuals/Bar%20Chart.jpg)
_(The last 6 bars are greater than every other bar with the exception of one)_

![](Visuals/Bar%20Graph%202.png)
_(The last 6 bars have a positive epa meaning the play benefited the offense)_

While the logistic regression model achieved modest classification performance, its true value lies in producing a confusion score that correlates with defensive confusion or offensive success. This validates my feature engineering, labeling process, and modeling approach. Future iterations could explore ensemble methods or unsupervised clustering, but for interpretability and project goals, this model is a solid foundation. Moving forward, I could try using weights based on our analysis without the model's help and see if we can achieve better results. It would also be interesting to investigate outlier bins in the graphs above.
