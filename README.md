# Recipe ratings against nutritional values
Project on a given data for Recipe and Ratings for DSC80


## Introduction
The following report is based off of data from food.com. The main topic of interest is whether or not the different healthy levels of nutrition variables a food contains (calories, fats, sugar, etc.) may potentially play a part in the ratings given to each recipe. This could help conclude whether how a food is appreciated by a person on food.com may by influenced by nutritional values (such as calories or fats). Of course there may obviously be other factors that could play importance, but currently we're interested in how specifically nutrients may determine a different distribution to ratings (if at all). We'll be working with 83,782 rows of data and our main two columns of focus will obviously be "nutrition" (contains a list of numerical values on a nutrients' level in a food based on:  [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)], where pdv stands for "percentage of daily value") and "average_ratings_per_recipe" (just a numerical of specifically the average rating of all ratings that were done of the given recipe). Other potentially important columns include "name" (recipe name), "submitted" (when recipe was submitted to food.com), and "description" (description of the recipe). The names and descriptions of recipes may help explain certain nutrition variable patterns which in turn may also affect ratings of specific food types. Submission dates may seem non-important however in case of duplicate food values, it may be important to compare nutrition values and rating relationships between these food values as time of submission may play a likely factor towards how they may differ.


## Cleaning and EDA

#### Data Cleaning

For each column I followed the following procedure:
1. Run df[column_name].isna() to get the amount of na values (if any) in each column to replace.
2. Run (df[column name] <= 0).sum() or (df[column name].str.len == 0).sum() to see if any values had a 0 or less or a string with essentially no value.

If the column had a na value, we would fill it with a relevant value. For example, if the column "descriptions" had nas, all the nas were filled with the string "No Descripton Given" (since all non na values in "description" column were strings)
Some columns had 0 for values but were fine to include (for example "average_ratings_recipe") so no changes were made. There also weren't any strings that had length 0 so no changes were towards any string columns.

I also checked for duplicates on certain columns (for example recipe_id) to confirm no duplicates would occur incorrectly and didn't find any hits. 
One specific column ("submitted") was converted to a datetime object else nothing much else about it was changed.

The most important variable change was that specific values in the dataframe that looked like lists were really strings (examples include "nutrition", "steps", etc.). These strings were converted to lists with the following methods:

1. "df[column_name].str.replace("'", "").str.replace("'[", "").str.replace("]", "").str.split(", ")":   (naive method, assumed there weren't any commas inside a value for example)
  
2. "df[column_name].apply(eval)":  (better method but ran much slower. Had to use on certain columns like "steps" which had commas in some values so couldn't split on that variable)
I used method 1 when possible else method 2 since method 1 seemed to computate faster, but there were still columns that had to be ran on "eval" (again "steps" for example.)
One unique column was nutrition which after I converted to lists I had to change the strings inside to floats. This was achieved use an apply function and more importantly, "map()".

Finally, since we were interested in variable levels independently, I took the nutritions column we cleaned and made a series based off of each specific index that the nutrition list would indicate based on. For example, I had a column named "number of calories" that made a series based on the first value of every list in "nutrition" (since the first value of the nutrition columns' list values represented calories). I repeated this process for all the variables that were represented in the nutrition list values, creating 7 columns for each overall.

Here's the head of our cleaned dataframe:

| name                               |     id |   minutes |   contributor_id | submitted           |   n_steps |   n_ingredients |   average_rating_per_recipe |   number of calories |   total_fat (PDV) |   sugar (PDV) |   sodium (PDV) |   protein (PDV) |   saturated_fat (PDV) |   carbohydrates (PDV) |
|:-----------------------------------|-------:|----------:|-----------------:|:--------------------|----------:|----------------:|----------------------------:|---------------------:|------------------:|--------------:|---------------:|----------------:|----------------------:|----------------------:|
| 1 brownies in the world  best ever | 333281 |        40 |           985201 | 2008-10-27 00:00:00 |        10 |               9 |                           4 |                138.4 |                10 |            50 |              3 |               3 |                    19 |                     6 |
| 1 in canada chocolate chip cookies | 453467 |        45 |          1848091 | 2011-04-11 00:00:00 |        12 |              11 |                           5 |                595.1 |                46 |           211 |             22 |              13 |                    51 |                    26 |
| 412 broccoli casserole             | 306168 |        40 |            50969 | 2008-05-30 00:00:00 |         6 |               9 |                           5 |                194.8 |                20 |             6 |             32 |              22 |                    36 |                     3 |
| millionaire pound cake             | 286009 |       120 |           461724 | 2008-02-12 00:00:00 |         7 |               7 |                           5 |                878.3 |                63 |           326 |             13 |              20 |                   123 |                    39 |
| 2000 meatloaf                      | 475785 |        90 |          2202916 | 2012-03-06 00:00:00 |        17 |              13 |                           5 |                267   |                30 |            12 |             12 |              29 |   

(Note that we dropped columns in our above data head display (specifically columns "tags", "nutrition", "steps", "description", and "ingredients") since they contained information overflow for reference sake. These columns, while dropped in this preview, are still included in real dataframe.)

#### Univariate Analysis

We'll look at graphs for our newly created nutrition columns and our average ratings' column.

###### Average ratings graph
Firstly, make note that the range of values for the average ratings' column is 0-5 which is a very small range. Hence I found it appropiate to use a box and whisker plot to demonstrate the distribution of the data. The result is below:

<iframe src="fig_average.html" width=800 height=600 frameBorder=0></iframe>

Looking at it carefully, we can clearly see some interesting behavior here. The biggest interest is that the maximum, upper quartile and median is all on the same value of 5. Our lower quartile is still quite high at 4 and an approximate minimum at around 2.43 and a large amount of listed outlier minimums on the left side of the box and whisker plot, ranging from 2.3 to 0. To me, this distribution seems to be telling us that a lot of our average ratings for this data seems to be quite high. In fact, we might be having a very large amount of data that's distrbuted around the score 5 (hence the median, upperquartile, and max all being 5). Even disregarding the 5 values, a lot of the data seems to still be shifted upwards, where 4 is the first quartile which is still quite a bit higher than expected.

###### Nutrition variables graph
The distribution of nutrients variables are all much more spread out then what we have on the average ratings. For example, while the average ratings ranged from 0 - 5.0 the column on the nutrition variable called protein ranges from a minimum of 0 to a max of 4356! (PDV). So a box and whisker isn't going to be ideal in plotting a distribution of the values here so instead we'll use something larger, a histogram. 

We still run into some value issues however, namedly we have a lot of bins with low counts that will reach out to the maximum! For example, consider this histogram on the calories column:

<iframe src="fig_calories.html" width=800 height=600 frameBorder=0></iframe>

We have ALOT of bins and most of them aren't representative of the larger population (Which is near the start at 0). So we'll have to set reduce our value ranges for x using the "x_range":

<iframe src="fig_calories_reduced.html" width=800 height=600 frameBorder=0></iframe>

Much better, we now see more clearly that the distribution goes up early before starting to skew right as calories increase to their maximum. We also begin starting to see small difference changes as the counts reduce around the 1000 calories mark. 

We end up using this process for all of the nutrition variables since all of them have some very large ranges and counts for said values in each range. The determined x_range varies between each variable but the general premise I used was that I removed the values that had the top 200 numerical values of that variable. (so in the instant of calories, setting the x_range max to 5000 reduced around 200 values that had more than 5000 calories. In fact, calories' max went up to 10,000). Here a few examples of histograms we made on protein and carbohydrates:


<iframe src="fig_carbohydrates_reduced.html" width=800 height=600 frameBorder=0></iframe>

<iframe src="fig_protein_reduced.html" width=800 height=600 frameBorder=0></iframe>


Essentially all the histograms we made on nutrition variables came to a simular conclusion: The data tended to be right skewed (or even started at the maximum count as you can see in protein) before slowly reducing in counts height and reaching a very low bar count while heading towards the maximum. So we then know for the most part the nutrition variables here tend to be bounded near a value that's clsoer to 0 then some maximum (for example calories at around 1000) and that by right skewed logic, their means >= median >= mode. But this is just the distribution on each nutrition variable separately and not together.(for example a recipe may have near a maximum on calories but near a minimum on protein) How do these variables compare to our average ratings when factored and could we perhaps compound some of them together to create a relationship?

#### Bivariate Analysis

I ended up focusing on a potential relationship between each nutrition variable and average ratings. Here one plot result for the variable calories (Note I filtered off of our previous range of the histogram used for calories to demonstrate values that weren't too extreme):

<iframe src="alt_calories_scatter.html" width=800 height=600 frameBorder=0></iframe>

The correlation trendline (in red) does demonstrate that the correlation is below 0. There's also potential interest in 2 factors. Firstly if the rating is an integer, we have a strong distribution of low and high calories. This is most viewable in rating 0, who despite its rating still has a column of calories with a similar distribution to rating 5. Another major interest is the amount of points in the higher bins of ratings (between 3.5 - 5). This aligns with the idea that our whisker plot focused greater around higher values past the first quartile (4 and above). In regards to our hypothesis question, depending on where our level of healthy calories is located, the plot implies that if above approximately 1500 calories we'll have different distributions in ratings. This is highly due to the large amount of values focused around the 4-5 bin of ratings that are situated around range 1000. Hence this scatterplot may not demonstrate a correlation between the two columns but it will be highly useful in predicting whether a healthy calory range may prove dependent on ratings since we can clearly see the borderline where we may have either similar or very different distributions between above and below a specific calory level limit. (example y=500 calories, may indicate similar distribution of values of ratings for above values and below values of y, while y=2000 columns may have very different values since we have a lot more values beneath 2000 calories and the distribution is then more clearly different in regards to the range of ratings.)

#### Interesting Aggregates

All of the columns we're working with are numerical so for proper pivot tabling, I used pd.qcut on columns of interest to create bins to pivot on. One such example where I made bins on average ratings and calories and pivot table on count is shown below:

| rating bins (down) against calory bins (right)   |   (-0.001, 58.0] |   (58.0, 91.1] |   (91.1, 119.715] |   (119.715, 146.5] |   (146.5, 171.325] |   (171.325, 196.6] |   (196.6, 222.4] |   (222.4, 248.9] |   (248.9, 276.7] |   (276.7, 305.4] |   (305.4, 337.2] |   (337.2, 370.6] |   (370.6, 407.1] |   (407.1, 448.1] |   (448.1, 498.7] |   (498.7, 563.3] |   (563.3, 650.3] |   (650.3, 783.0] |   (783.0, 1078.775] |   (1078.775, 45609.0] |
|:--------------|-----------------:|---------------:|------------------:|-------------------:|-------------------:|-------------------:|-----------------:|-----------------:|-----------------:|-----------------:|-----------------:|-----------------:|-----------------:|-----------------:|-----------------:|-----------------:|-----------------:|-----------------:|--------------------:|----------------------:|
| (-0.001, 3.0] |              461 |            433 |               447 |                439 |                437 |                452 |              433 |              436 |              436 |              429 |              450 |              423 |              450 |              455 |              489 |              457 |              501 |              487 |                 508 |                   591 |
| (3.0, 4.0]    |              726 |            768 |               759 |                777 |                782 |                796 |              790 |              780 |              806 |              794 |              845 |              803 |              811 |              778 |              835 |              836 |              820 |              847 |                 804 |                   795 |
| (4.0, 4.6]    |              366 |            403 |               427 |                472 |                446 |                407 |              459 |              426 |              455 |              462 |              452 |              422 |              454 |              421 |              457 |              448 |              422 |              407 |                 411 |                   378 |
| (4.6, 5.0]    |             2638 |           2593 |              2547 |               2511 |               2514 |               2542 |             2500 |             2558 |             2481 |             2510 |             2443 |             2535 |             2476 |             2534 |             2407 |             2450 |             2446 |             2448 |                2462 |                  2426 |


Main thing of interest for this pivot table was how many recipes were binned based on not only ratings but also number of calories (Based on pdcut bins of 10). Notice that bins (-0.001, 3.0] and (4.0, 4.6] have very similiar distributions and counts between one another in terms of calories, though (-0.001, 3.0] is a bit higher in terms of values (both are spread around the 300-500 count). Meanwhile bin (3.0, 4.0] takes 2nd place with viewable increase in counts based on calory bins (around the 800 counts) and as we see there are a lot of values in the (4.6, 5.0] bin row (Around 2500 counts on each cell!). Besides further showcasing the already established massive count of values near ratings 5, the table indicates that the range (3.0, 4.0] is likely secondary in that count with bins (-0.001, 3.0] and (4.0, 4.6] taking 3rd and 4th respectively in terms of count as well potentially having similar counts of ratings between one another. If we do some groupbys on the rating bins and then agg some funcs (mean, median, and mode), here's our results on calories respectively:

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

While our pivot table above looks rather spread out equally, we did create other pivot tables with much less of a lack of spread between row values. For example, on our protein count pivot table to the left of the (4.6, 5.0] bin for ratings, we had a very high amount of recipes in the (-0.001, 1.0] protein bin (4481 to be exact) in comparison to the rest of the row values (which were spread across the usual 1500 - 2500 range, though one value came with 3394 under protein bin (3.0, 5.0]).

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

Let's start with the quickest one, "average_rating_per_recipe". This was a column that was created by merged data from another dataset, "interactions", which contained, per each recipe id, multiple rating reviews. For each unique recipe id, we calcualted their mean from the ratings they've recieved in this dataset and add them as a new column in our non-merged recipe dataset. And yet despite this process we still recieved one missing value, on id 314968, "napa dave s individual breakfast casseroles". If we didn't recieve a value for this recipe on the column, it doesn't exist in the interactions dataset, or rather it never recieved a rating. To confirm this, we can searching up the id here in the interactions dataset ( "interactions[interactions["recipe_id"] == 314968]" ) and we end up with an empty dataset. But ignoring this, we know based on how the data for this column is generated, we can predict accurately if a value will have missing data depending on whether they have ratings in the interaction dataset. Hence the missing value in this column is not NMAR but instead MD, or "missing by design".

Next we have the "name" column. If we look at the recipe itself (id 368257), based on info from the "ingredients" and the "instruction step" columns, it seems to be some kind of salad complementary dish (minus the greens which the steps indicate to add with). I believe this column is the closest data we have to NMAR, or not missing at random. Since the dish itself is simple but hard to describe a specific name (it's a sald without the greens?), I think the user decided to not add a name or rather couldn't add a name that would be appropiate for said dish. (else they would've easily add one in if it could even be explained somewhat, like cookies or pasta). This is further evidenced by the added tags relating to salad dishes, which makes sense, it's effectively a salad dish once you've add the greens but not exactly until you do so. What would you name that dish as then? Hence I think the missing value for this column is indeed NMAR since if the user could have had an appropiate name for the recipe, there's a very good chance they would've named it so. Note we can't really confirm this 100% since this is only based off of one single missing value we have for this column, however from how data is generated for this column and the likely simplicity of filling out this column could be,  I believe the reasoning is strong enough to make a basis off that the column is NMAR.

Finally we have the "description" column. At a first glance, I thought this might be a potential canidate towards NMAR since if a recipe was generic enough (like say "very tasty chocolate chip cookies") it would be less reliant in needing a recipe name (since most people know a premise of a "chocolate chip cookie"). Essentially. it was possible that people didn't put a description for recipes that needed less of a description. Looking at what recipes are missing descriptions however deconfirms this, since we have recipes like "apricot gorgonzola crescent appetizers" or "baked yams with spicy molasses butter" which I don't think are generic enough to be relevant towards a missing description. In fact, something like a recipe containing the word "appetizer" is a bit vague (what kind of appetizer?) hence I think one would logically put a description for a recipe titled that way. I couldn't think of any other possible NMAR reasons for this column anad hence I would label it as not NMAR.

#### Missingness Dependency

Since I determined the "description" column as neither NMAR nor MD, I've done some permutation tests on "description" against other columns in my dataset to determine if certain columns may influence the missingness on the "description" column. For example, here are the results below for a permutation test of missingness on "description" against the column "minutes".

<iframe src="Distribution_missingess_minutes.html" width=800 height=600 frameBorder=0></iframe>

Hovering over the redline (which represents our observed statistic for our dataset), we end up seeing the distribution on missingness against minutes seems to be situated in the highly common area (for reference, the calculated p value we got on this dataset was approximately 0.592, where we did the absolute difference between means for our test statistic). So we know our observed statistic ended up seeming quite common among simulations but let's anaylze further. Below is the data distributions of minutes for values with a missing description value and values without a missing description value (both on a histogram):

<iframe src="minutes_distribution_isna.html" width=800 height=600 frameBorder=0></iframe>

<iframe src="minutes_distribution_notna.html" width=800 height=600 frameBorder=0></iframe>

Notice how similar both distributions are! Recall from lecture 7 that if a column was not MAR to another column, that secondary column would have the same distribution for both non missing values on the first column and missing values on the first column. So with that piece of info in mind, since both plots have similar distributions to one another, we conclude that the missing values of "description" are not MAR with the "minutes" column.

Let's try another column. Suppose I decided to now do permutations tests with a test statistic of the absolute difference between means for missing values of "description" on the column "n_ingredients" (or the number of ingredients a recipe has). Here's our result:

<iframe src="missingness_descrip_and_n_ingredients" width=800 height=600 frameBorder=0></iframe>

Notice our observed statistic is signficantly farther from the distribution to the rest of data. Here our calculated p_value for this test 0.004 (likely really ranging from 0 to 0.005 on a normal basis approximately). This then implies evidence against the null hypothesis here that the missingness of the "description" is not dependent on the column "n_ingredients", but let's try looking further by doing distribution plots on "n_ingredients" for missing values on "description" and non-missing values on "description":

<iframe src="n_ingredients_distribution_isna.html" width=800 height=600 frameBorder=0></iframe>

<iframe src="n_ingredients_distribution_notna.html" width=800 height=600 frameBorder=0></iframe>

Notice how unlike the distributions on "minutes", the distribution for "n_ingredients" seems to be very different between missing and non missing values of "description"!. Therefore since the two distribution plots seem non-similar to one another and we have a very low p significant value, we can conclude that there's a strong possibility that "description" could be MAR on the column "n_ingredients". As for good reasoning towards why this might be the case, consider my previous description of "description" where I considered certain recipes that were simplistic (chocolate chip cookies) may not necessarily warranted a description. Something similar could be happening with the "n_ingredients" column, where certain recipes have self explanatory types of ingredients (such as a salad: greens, olives, tomatoes, etc.) and a lower number of ingredients required could warrant less of a need for a description (especially if the ingredients themselves are already self explanatory and not as unique). This is further evidenced by how overall the distribution values of "n_ingredients" on missing values were on average lower than the distribution values of "n_ingredients" on non-missing values.


Other columns that we commited permutations test are available on our python file, which includes no evidence for MAR on average ratings of recipes, number of steps, and name of a recipe (used string length of the recipe to check); and evidence for MAR on the number of days from the day submitted to the current day (11/6/2023). 

Notice how our columns of missing values for description isn't really relevant to our project question at a first glance. While this may be true, it's still important to consider, especially if one of our columns that we aimed to hypothesize about ended up helping to determine the missing values. For example, suppose we found the average ratings of a recipe determined MAR of missing values in column "description". If that was the case, we might be more wary on how we commited tests on the average ratings since it may or may not have been deliberately engineered to be (or at least look to be) MAR. Or what if we tested one of nutrition values (say calories) on the missingess and recieved evidence for MAR? Well then even if we later determine calories to not be related to average ratings, we could say that this data may be skewed due to the fact that we don't know if calories being MAR for a column of missing values is natural or not for such a dataset, meaning said columns and result may not necessarily be 100% true and filtered. We could also come to a similar conclusion even if calories were found to be related to average ratings. Hence even though the missingness column isn't really relevant to our question, it's still important to analyze it since it may play a larger role in one of the columns we want to test or even biase a result because of it.

## Hypothesis Testing



