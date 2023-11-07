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

Here's the head of our cleaned dataframe:

| name                               |     id |   minutes |   contributor_id | submitted           |   n_steps |   n_ingredients |   average_rating_per_recipe |   number of calories |   total_fat (PDV) |   sugar (PDV) |   sodium (PDV) |   protein (PDV) |   saturated_fat (PDV) |   carbohydrates (PDV) |
|:-----------------------------------|-------:|----------:|-----------------:|:--------------------|----------:|----------------:|----------------------------:|---------------------:|------------------:|--------------:|---------------:|----------------:|----------------------:|----------------------:|
| 1 brownies in the world  best ever | 333281 |        40 |           985201 | 2008-10-27 00:00:00 |        10 |               9 |                           4 |                138.4 |                10 |            50 |              3 |               3 |                    19 |                     6 |
| 1 in canada chocolate chip cookies | 453467 |        45 |          1848091 | 2011-04-11 00:00:00 |        12 |              11 |                           5 |                595.1 |                46 |           211 |             22 |              13 |                    51 |                    26 |
| 412 broccoli casserole             | 306168 |        40 |            50969 | 2008-05-30 00:00:00 |         6 |               9 |                           5 |                194.8 |                20 |             6 |             32 |              22 |                    36 |                     3 |
| millionaire pound cake             | 286009 |       120 |           461724 | 2008-02-12 00:00:00 |         7 |               7 |                           5 |                878.3 |                63 |           326 |             13 |              20 |                   123 |                    39 |
| 2000 meatloaf                      | 475785 |        90 |          2202916 | 2012-03-06 00:00:00 |        17 |              13 |                           5 |                267   |                30 |            12 |             12 |              29 |   

(Note that we dropped columns in our above data head display (specifically columns "tags", "nutrition", "steps", "description", and "ingredients") since they contained information overflow for reference sake. Note these columns while dropped are still included in real dataframe.)

#### Univariate Analysis

We'll look at graphs for our newly created nutrition columns and our average ratings' column.

###### Average ratings graph
Firstly, make note that the range of values for the average ratings' column is 0-5 which is a very small range. Hence I found it appropiate to use a box and whisker plot to demonstrate the distribution of the data. The result is below:

<iframe src="fig_average.html" width=800 height=600 frameBorder=0></iframe>

Looking at it carefully, we can clearly see some interesting behavior here. The biggest interest is that the maximum, upper quartile and median is all on the same value of 5. Our lower quartile is still quite high at 4 and an approximate minimum at around 2.43 and a large amount of listed outlier minimums on the left side of the box and whisker plot, ranging from 2.3 to 0. To me, this distribution seems to be telling us that a lot of our average ratings for this data seems to be quite high, in fact we might be having a very large amount of data that has is around the score 5 in fact (hence the median, upperquartile, and max all being 5). Even disregarding the 5 values, a lot of the data seems to still be shifted upwards, where 4 is the first quartile which is still quite a bit higher than expected.

###### Nutrition variables graph
The distribution of nutrients variables are all much more spread out then what we have on the average ratings. For example, while the average ratings ranged from 0 - 5.0 the column on the nutrition variable called protein ranges from a minimum of 0 to a max of 4356! (PDV). So a box and whisker isn't going to be ideal in plotting a distribution of the values here so instead we'll use something larger, a histogram. 

We still run into some value issues however, namedly we have a lot of bins with low counts that will reach out to the maximum! For example, consider this histogram on the calories column:

<iframe src="fig_calories.html" width=800 height=600 frameBorder=0></iframe>

We have ALOT of bins and most of them aren't representative of the larger population (Which is near the start at 0). So we'll have to set reduce our value ranges for x using the "x_range":

<iframe src="fig_calories_reduced.html" width=800 height=600 frameBorder=0></iframe>

Much better, we now see a more clearly distribution goes up early before starting to skew right as calories increase to their maximum. We also begin starting to see small difference changes as the counts reduce around the 1000 calories mark. 

