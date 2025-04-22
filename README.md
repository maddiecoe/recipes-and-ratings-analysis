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