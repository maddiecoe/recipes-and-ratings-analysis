# How Long Will This Take?

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


### Merging the Datasets

I performed a left merge to combine the **recipes** and **interactions** datasets. This merge was done according to the id (in **recipes**) and recipe_id (in **interactions**). That way, all of the recipes remained in the dataset, even if they were not reviewed or interacted with. Thus, the full recipe catalog could still be analyzed. 

To do this, I used the merge() function in pandas with how='left' to preserve all rows from the **recipes** table, regardless of whether an interaction existed on it.


### Handling Ratings of 0

In the original **interactions** dataset, some recipes had a rating of 0. Since ratings are inteded to be on a 1-5 scale, a value of 0 meant that the review was missing, or it was a placeholder value. Thus, this 0 did not actually come from a user interacting with the recipe. Thus, I replaces these 0s with a np.nan. That way, this ensured that these values did not affect the average ratings, and it avoided problems in the model as they were incorrect values.

To do this, I used the replace() function to turn 0s into nans in the **`rating`** column.


## Calculated Average Ratings

Rather than relying on individual user raings (which may be biased), I computed the average rating for each recipe and added it as a new column. This summary served as a more stable signal of recipe quality, which could help later in analyzing trends of cooking time.

To do this, I first grouped the dataframe by **`recipe_id`** and calculated the mean rating. Then, I mapped those values back into a new **`average_rating`** column in the main dataframe.


### Removing Outliers in Cooking Time

I noticed that a small portion of recipes had extremely high cooking times, which could heavily skew the model and visualizations. These times were often due to outliers (like slow-cooking recipes), recipes with a high inactive time (like chilling, marinating, etc), or it could even be due to a data entry error. To reduce these outliers' influence in the model, I limited the dataset to the 90th percentile of cooking times, therefore I only kept recipes whose **`minutes`** value was in the bottom 90%. This resulted in more stable modeling, an easier interpretation of trends, and a focus on "everyday" recipes, which was the goal of this analysis.

To do this, I calculated the 90th percentile cutoff for the **`minutes`** column using quantile(0.90), then filtered the dataframe to include only recipes below that value.


### Note

Due to my handling of the ratings of 0, I did not use any imputation strategies.


### First Five Rows

Here are the first five rows of the data, post data cleaning, and including only some relevant columns:

|   recipe_id |   n_steps |   n_ingredients |   minutes |
|------------:|----------:|----------------:|----------:|
|      333281 |        10 |               9 |        40 |
|      453467 |        12 |              11 |        45 |
|      306168 |         6 |               9 |        40 |
|      306168 |         6 |               9 |        40 |
|      306168 |         6 |               9 |        40 |


### Number of Ingredients

Below is a histogram of the number of ingredients each recipe calls for. The histogram visualizes this distribution in a nice, interpretable way. As you can see, most recipes call for 8-9 ingredients. This is important to remember as number of ingredients will be used in the model.

<iframe
src="assets/ingredients_distribution.html"
width="800"
height="600"
frameborder="0"
></iframe>

### Recipe Duration By Number Of Ingredients

Below are box plots of how the number of ingredients compares to the duration of cooking time. As you can see, with the more ingredients a recipe calls for, the longer on average a recipe takes. However, this isn't precise and needs to be explored more in the model.

<iframe
src="assets/duration_by_num_ingredients.html"
width="800"
height="600"
frameborder="0"
></iframe>


### Cooking Time By Ingredient Bin

| ingredient_bins   |     0-5 |    6-10 |   11-20 |     21+ |
|:------------------|--------:|--------:|--------:|--------:|
| 0-5               | 14.1936 | 28.5485 | 39.0072 | 40.9332 |
| 6-10              | 23.1066 | 35.6865 | 44.5742 | 54.7995 |
| 11-15             | 31.0787 | 42.1744 | 49.9156 | 59.9454 |
| 16-20             | 41.3515 | 49.6151 | 54.2731 | 65.4599 |
| 21+               | 49.3214 | 50.6863 | 64.7979 | 69.7297 |

