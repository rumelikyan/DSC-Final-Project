# Introduction and Question Identification:
It is important to every restaurant to know how their recipes will influence customer satisfaction. More importantly, we find it to be rather important to us to be able to guage how good a recipe is solely from its contents and the categorization of the recipe. Knowing this information helps restaurants optimize their recipes and it also helps people to optimize their cooking experience. Therefore, our investigration aims to find how the amount of ***Carbohydrates (PDV)*** and ***Protein (PDV)*** affect a recipe's ratings. This will help us unravel to what extent do people consider the perceived healthiness of a meal when making a rating. [High Percentage Daily Value of a nutrient according to the FDA](https://www.fda.gov/food/nutrition-facts-label/lows-and-highs-percent-daily-value-nutrition-facts-label#:~:text=20%25%20DV%20or%20more%20of,per%20serving%20is%20considered%20high.) is 20%, and we thus plan to utilize this in our further analysis.

The original merged dataframe contains 12 columns

| Column          | Description |
|-----------------|-------------|
| `name`          | Recipe name assigned by creator |
| `id`            | Unique recipe ID |
| `minutes`       | Recipe preperation time in minutes |
| `contributor_id`| Unique contributor user ID |
| `submitted`     | Date recipe was submitted |
| `tags`          | Food.com tags from creator |
| `nutrition`     | Nutrition information on calories and percent daily value for sodium, protein, carbs, sugar and saturated fats|
| `n_steps`       | Number of steps in recipe |
| `steps`         | Text for recipe steps, in order |
| `description`   | User-provided description |
| `ingredients`   | Text for recipe ingredients |
| `n_ingredients` | Number of ingredients in recipe |
| `user id`         | unique user ID from interactions DF |
| `recipe id`   | unique recipe ID from interactions DF |
| `date`   | Date of interation with recipe |
| `rating` | Given Rating |
| `review` | Given Review |

The most important components in this dataframe is the nutrition column that will provide unique health information to each recipe and be distinct every time. Another important column is the number of steps, the complexity of a recipe can certainly contribute twoards the likelihood of leaving a rating and the rating itself. The tags column also has important information in finding trends between types of foods and their ratings.

# Data Cleaning and Exploratory Analysis

In this section the goal was firstly to yield a dataframe from our initially merged dataset with all columns in the most convenient data type, the columns that needed changing were:

| `nutrition`          | Exploded the elements and changed it from str to list |
| `tags`            | Changed the column such that the tags were in a list |
| `Average Rating`       | Added average rating column int |
| `contributor_id`| Unique contributor user ID |
| `submitted`     | Date time object |

***Average Rating***
Important adjustments:
1. Merged the recipe and interactions dataframes
   - A left merge on 'id' was performed in order to match each unique recipe with their corresponding rating and review.

***merged dataset***
<iframe
  src="assets/interactive_dataframe_two.html"
  width="800"
  height="600"
  frameborder="0"
style="background: #FFFFFF;"
></iframe>

2. Filling null values in ratings with np.nan
   - This was an important step because the recipe's that received a rating of 0 were not properly rated and were just observations, therefore they should not be taken in to account when taking an average. Additionally ratings are typically between 1-5 therefore to avoid bias in the data, we filled it with 0's.
3. Added the column ***Average Rating***:
   - We grouped by the unique recipe id column and found the average rating per recipe. This is because there are multiple ratings from different users per recipe.
4. Renamed columns
   - When merging and there happens to be a column in the left df with the same name as a column in the right df, it will add '_x' or '_y' to the end of the column name. We made sure to rename 'rating_y' to 'average rating', and 'rating_x' to 'rating' to make our dataframe names more coherent.
5. Applied the eval operation to the nutrition and tags columns:
   - The nutrition and tags columns contained strings of list. Applying eval removes these strings, and ensures that they are lists as is desired.
6. Split the nutrition column into 7 other columns (one for each piece of data in the nutrition list)
   - We applied a function that grabbed each unique value in the list, created a new column for each value in the list, and added that value to its corresponding new column.
7. Converted 'submitted' and 'date' columns to correct format
   - Using pd.to_datetime, we converted these two columns from objects to the correct datetime format of Year-Month-Day, which will allow us to easily perform analysis on dates if needed.


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
