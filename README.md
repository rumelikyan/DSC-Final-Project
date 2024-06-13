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
  width="800"
  height="600"
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