This table shows the average cooking time for recipes based on the number of ingredients and cooking steps. It reveals a clear trend that recipes with more ingredients and steps tend to take longer to cook. By analyzing these patterns, we can gain insights into how recipe complexity influences cooking time.



## Framing a Prediction Problem

For this project, the prediction problem is a regression task, where the goal is to predict the cooking time, or the **minutes** of a recipe. I chose cooking time as the response variable because it directly correlates with the complexity and preparation time involved in making a recipe. By understanding how various factors such as the number of steps and ingredients affect cooking time, I can help users select recipes that fit their available time.

The evaluation metrics for the final model are Root Mean Squared Error (RMSE) and Mean Absolute Error (MAE). RMSE penalizes larger errors more heavily, providing an intuitive understanding of the magnitude of prediction errors, while MAE offers a simpler interpretation by calculating the average error. By using both metrics, I can evaluate the performance of the model in different ways, ensuring that the predictions are both accurate and consistent.

At the time of prediction, I would know the number of steps, ingredients, the length of the recipe description, and the year the recipe was submitted. These features are available before the user begins preparing the recipe, ensuring that all information used for prediction is accessible at the point of recipe selection.



## Baseline Model

The model used for the baseline model was a Linear Regression model, which predicted the cooking time (in minutes) based on the number of steps and ingredients in a recipe. These features were selected for their potential to influence cooking time, as seen in the exploration above, and as recipes with more ingredients or steps logistically require more time.

The features used in the model were:
- **`n_steps`** — The number of instruction steps in the recipe. This is a continuous variable. 
- **`n_ingredients`** — The number of ingredients included in the recipe. This is also a continous variable.

Both features are **quantitative**, as they represent counts or measurement that can take any numeric value within reason. No encoding was necessary for these features, because they are already numeric.


### Performance of the Model

The performance of the baseline model was evaluated using **Root Mean Sequared Error (RMSE)**, which measured the average magnitude of prediction errors. In this case, the baseline model had an RMSE of **22.36 minutes**, meaning that the model's predictions were off by about 22 minutes on average.

Given that the cooking times for recipes might vary, this RMSE suggests that the model has significant room for improvement, especially when predicting times for more complex recipes. While functional, the model may not be considered highly accurate for practical use, especially if prediction precision is important.


### Is the Model "Good"?

Based on the current performance of an RMSE = 22.36 minutes, the baseline model is not ideal in terms of predictive accuracy. While it serves as a solif starting point, its performance could be improved. Improvements could come about with adding more informative features and/or using more advanced algorithms.



## Final Model

To improve on the baseline linear regression model, I engineered two additional features:
- **`len_description`** — This feature captured the length of the recipe description. Recipes with longer descriptions tend to be more detailed, possibly indicating a greater complexity or more steps. This context helped the model differentiate between quick summaries and longer, potentially more time-consuming recipes.
- **`year_submitted`** — This feature represented the year a recipe was submitted. I cinluded this because newer recipes may favor simplicity as our lifestyles change. 

All four features used in the final model, `n_steps`, `n_ingredients`, `len_description`, and `year_submitted`, are quanititative and did not need any encoding.


### Modeling and Hyperparameters

For the final model, I selected a **Random Forest Regressor**, which is a tree-based model that is well-suited for caputring non-linear relationships in the data. It did not assume a linear relationship between the selected features and the response variable, making it more flexible. 

I performed a **Grid Search** to tune the following parameters:
- **`n_estimators`** — Number of trees in the forest (tested values: 100, 200).
- **`max_depth`** — Maximum depth of each tree (tested values: 5, 10, 20).
- **`min_samples_split`** — Minimum number of samples required to split an internal node (tested values: 2, 5).

The best performing combination was 200, 20, and 2, in that order.


### Performance

The final model's RMSE was 0.41 minutes, which is a drastic improvement from the 22.36 RMSE from the baseline model. The Random Forest Regressor, with additional informative features and tuned hyperparameters, significantly reduced prediction error. An RMSE of 0.41 minutes implies that the model can now predict cooking time within less than a minute on average, which is highly accurate for this task.