# Recipe ratings against nutritional values
Project on a given data for Recipe and Ratings for DSC80


## Introduction
The following report is based off of data from food.com. The main topic of interest is whether or not the different variables a food contains (calories, fats, sugar, etc.) may potentially play a part in the ratings given to each recipe by any users. This could help conclude whether how a food is appreciated by a person on food.com may also come in part as being a specific factor in a variable (such as high or low in calories, or low/high on fats). Of course there may obviously be other factors that could play importance, but currently we're interested in specifically nutrients may play an importance to ratings (if at all). We'll be working with 83,782 rows of data and our main two columns of focus will obviously be "nutrition" (contains a list of numerical values on a nutrients' level in a food based on:  [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)], where pdv stands for "percentage of daily value") and "average_ratings_per_recipe" (just a numerical of specifically the average rating of all ratings that were done of the given recipe). Other potentially important columns include "name" (recipe name), "submitted" (when recipe was submitted to food.com), and "description" (description of the recipe). The names and descriptions of recipes may help explain certain nutrition variable patterns which in turn may also affect ratings of specific food types. Submission dates may seem non-important however in case of duplicate food values, it may be important to compare nutrition values and rating relationships between these food values as time of submission may play a likely factor how they may differ.


## Cleaning and EDA

#### Data Cleaning

For each column I followed the following procedure:
1. run df[column_name].isna() to get the amount of na values if any in each column to replace
2. run (df[column name] <= 0).sum() or (df[column name].str.len == 0).sum() to see if any values had a 0 or less or a string with essentially no value

If the column had an na value we would fill it in this df with a relevant value. For example, if the column "descriptions" had nas, all the nas were filled with the string "No Descripton Given" (since all non na values in "description" column were strings)
Some columns had 0 for values but were fine to include (for example "average_ratings_recipe") so no changes were made. There also weren't any strings that had length 0 so no change made there either

I also checked for duplicates on certain columns (for example recipe_id) to confirm no duplicates would occur incorrectly and didn't find any hits. 
One specific column ("submitted") was converted to a datetime object else nothing much else about it was changed.

The most important variable change was that specific values in the dataframe that looked like lists were really strings (examples include "nutrition", "steps", etc.). These strings were converted to lists with the following methods:
1. df[column_name].str.replace("'", "").str.replace("'[", "").str.replace("]", "").str.split(", ")   (naive method, assumed there weren't any commas inside a value for example)
2. df[column_name].apply(eval)  (better method but ran much slower. Had to use on certain columns like "steps" which had commas in some values so couldn't split on that variable)
I used method 1 when possible else method 2 since method 1 seemed to computate faster, but there were still columns that had to be ran on "eval" (again "steps" for example.)
One unique column was nutrition which after I converted to lists I had to change the strings inside to floats. this was achieved use an apply function and more importantly, map().

Finally, since we were interested in variable levels independently, I took the nutritions column we cleaned and made a series based off of each specific index that the nutrition list would indicate based on. For example, I had a column named "number of calories" that made a series based on the first value of every list in "nutrition" (since the first value of the nutrition columns' list values represented calories). I repeated this process for all the variables that were represented in the nutrition list values, creating 7 columns for each overall.
```py
print(recipes_new.head().to_markdown(index=False))
```
