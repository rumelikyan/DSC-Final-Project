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
This pivot table shows the mean ratings of recipes categorized by whether they have healthy protein content (protein_PDV > 20) and healthy carb content (carbs_PDV < 20). For example, the top line of the pivot table shows us that the average rating for recipes with an "unhealthy" amount of protein and carbs is 4.658152 whereas the bottom line shows us that the the average rating for recipes with an "healthy" amount of both protein and carbs is 4.676057.

<iframe
  src="assets/figure.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

***merged dataset***
<iframe
  src="assets/interactive_dataframe_two.html"
  width="800"
  height="600"
  frameborder="0"
style="background: #FFFFFF;"
></iframe>

**Aggregating Table**
<iframe
  src="assets/interactive_pivot.html"
  width="800"
  height="600"
  frameborder="0"
style="background: #FFFFFF;"
></iframe>

<iframe
  src="assets/rating_v_minutes.html"
  width="800"
  height="600"
  frameborder="0"
style="background: #FFFFFF;"
></iframe>

<iframe
  src="assets/rating_v_nsteps.html"
  width="800"
  height="600"
  frameborder="0"
style="background: #FFFFFF;"
></iframe>

