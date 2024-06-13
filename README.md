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
8. Converted 'submitted' and 'date' columns 
