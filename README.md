# Introduction and Question Identification

Understanding how a recipe influences customer satisfaction is crucial for any restaurant. In our project, we aim to explore whether it's possible to **predict how good a recipe is** based solely on its contents and categorization.

More specifically, we investigate how the **Percent Daily Value (PDV)** of **Carbohydrates** and **Protein** impacts a recipe’s user **rating**. This can help restaurants optimize their offerings and also guide home cooks in creating more appealing dishes.

According to the [FDA](https://www.fda.gov/food/nutrition-facts-label/lows-and-highs-percent-daily-value-nutrition-facts-label#:~:text=20%25%20DV%20or%20more%20of,per%20serving%20is%20considered%20high.), a PDV of 20% or more is considered high. We'll use this threshold as a reference point in our analysis to evaluate whether nutrient density (perceived "healthiness") plays a role in how recipes are rated.

---

## Dataset Overview

Our merged dataset contains **17 columns** sourced from recipe and user interaction data:

| Column          | Description |
|-----------------|-------------|
| `name`          | Recipe name assigned by creator |
| `id`            | Unique recipe ID |
| `minutes`       | Recipe preparation time in minutes |
| `contributor_id`| Unique contributor user ID |
| `submitted`     | Date the recipe was submitted |
| `tags`          | Food.com tags assigned by the creator |
| `nutrition`     | Nutrition info: calories and PDV for sodium, protein, carbs, sugar, and saturated fats |
| `n_steps`       | Number of steps in the recipe |
| `steps`         | Ordered list of recipe steps |
| `description`   | User-provided recipe description |
| `ingredients`   | List of ingredients |
| `n_ingredients` | Number of ingredients |
| `user_id`       | Unique user ID from interactions data |
| `recipe_id`     | Unique recipe ID from interactions data |
| `date`          | Date of interaction with the recipe |
| `rating`        | Given user rating |
| `review`        | Given user review |

---

## Key Data Components

Among all the columns in our dataset, a few stand out as particularly important:

- **`nutrition`**: This column holds detailed health-related information for each recipe and is unique for every entry. It includes key nutritional metrics such as calories, sodium, protein, carbohydrates, sugar, and saturated fat — all expressed in Percent Daily Value (PDV).
- **`n_steps`**: The number of steps in a recipe can serve as a proxy for its complexity, which may influence a user's likelihood of leaving a rating, as well as the rating itself.
- **`tags`**: These provide useful metadata about the type of food, cooking method, cuisine, or occasion. Analyzing these can reveal trends between recipe types and user preferences.

---

## Data Cleaning and Exploratory Analysis

The first goal in our analysis was to clean and prepare the data for further exploration. We started by transforming the merged dataset so that all columns were in the most convenient and usable data types.

### Key Changes:

| Column             | Transformation |
|--------------------|----------------|
| `nutrition`        | Converted from string to list format (exploded elements) |
| `tags`             | Converted from string to list format |
| `Average Rating`   | Added as a new numeric column |
| `contributor_id`   | Converted to consistent ID format |
| `submitted`        | Converted to `datetime` object |

---

### Average Rating Calculation

**Important adjustments:**

1. **Merging Datasets**  
   We performed a left merge between the recipe and interactions dataframes using the `id` column, matching each unique recipe with its corresponding ratings and reviews.

2. **Handling Missing Ratings**  
   Recipes with a rating of `0` were treated as missing values (`np.nan`) since these were not proper ratings but rather passive interactions. Including them would have skewed the results, as valid ratings typically range from 1 to 5.

3. **Creating the `Average Rating` Column**  
   Ratings were aggregated by `recipe_id` to calculate the average rating for each recipe, accounting for multiple ratings by different users.

4. **Renaming Columns**  
   After merging, pandas appends `_x` and `_y` to duplicate column names. We renamed `rating_y` to `average_rating` and `rating_x` to `rating` for clarity.

5. **Evaluating Stringified Lists**  
   Both `nutrition` and `tags` originally contained strings that represented Python lists. We used `eval()` to convert these into proper list objects.

6. **Splitting the Nutrition Column**  
   The `nutrition` list was split into **seven new columns**, each representing a specific nutritional value (e.g., calories, protein, carbs, etc.). This made the nutritional data easier to analyze individually.

7. **Datetime Conversion**  
   The `submitted` and `date` columns were converted from object types to proper datetime format (`YYYY-MM-DD`) using `pd.to_datetime()`. This facilitates time-based analysis in future steps.

---

### Preview of Merged Dataset

You can interact with a sample of the cleaned dataset below:

```html
<iframe
  src="assets/interactive_dataframe_two.html"
  width="800"
  height="600"
  frameborder="0"
  style="background: #FFFFFF;"
></iframe>



***Univariate Analysis***
<iframe
  src="assets/figure_univariate.html"
  width="600"
  height="300"
  frameborder="0"
></iframe>

For our univariate analysis we are examining the distribution of the ratings column. We see here a heavily left skewed distribution as nearly 80% of the ratings are a 5. This will be extremely important for our model predictions later in this project. 

***Bivariate Analysis***
<iframe
  src="assets/figure_bivariate_correct.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

For the bivariate analysis, we are examining the relationship between protein percentage daily value and recipe ratings. High protein refers to protein pdv above 20%, whereas low protein refers to protein pdv below 20%. For both low and high protein recipes, the count per rating is descending, with the most of each being rated as 5 and the least being 1. Additionally, the difference in the proportions does not vary very much per rating.

***Interesting Aggregates***
This pivot table was created by aggregating by mean the high and low protein and carb recipes. This pivot table shows the mean ratings of recipes categorized by whether they have healthy protein content (protein_PDV > 20) and healthy carb content (carbs_PDV < 20). For example, the top line of the pivot table shows us that the average rating for recipes with an "unhealthy" amount of protein and carbs is 4.658152 whereas the bottom line shows us that the the average rating for recipes with an "healthy" amount of both protein and carbs is 4.676057. This can be helpful for our model later, since recipes with protein_PDV > 20 and carbs_PDV < 20 are more likely to be higher ratings.



**Aggregating Table**
<iframe
  src="assets/interactive_pivot.html"
  width="800"
  height="600"
  frameborder="0"
style="background: #FFFFFF;"
></iframe>

# Assessment of Missingness

***NMAR Analysis***
We believe that the rating column could be NMAR. If someone does not have strong feelings about the recipe they tried, it is reasonable to assume that they might not leave a rating. If someone strongly enjoyed the recipe, they are quite likely to leave a rating (a high rating), and similarly, if someone strongly disliked the recipe, they are quite likely to leave a rating (a low rating). Speaking from our own experience, we have only left ratings on things (games, movies, restaurants) when we either strongly enjoyed it or strongly disliked it. Notably, we also went through each column and found the proportion of missing values, and *'rating'* contained the most by a significant margin.

***Missingness Dependency***
*Ratings vs. number of steps*
We are now examining the missingness of the ratings column. Specifically here, we are investigating whether the missiness in the *'rating'* column depends on the *'n_steps'* column. 

**Null Hypothesis**: The missingness of ratings does not depend on the recipe's number of steps.
**Alternate Hypothesis**: The missingness of ratings does depend on the recipe's number of steps.
**Test Statistic**: The absolute difference of the mean number of steps for ratings that are missing and ratings that are not missing.
**Significance Level**: 0.05

We performed a standard permutation test, shuffling on the *'rating'* column, for each repetition, computing the absolute difference between the mean number of steps for ratings that are missing and ratings that aren't missing. The **observed test statistic** was approximately **1.34**. As can be seen in the graph, after running 1000 simulations, most test statistics in the permutation test hover between 0 and 0.14. The **p-value** is **0.00** and thus we can reject the null hypothesis. The missingness of 'rating' does depend on the 'n_steps' column (the number of steps in the recipe). 

*Ratings vs. Time to Complete Recipe*
We are once again examining the missingness of the ratings column. Specifically here, we are investigating whether the missiness in the *'rating'* column depends on the *'minutes'* column. 

**Null Hypothesis**: The missingness of ratings does not depend on the time to complete the recipe.
**Alternate Hypothesis**: The missingness of ratings does depend on the time to complete the recipe.
**Test Statistic**: The absolute difference of the mean percent daily value of protein of the distribution of the group without missing ratings and the distribution of the group with missing ratings.
**Significance Level**: 0.05

<iframe
  src="assets/rating_v_nsteps.html"
  width="800"
  height="600"
  frameborder="0"
style="background: #FFFFFF;"
></iframe>

We performed another standard permutation test, shuffling on the *'rating'* column, for each repetition, computing the absolute difference of the mean percent daily value of protein of the distribution of the group without missing ratings and the distribution of the group with missing ratings. The **observed test statistic** was approximately **51.45**. As can be seen in the graph, after running 1000 simulations, most test statistics in the permutation test hover between 0 and 0.14. The **p-value** is **0.13** and thus we fail to reject the null hypothesis. The missingness of 'rating' does not depend on the 'minutes' column (the amount of time to complete the recipe).

**Missingness permutation tests**
<iframe
  src="assets/rating_v_minutes.html"
  width="800"
  height="600"
  frameborder="0"
style="background: #FFFFFF;"
></iframe>

# Hypothesis Testing

To investigate the question, we ran a permutation test with the following hypotheses, test statistic, and significance level.
Of note, we are using the difference in means as the test statistic because it allows us to directly compare the average ratings of recipes with high protein against those with low protein. 

**Null Hypothesis**: The amount of protein in a recipe does not affect the rating 
**Alternate Hypothesis**: Higher protein recipes yield higher ratings 
**Test Statistic**: Difference in means between ratings of high protein and low protein 
**Significance Level**: 0.05

A permutation test is being conducted because we want to check if the two distributions (high protein recipe ratings and low protein recipe ratings) look like they come from the same population—and, we don't have the entire population's information. The **observed test statistic** was approximately **-0.0203**. The **p-value** is **0.04** and which is <0.05 and thus we we reject the null hypothesis. This suggests that higher protein recipes tend to be rated higher, possibly because consumers perceive high-protein recipes as healthier and more beneficial for their dietary needs.

**Hypothesis Test**
<iframe
  src="assets/hypothesis.html"
  width="800"
  height="600"
  frameborder="0"
style="background: #FFFFFF;"
></iframe>

# Framing a Prediction Problem

Our prediction problem involves predicting the ratings of recipes based on their nutritional content and other tags, making it a classification problem. Specifically, we are performing multiclass classification because the ratings are discrete values that fall into multiple categories (e.g., 1, 2, 3, 4, 5). The response variable we are predicting is the rating because the dataset is centered around the ratings of each of the recipes, and understanding what influences these ratings can provide valuable insights for recipe improvement and personalization. We chose the F1-score as our evaluation metric because it balances both precision and recall, providing a more comprehensive measure of a model's performance, especially in cases where the data may be imbalanced. Accuracy alone can be misleading if certain ratings are more frequent—which is something that we have observed earlier when looking at the distributions of ratings (5s are way more common than any other rating). The F1-score ensures that our model performs well across all rating categories.

Additionally, our predictive model is solely based on the columns in just the rating dataset, ensuring that we are not using future information or data unavailable at the prediction time, maintaining the integrity and validity of our model.

# Baseline Model

For our baseline model, we began by splitting the data into features and target, and then splitting the data into training and testing sets. We aimed to predict recipe ratings using a simple classifier. The features included in our model were derived from binarizing a discrete variable (number of steps) and two continous variables (protein and carb pdv). After binarizing, these features are all nominal because they represent categorical data without any inherent ordering.

Our metric score (F1) is 0.67. We believe that our current model is not optimal because it tends to just predict a 5 for every single recipe. This will yield the highest accuracy, so our model finds that it is safer just to predict a 5 for every recipe. This is in part due to the large skew in the ratings column as nearly 80% of all ratings are a 5. In addition, we found that when we look at the mean of the models predictions, we found that the mean is a 5. This means that it predicted a 5 every time no matter—which is understandable based on what we just said.

# Final Model

Our model improvement strategy began with writing a for loop to iterate through the tags columnn and to find the average rating for each unique tag. A pattern that we picked up on when scrolling through the 500+ unique tags was that the lowest rated ones tended to have a certain theme of being 'treats' and thus include the words 'cookie', 'brownie', and/or 'cake'. Because of this, we decided to binarize 'is_a_treat' in which contained a 1 if the recipe's tags contained one of 'cookie', 'brownie', or 'cake'. 

Another attempt made at improving our model was reducing the frequencies of recipes with ratings of fives. As we noted in the baseline model, the model tended to prefer to pick a rating of 5 because this maximized accuracy. So, in order to counteract this, we artificially deleted 25% of recipes that received a rating of 5. This unfortunately did not improve our model and lowered the respective F-1 score, by keeping just the is_treat binarized column the model f-1 score is more accurate. 

We also did use a RandomForestClassifier and conducted a GridSearchCV to tune the following hyperparameters: max_depth and n_estimators. Through a number of iterations, we found the best combinations of the hyperparameters to be  for the max_depth and  for the n_estimators.

Our final F1-score is 0.67. So, ultimately this final model demonstrates minimal improvement over the baseline model. A higher F1-score would have improved precision and recall in predicting recipe ratings. TWe would have liked if the incorporation of 'is_a_treat' and the dataset balancing helped to capture patterns in recipe ratings, but the RandomForestClassifier tuning hyperparameters did indeed slightly enhance the model's predictive ability.



# Fairness Analysis

For our fairness analysis we wanted to investigate if groups that were marked as a treat, which in our analysis refers to tags that included cookies, cake or brownies had a statistically significant difference in the models performance when compared against recipes that did not include treat related tags. 

Choice of Group X and Group Y:

Group X: Items tagged as "treat" (is_a_treat == 1).
Group Y: Items not tagged as "treat" (is_a_treat == 0).
Evaluation Metric:

Metric: Root Mean Squared Error (RMSE).
Justification: RMSE is suitable for regression tasks, such as predicting ratings, where we want to measure the average magnitude of the errors in predicting numerical values.
Null and Alternative Hypotheses:

Null Hypothesis: The model's RMSE for items tagged as dessert (Group X) is the same as for items not tagged as dessert (Group Y), indicating no systematic difference in prediction error.
Alternative Hypothesis: The model's RMSE for items tagged as dessert (Group X) is higher than for items not tagged as dessert (Group Y), suggesting that the model performs worse on desserts.

Choice of Test Statistic and Significance Level:

Test Statistic: Difference in RMSE between Group X (desserts) and Group Y (non-desserts).
Significance Level: Typically set to α = 0.05
Resulting p-value:

After performing the permutation test based on 10,000 permutations, the computed p-value is 0.86.
Conclusion:

Conclusion: Since the p-value (0.86) is more than the significance level (0.05), we reject the null hypothesis. The model's performance is not significantly different between items tagged as treat and items not tagged as treat (fail to reject the null hypothesis).

**Permutation test for fairness analysis**
<iframe
  src="assets/fairness_plot.html"
  width="800"
  height="600"
  frameborder="0"
style="background: #FFFFFF;"
></iframe>