We end up using this process for all of the nutrition variables since all of them have some very large ranges and counts for said values in each range. The determined x_range varies between each variable but the general premise I used was that I removed the values that had the top 200 numerical values of that variable. (so in the instant of calories, setting the x_range max to 5000 reduced around 200 values that had more than 5000 calories. In fact, calories' max went up to 10,000). Here a few examples of histograms we made on protein and carbohydrates:


<iframe src="fig_carbohydrates_reduced.html" width=800 height=600 frameBorder=0></iframe>

<iframe src="fig_protein_reduced.html" width=800 height=600 frameBorder=0></iframe>


Essentially all the histograms we made on nutrition variables came to a simular conclusion: The data tended to be right skewed (or even started at the maximum count as you can see in protein) before slowly reducing in counts height and reaching a very low bar count while heading towards the maximum. So we then know for the most part the nutrition variables here tend to be bounded near a value that's clsoer to 0 then some maximum (for example calories at around 1000) and that by right skewed logic, their means >= median >= mode. But this is just the distribution on each nutrition variable separately and not together.(for example a recipe may have near a maximum on calories but near a minimum on protein) How do these variables compare to our average ratings when factored and could we perhaps compound some of them together to create a relationship?

#### Bivariate Analysis

I ended up focusing on a potential relationship between each nutrition variable and average ratings. Here one plot result for the variable calories:

<iframe src="calories_scatter.html" width=800 height=600 frameBorder=0></iframe>

Looking at this plot, it's rather hard to really identify if there is a correlation at all. Based on the plotted trendline, it's really quite the negative slope with a very large constant. Hence it's likely that the correlation between calories and average ratings is in fact not related at all, or no correlation, since the trendline aims to go downward but our data points are shifting upwards as average ratings would increase. What if we did the filtered x range as we did previous on this plot?

<iframe src="alt_calories_scatter.html" width=800 height=600 frameBorder=0></iframe>

The slope doesn't change that much for the trendline and we still have a lot of high values in just about every type of rating value, so it's quite likely we're seeing a pattern of no correlation. Also to note with both plots is again that we have increasing points for each rating point increase with based on the plot, we have a very high number of datapoints in the ratings of 3 - 5. This matches the pattern we saw in our previous box and whisker plot on just the average ratings. Otherwise, we should expect a lack of correlation between calories and ratings based on these results since the calorie levels are so scattered between all the ratings. (Note this was a similar plot pattern for the rest of the nutrition variables).

#### Interesting Aggregates

All of the columns we're working with are numerical so for proper pivot tabling, I used pd.qcut on columns of interest to create bins to pivot on. One such example where I made bins on average ratings and calories and pivot table on count is shown below:

| rating bins (down) against calory bins (right)   |   (-0.001, 58.0] |   (58.0, 91.1] |   (91.1, 119.715] |   (119.715, 146.5] |   (146.5, 171.325] |   (171.325, 196.6] |   (196.6, 222.4] |   (222.4, 248.9] |   (248.9, 276.7] |   (276.7, 305.4] |   (305.4, 337.2] |   (337.2, 370.6] |   (370.6, 407.1] |   (407.1, 448.1] |   (448.1, 498.7] |   (498.7, 563.3] |   (563.3, 650.3] |   (650.3, 783.0] |   (783.0, 1078.775] |   (1078.775, 45609.0] |
|:--------------|-----------------:|---------------:|------------------:|-------------------:|-------------------:|-------------------:|-----------------:|-----------------:|-----------------:|-----------------:|-----------------:|-----------------:|-----------------:|-----------------:|-----------------:|-----------------:|-----------------:|-----------------:|--------------------:|----------------------:|
| (-0.001, 3.0] |              461 |            433 |               447 |                439 |                437 |                452 |              433 |              436 |              436 |              429 |              450 |              423 |              450 |              455 |              489 |              457 |              501 |              487 |                 508 |                   591 |
| (3.0, 4.0]    |              726 |            768 |               759 |                777 |                782 |                796 |              790 |              780 |              806 |              794 |              845 |              803 |              811 |              778 |              835 |              836 |              820 |              847 |                 804 |                   795 |
| (4.0, 4.6]    |              366 |            403 |               427 |                472 |                446 |                407 |              459 |              426 |              455 |              462 |              452 |              422 |              454 |              421 |              457 |              448 |              422 |              407 |                 411 |                   378 |
| (4.6, 5.0]    |             2638 |           2593 |              2547 |               2511 |               2514 |               2542 |             2500 |             2558 |             2481 |             2510 |             2443 |             2535 |             2476 |             2534 |             2407 |             2450 |             2446 |             2448 |                2462 |                  2426 |


Main thing of interest for this pivot table was how many recipes were binned based on not only ratings but also number of calories (Based on pdcut bins of 10). The main topic of interest is that bins (-0.001, 3.0] and (4.0, 4.6] have very similiar distributions and counts between one another in terms of calories, though (-0.001, 3.0] is a bit higher in terms of values (both are spread around the 300-500 count). Meanwhile bin (3.0, 4.0] takes 2nd place with viewable increase in counts based on calory bins (around the 800 counts) and as we see there are a lot of values in the (4.6, 5.0] bin row (Around 2500 counts on each cell!). This showcases besides the already established massive count of values near ratings 5 that the range (3.0, 4.0] is probably secondary in that count with bins (-0.001, 3.0] and (4.0, 4.6] taking 3rd and 4th respectively in terms of count as well likely having similar counts of ratings. If we do some groupbys on the rating bins and then agg some funcs (mean, median, and mode), here's our results on calories respectively:

Mean:
| rating bins   |   number of calories |
|:--------------|---------------------:|
| (-0.001, 3.0] |              477.289 |
| (3.0, 4.0]    |              426.041 |
| (4.0, 4.6]    |              410.256 |
| (4.6, 5.0]    |              425.822 |

Median:
| rating bins   |   number of calories |
|:--------------|---------------------:|
| (-0.001, 3.0] |               320.1  |
| (3.0, 4.0]    |               312.85 |
| (4.0, 4.6]    |               303.3  |
| (4.6, 5.0]    |               300.6  |

Mode:
| rating bins   |   number of calories |
|:--------------|---------------------:|
| (-0.001, 3.0] |                 71.5 |
| (3.0, 4.0]    |                239.7 |
| (4.0, 4.6]    |                159.1 |
| (4.6, 5.0]    |                176.9 |

As we see, the distribution is highly biased on mean where mean > median > mode (which makes sense when we consider the graph we made on calories). If we ever plan to do some distribution testing on the calories, we should likely avoid the mean in this scenario and determine whether we should use the mode or median.

While our pivot table above looks rather spread well, we did create other pivot tables with much less of a lack of spread between row values. For example, on our protein count pivot table to the left of the (4.6, 5.0] bin for ratings, we had a very high amount of recipes in the (-0.001, 1.0] bin under protein (4481 to be exact) in comparison to the rest of the row values (which were spread across the usual (1500 - 2500 range though one value came with 3394 under protein bin (3.0, 5.0]).

## Assessment of Missingness

#### NMAR Anaylsis

During our cleaning, we already singled out any possible missing values in our data. Here's our result below:

|                           |   Missing values |
|:--------------------------|----:|
| name                      |   1 |
| id                        |   0 |
| minutes                   |   0 |
| contributor_id            |   0 |
| submitted                 |   0 |
| tags                      |   0 |
| nutrition                 |   0 |
| n_steps                   |   0 |
| steps                     |   0 |
| description               |  70 |
| ingredients               |   0 |
| n_ingredients             |   0 |
| average_rating_per_recipe |   1 |

As you can see, we have 3 columns with missing values, "name", "average_rating_per_recipe", and "description".

Let's start with the quickest one, "average_rating_per_recipe". See this was column that was created by merged data from another dataset, "interactions", which contained per each recipe id multiple rating reviews. For each recipe id, we calcualted their mean from the ratings they've recieved in this dataset and add them as a new column in our non-merged recipe dataset. And yet despite this process we still recieved one missing value, on id 314968, "napa dave s individual breakfast casseroles". If we didn't recieve a value for this recipe on the column, it doesn't exist in the interactions dataset, or rather it never recievewed a rating. To confirm this, we can searching up the id here in the interactions dataset ( "interactions[interactions["recipe_id"] == 314968]" ) and we end up with an empty dataset. But ignoring this we know based on the data for this column is generated, we can predict accurately if a value will have missing data base on whether they have ratings in the interaction dataset. Hence the missing value in this column is not NMAR but instead MD, or "missing by design".

Next we have the "name" column. If we look at the recipe itself (id 368257), it seems based on the ingredients and the instruction steps, it seems to be some kind of salad complementary dish (minus the greens which the steps indicate to add with). I believe this column is the closest data we have to NMAR, or not missing at random. Since the dish itself simple but hard to describe a specific name (it's a sald without the greens?) I think user decided to not add a name or rather couldn't add a name that would be appropiate for said dish. (else they would've easily add one in if it could even be explained somewhat, like cookies or pasta). This is further evidenced by the added tags relating to salad dishes, which makes sense, it's a salad complentary dish when you add the greens but not exactly until you do so. What would you name that dish as then? Hence I think the missing value for this column is indeed NMAR since if the user could have had an appropiate name for the recipe, there's a very good chance they've would've named it so. Note however we can't really confirm this 100% since this is only based off of one single missing value we have for this column but based on how data is generated for this column and the likely simplicity of filling this column is,  I believe the reasoning is strong to make a basis off that it is NMAR.

Finally we have the "description" column. At a first glance, I thought this might be a potential canidate towards NMAR since if a recipe was generic enough (like say "very tasty chocolate chip cookies") it would be less reliant in needing a recipe name (since most people know a premise of a "chocolate chip cookie"). Essentially it was possible that people didn't put a description for recipes that needed less of a description. Looking at what recipes are missing descriptions however deconfirms this, since we have recipes like "apricot gorgonzola crescent appetizers" or "baked yams with spicy molasses butter" which I don't think are generic enough to be relevant towards a missing description. In fact, something like a recipe containing the word "appetizer" is a bit vague (what kind of appetizer?) hence I think one would logically put a description for a recipe titled that way. I couldn't think of any other possible NMAR reasons for this column anad hence would label it as not NMAR.

#### Missingness Dependency

Since I determined the "description" column as neither NMAR nor MD, I've done some permutation tests on "description" against other columns in my dataset to determine if certain columns may influence the missingness on the "description" column. For example, here are the results below for a permutation test of missingness on 
"description" against the column "minutes".




