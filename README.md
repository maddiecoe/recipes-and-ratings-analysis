# recipes-and-ratings-analysis

Created by Maddie Coe (madcoe@umich.edu)

## Introduction

For this project, the dataset used was from Food.com, a website that comprises of a collection of rescipes that are submitted, rated, and reviewed by its users. The particular dataset in this analysis was a mixture of two provided datasets: **recipes** and **interactions**. **Recipes** containes a list of recipes, with info such as minutes, nutrition facts, the steps, and ingredients. **Interactions** contains ratings and reviews of the recipes. 

My project centers around the question: how long can I expect a recipe to take?

I am a college student, and my time is precious. I have classes, meetings, and homework to do at any given night, so scheduling out my day is critical. However, the one thing I am not good at is estimating how long cooking will take. Thus, having a good idea of how long cooking a certain recipe will take will greatly improve how I schedule out my days. This can be helpful to any college student, or really anyone that cooks their own meals. It will be useful to help people prepare and will lead to better, more delicious meals in the process!

I merged the **recipes** and **interactions** datasets together into one dataframe. After this merge, there were 234,429 rows and 13 columns. The relevant columns in this analysis are as follows:

- **`minutes`** — The total time required to prepare and cook the recipe (our target variable). 
- **`n_steps`** — The number of instruction steps in the recipe. 
- **`n_ingredients`** — The number of ingredients included in the recipe. 
- **`tags`** — A list of user-generated tags that describe the recipe (for example: “one-pot”, “holiday”, “easy”).
- **`description`** — A short written summary of the recipe.
- **`submitted`** — The date the recipe was submitted, which was used to account for trends over time.


## Data Cleaning and Exploratory Data Analysis

To prepare the dataset for modeling and analysis, I merged and cleaned the **recipes** and **interactions** dataset. The cleaning process focused on aligning these sources and addressing inconsistencies or missing values introduced during user data collection.

**Merging the Datasets**

I performed a left merge to combine the **recipes** and **interactions** datasets. This merge was done according to the id (in **recipes**) and recipe_id (in **interactions**). That way, all of the recipes remained in the dataset, even if they were not reviewed or interacted with. Thus, the full recipe catalog could still be analyzed. 

To do this, I used the merge() function in pandas with how='left' to preserve all rows from the **recipes** table, regardless of whether an interaction existed on it.

**Handling Ratings of 0**

In the original **interactions** dataset, some recipes had a rating of 0. Since ratings are inteded to be on a 1-5 scale, a value of 0 meant that the review was missing, or it was a placeholder value. Thus, this 0 did not actually come from a user interacting with the recipe. Thus, I replaces these 0s with a np.nan. That way, this ensured that these values did not affect the average ratings, and it avoided problems in the model as they were incorrect values.

To do this, I used the replace() function to turn 0s into nans in the **`rating`** column.

**Calculated Average Ratings**

Rather than relying on individual user raings (which may be biased), I computed the average rating for each recipe and added it as a new column. This summary served as a more stable signal of recipe quality, which could help later in analyzing trends of cooking time.

To do this, I first grouped the dataframe by **`recipe_id`** and calculated the mean rating. Then, I mapped those values back into a new **`average_rating`** column in the main dataframe.

**Removing Outliers in Cooking Time**

I noticed that a small portion of recipes had extremely high cooking times, which could heavily skew the model and visualizations. These times were often due to outliers (like slow-cooking recipes), recipes with a high inactive time (like chilling, marinating, etc), or it could even be due to a data entry error. To reduce these outliers' influence in the model, I limited the dataset to the 90th percentile of cooking times, therefore I only kept recipes whose **`minutes`** value was in the bottom 90%. This resulted in more stable modeling, an easier interpretation of trends, and a focus on "everyday" recipes, which was the goal of this analysis.

To do this, I calculated the 90th percentile cutoff for the **`minutes`** column using quantile(0.90), then filtered the dataframe to include only recipes below that value.

**Note**

Due to my handling of the ratings of 0, I did not use any imputation strategies.

**First Five Rows**

Here are the first five rows of the data, post data cleaning, and including only some relevant columns:

|   recipe_id |   n_steps |   n_ingredients |   minutes |
|------------:|----------:|----------------:|----------:|
|      333281 |        10 |               9 |        40 |
|      453467 |        12 |              11 |        45 |
|      306168 |         6 |               9 |        40 |
|      306168 |         6 |               9 |        40 |
|      306168 |         6 |               9 |        40 |

##Number of Ingredients##

<iframe
src="assets/ingredients_distribution.html"
width="800"
height="600"
frameborder="0"
></iframe>
