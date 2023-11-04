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

#### Univariate Analysis

We'll look at graphs for our newly created nutrition columns and our average ratings' column.

###### Average ratings graph
Firstly, make note that the range of values for the average ratings' column is 0-5 which is a very small range. Hence I found it appropiate to use a box and whisker plot to demonstrate the distribution of the data. The result is below:

<iframe src="fig_average.html" width=800 height=600 frameBorder=0></iframe>

Looking at it carefully, we can clearly see some interesting behavior here. The biggest interest is that the maximum, upper quartile and median is all on the same value of 5. Our lower quartile is still quite high at 4 and an approximate minimum at around 2.43 and a large amount of listed outlier minimums on the left side of the box and whisker plot, ranging from 2.3 to 0. To me, this distribution seems to be telling us that a lot of our average ratings for this data seems to be quite high, in fact we might be having a very large amount of data that has is around the score 5 in fact (hence the median, upperquartile, and max all being 5). Even disregarding the 5 values, a lot of the data seems to still be shifted upwards, where 4 is the first quartile which is still quite a bit higher than expected.

###### Nutrition variables graph
The distribution of nutrients variables are all much more spread out then what we have on the average ratings. For example, while the average ratings ranged from 0 - 5.0 the column on the nutrition variable called protein ranges from a minimum of 0 to a max of 4356! (PDV). So a box and whisker isn't going to be ideal in plotting a distribution of the values here so instead we'll use something larger, a histogram. 

We still run into some value issues however, namedly we have a lot of bins with low counts that will reach out to the maximum! For example, consider this histogram on the calories column:

```py
<iframe src="fig_calories.html" width=800 height=600 frameBorder=0></iframe>
```

We have ALOT of bins and most of them aren't representative of the larger population (Which is near the start at 0). So we'll have to set reduce our value ranges for x using the "x_range":

```py
<iframe src="fig_calories_reduced.html" width=800 height=600 frameBorder=0></iframe>
```

Much better, we now see a more clearly distribution goes up early before starting to skew right as calories increase to their maximum. We also begin starting to see small difference changes as the counts reduce around the 1000 calories mark. 

We end up using this process for all of the nutrition variables since all of them have some very large ranges and counts for said values in each range. The determined x_range varies between each variable but the general premise I used was that I removed the values that had the top 200 numerical values of that variable. (so in the instant of calories, setting the x_range max to 5000 reduced around 200 values that had more than 5000 calories. In fact, calories' max went up to 10,000). Here a few examples of histograms we made on protein and carbohydrates:

```py
<iframe src="fig_carbohydrates_reduced.html" width=800 height=600 frameBorder=0></iframe>
```

```py
<iframe src="fig_protein_reduced.html" width=800 height=600 frameBorder=0></iframe>
```

Essentially all the histograms we made on nutrition variables came to a simular conclusion: The data tended to be right skewed (or even started at the maximum count as you can see in protein) before slowly reducing in counts height and reaching a very low bar count while heading towards the maximum. So we then know for the most part the nutrition variables here tend to be bounded near a value that's clsoer to 0 then some maximum (for example calories at around 1000) and that by right skewed logic, their means >= median >= mode. But this is just the distribution on each nutrition variable separately and not together.(for example a recipe may have near a maximum on calories but near a minimum on protein) How do these variables compare to our average ratings when factored and could we perhaps compound some of them together to create a relationship?

#### Bivariate Analysis
