# Cooking Clock: Analyzing the Relationship Between Recipe Duration and Ratings

**Name(s)**: Cecilia Li & Marina Ma

**Website Link**: https://jum010.github.io/ReceiptRatingAnalysis/

**Overview**:
This is the DSC80 project at UCSD, which is based on exploring the relationship between the average receipt ratings and different periods of cooking time.


## Introduction

Food is not only a requirement but also a source of creativity, culture, and community. As online cooking sites are on the rise, it is increasingly important for chefs, food bloggers, and food companies to understand what makes a recipe successful among users. Our project analyzes a large corpus of culinary data from an online recipe community [food.com](https://www.food.com/). The information comprises two related sets of data: one with in-depth recipe details and another recording users' ratings and reviews. In this project, we concentrate on a single essential question:

**Is there a significant difference in ratings between recipes based on their preparation time?**

Specifically, we examine whether recipes can be categorized as quick (<30 minutes), moderate (30–60 minutes), long (60–120 minutes), and extra long (>120 minutes), and if these categories differ significantly in terms of their average ratings. With the answer to this question, we can further comprehend whether cooking time, an important factor when planning recipes, shares any correlation with how pleasing a recipe is. This can be used by both content providers to tailor their recipes based on user enjoyment as well as consumers to make well-informed decisions based on available time. First, we will look into each dataset and get their information to see how it can better answer our question.

Dataset 1: Recipes

This dataset includes a total of 83,782 recipes and provides extensive details about each recipe. Key columns include:

- name: The recipe name. (Missing values in this column are NMAR, reflecting cases where the website did not display a name.)
- id: A unique recipe identifier.
- minutes: The time (in minutes) needed to prepare the recipe. This variable is central to our analysis as it allows us to group recipes by preparation time.
- contributor_id: The user ID of the person who submitted the recipe.
- submitted: The submission date.
- tags: Keywords or tags associated with the recipe.
- nutrition: Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)].
- n_steps: The number of steps in the recipe, offering insight into the complexity of instructions.
- steps: The text detailing each step of the recipe.
- description: A user-provided description of the recipe. (Missing descriptions are NMAR since they reflect omissions on the website.)

Dataset 2: Ratings

This dataset records user interactions with recipes, including a total of 731,927 instances of users' ratings. Its key columns are:

- user_id: The identifier of the user providing feedback.
- recipe_id: The unique identifier linking a review to a recipe.
- date: The date of the interaction.
- rating: The rating given by the user.
- review: The text of the user's review.

## Data Cleaning and Exploratory Data Analysis
### Data Cleaning
- Left merge the recipes and interactions datasets together.
    - This step helps us align each review to its corresponding recipe. 
- Check the columns of merged_df and their data types
    - This step helps us evaluate what data-cleaning steps are appropriate for the dataset and if we need to conduct data type conversion. 
    - Our merged_df has 234429 rows × 17 columns. 

- Fill all ratings of 0 with np.nan

    We replace all instances of rating 0 with np.nan because a 0 rating might be a placeholder for “no rating” rather than an actual score. When we go to https://www.food.com/ and try to rate one of the recipes, we find that the minimal rate is 1, and users can choose to give a review without rating. Moreover, when we look into the reviews where the rating is 0, we can find out that some reviews are positive and give compliments rather than negative complaints, which gives us more confidence to say that those 0s are not actual ratings but missing values. We found out that there are a total 15036 of missing ratings. We also create a Boolean column, missing_rating, that indicates whether the rating is missing for further assessment of missingness.

#### NMAR Analysis on these missing ratings
NMAR (Not Missing At Random) means the probability that a missing rating may be related to the unobserved value or other factors. For instance, there could be no rating for a specific receipt, or it could simply be because of forgetting (if there is neither review nor rating for a specific user interaction), or users might choose not to rate a recipe if they had an extremely negative or extremely positive experience.

* Investigating missingness due to missing user interaction

    In our dataset, we found that there are 83781 receipts with user interactions, but we have 83782 recipes in our 'recipes' dataset. This shows that one recipe has zero user reviews.

    Users don't review certain recipes usually because none has tried this recipe or they don't have a strong opinion towards the recipe and didn't submit any interaction(nether reviews nor rates). Since this is caused by unobserved value, we conclude this missingness is NMAR, and we drop this recipe from our dataset.

- Investigating the rest of missingness

    We want to find whether the users choose not to rate a recipe if they had an extremely negative or extremely positive experience. In order to see the rating distribution of these reviews, we select a random sample of reviews with missing ratings. Then, we give these chosen reviews a properly predicted rating based on our knowledges.

    Based on the description of each review, I give them each a reasonable rating, and put them in a array 'rating_sample_reviews'.

    We draw the distribution of these predicted ratings and see if there is any significant bumps at both edges of the plot.

![Alt text](<images/Distribution of Imputed Ratings.svg>)


The plot shows that the imputed ratings is more likely a normal distribution, and it suggests that the missing ratings tend to cluster around an average value rather than being skewed toward very high or very low ratings. In other words, it indicates that—based on the review text—the missing ratings do not appear to be systematically biased, which means that these users chose not to review a recipe wasn’t because they had an extremely negative or extremely positive experience.

This evidence is consistent with the missingness being either Missing Completely at Random (MCAR) or Missing at Random (MAR), rather than NMAR (Not Missing At Random). 

So, a normal (bell-shaped) distribution of imputed ratings supports the idea that the missingness in ratings is not being driven by the underlying sentiment or the value of the rating itself(not NMAR).

#### Missingness Dependency of ratings: 
In this section, we investigate whether the missingness in the rating column is systematically related to certain recipe attributes. Specifically, we test the dependency of the missing ratings on two numeric columns: minutes (cooking time) and n_ingredients (number of ingredients). For each test, we use a permutation test with the following setup:
- Test Statistic: 
We use as our test statistic the absolute difference in the mean of the numeric column between recipes with missing ratings and those with non-missing ratings.

- Significance Level: We set our significance level at 0.05.

**Permutation Test Implementation**


Dependency on Cooking Time (minutes)
- **Null Hypothesis (H₀)**:
The missingness in rating does not depend on the recipe’s cooking time. In other words, the mean cooking time is the same for recipes with missing and non-missing ratings.

- **Alternate Hypothesis (H₁)**:
The missingness in rating does depend on the recipe’s cooking time; that is, the mean cooking time differs between recipes with missing and non-missing ratings.

- **Results**: Dependency test for 'minutes': Observed difference = 51.46, p-value = 0.1020

- **Conculsion**: The observed difference in mean cooking time between recipes with missing ratings and those with non-missing ratings is 51.46 minutes. However, the permutation test yields a p-value of 0.1020, which is greater than the significance level of 0.05. Therefore, we **fail to reject** the null hypothesis. This suggests that the missingness in rating does not significantly depend on the recipe’s cooking time.

<iframe
    src="images/Permutation Distribution of Mean Differences for 'minutes'.html"
    width="800"
    height="600"
    frameborder="0"
></iframe>

Dependency on Number of Ingredients (n_ingredients)
- **Null Hypothesis (H₀)**:
The missingness in rating does not depend on the number of ingredients in the recipe. That is, the mean number of ingredients is similar between recipes with missing ratings and those with non-missing ratings.

- **Alternate Hypothesis (H₁)**:
The missingness in rating does depend on the number of ingredients; in other words, there is a significant difference in the mean number of ingredients between the two groups.

- **Results**: Dependency test for 'n_ingredients': Observed difference = 0.16, p-value = 0.0000

- **Conculsion**: In this test, the observed difference in the mean number of ingredients between recipes with missing and non-missing ratings is 0.16. The permutation test produces a p-value of 0.0000, which is below our significance level of 0.05. Thus, we **reject** the null hypothesis. This result indicates that the missingness in rating is significantly associated with the number of ingredients: recipes with missing ratings tend to have a different number of ingredients compared to those with complete ratings.

<iframe
    src="images/Permutation Distribution of Mean Differences for 'n_ingredients'.html"
    width="800"
    height="600"
    frameborder="0"
></iframe>

**Summary of Findings**:

These results suggest that the mechanism for missing ratings is likely MAR (Missing At Random) rather than MCAR (Missing Completely At Random), therefore, instead of directly dropping the missing roles, we suggested to imputate those missing rates.


**Missing rating imputation Strategy**

The dataset utilized in this study has missing ratings for some of the recipes. As user ratings are one of the main indicators to identify recipe quality, these missing values need to be addressed in a manner that represents the corresponding information regarding each recipe propertly. To do this, we employed a hybrid imputation approach that utilizes global and recipe-level information.

- Global and Recipe-Level Statistics

First, we computed the global mean and standard deviation of the ratings across all recipes with available ratings. We then grouped the data by recipe ID to calculate recipe-specific statistics – namely, the mean rating, standard deviation, and the count of ratings for each recipe. For recipes that had no observed ratings, we used the global mean as a fallback. Additionally, any missing recipe-level statistics were replaced with appropriate global values (e.g., a missing mean was set to the global mean and a missing standard deviation was set to 0).

- Step 2: Hybrid Imputation

Then, we created a hybrid imputation function to address the issue of missing ratings on a recipe-by-recipe basis:

 Adequate Data (≥5 evaluations):
For recipes that have at least five observed ratings, we fill in missing values with the recipe's average rating. The assumption here is that a recipe with many ratings has a good estimate of its average quality.

Sparse Data (<5 ratings):
For recipes with fewer than five ratings, we adopt a probabilistic approach. If there is variation in the available ratings (i.e., a nonzero standard deviation), we sample a rating from a normal distribution centered on the recipe's mean with the corresponding standard deviation. If no variation is present, we simply use the mean. This method helps capture the uncertainty inherent in having limited data for a recipe.

Clipping:
Finally, the imputed ratings are clipped to ensure they fall within the acceptable range (for example, 1 to 5), maintaining consistency with the original rating scale.

By combining these techniques, our imputation method preserves the integrity of the original data distribution and minimizes bias in subsequent analyses. The resulting imputed ratings (stored as filled_rating in our dataset) are then used for further tasks such as predictive modeling and hypothesis testing.

#### Add column 'average_rating' containing average rating per recipe.
Use our filled_ratings, we computed the mean rating for each recipe and added it as a new column, "average_rating", to our dataset.

#### Create a new 'calories' column
Since we are also curious about the relationship bwtween ratings and calories, we will get the first elemnent in 'nutrition' column which represents the amount of calories, and store them in a new column 'calories'. 

#### Drop the meaningless columns 
Since there are many columns doesn't contribute to our analysis, therefore, we will drop these columns, which are, 'contributor_id', 'tags', 'steps', 'submitted', 'description', 'ingredients', 'recipe_id', 'rating', 'review', 'missing_rating', 'filled_rating', and now our cleaned dataset has 83781 rows corresponding to 83781 recipes, 10 columns, below are the information of the missingness in these 10 columns and the information of first five rows which represent five different recipes:

| Column         | Missing Count |
|----------------|---------------|
| name           | 1             |
| id             | 0             |
| minutes        | 0             |
| nutrition      | 0             |
| n_steps        | 0             |
| n_ingredients  | 0             |
| user_id        | 0             |
| date           | 0             |
| avg_rating     | 0             |
| calories       | 0             |

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>name</th>
      <th>id</th>
      <th>minutes</th>
      <th>...</th>
      <th>user_id</th>
      <th>date</th>
      <th>avg_rating</th>
      <th>calories</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>1 brownies in the world    best ever</td>
      <td>333281</td>
      <td>40</td>
      <td>...</td>
      <td>3.87e+05</td>
      <td>2008-11-19</td>
      <td>4.0</td>
      <td>138.4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>1 in canada chocolate chip cookies</td>
      <td>453467</td>
      <td>45</td>
      <td>...</td>
      <td>4.25e+05</td>
      <td>2012-01-26</td>
      <td>5.0</td>
      <td>595.1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5</td>
      <td>412 broccoli casserole</td>
      <td>306168</td>
      <td>40</td>
      <td>...</td>
      <td>5.21e+05</td>
      <td>2017-10-17</td>
      <td>5.0</td>
      <td>194.8</td>
    </tr>
    <tr>
      <th>3</th>
      <td>6</td>
      <td>millionaire pound cake</td>
      <td>286009</td>
      <td>120</td>
      <td>...</td>
      <td>8.13e+05</td>
      <td>2008-04-09</td>
      <td>5.0</td>
      <td>878.3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>8</td>
      <td>2000 meatloaf</td>
      <td>475785</td>
      <td>90</td>
      <td>...</td>
      <td>2.22e+06</td>
      <td>2012-03-21</td>
      <td>5.0</td>
      <td>267.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 11 columns</p>
</div>

### Univariate Analysis


#### Explore the distribution of the average rating (avg_rating)


#### Explore the distribution of the recipe preparation time (minutes)


### Bivariate Analysis
Next, we want to explore the relationship between the avg_rating of recipes and some related columns. The following plots visualize the relationships.

#### Explore the relationship between the average rating of recipes and the cooking time (minutes).


#### Explore the relationship between the average rating of recipes and the cooking steps (n_steps).


#### Explore the relationship between the average rating (avg_rating) of recipes and the number of ingredients (n_ingredients).


#### Explore the relationship between the average rating (avg_rating) of recipes and the calories (first element from the 'nutrition' column).



### Interesting Aggregates

Below is the aggregate statistics data:
| time_category           | mean | std  | count |
|-------------------------|------|------|-------|
| Extra Long (>120 min)   | 4.59 | 0.67 | 8428  |
| Long (60-120 min)       | 4.62 | 0.64 | 15570 |
| Moderate (30-60 min)    | 4.61 | 0.64 | 28665 |
| Quick (<30 min)         | 4.65 | 0.61 | 30055 |


## Assessment of Missingness
When We look at our orignial dataset merged_df to see if there is any missing values in each columns that could effect the bias on our analysis, we find out that there are totoal four columns contains missing values, which are: 'name', 'description', 'rating', and 'review'.

According to previous analysis, missingness in “rating” is mostly MAR and were handled by imputation, and now we are going to look at the other three. 

| Column           | Missing Count |
|------------------|---------------|
| name             | 1             |
| id               | 0             |
| minutes          | 0             |
| contributor_id   | 0             |
| submitted        | 0             |
| tags             | 0             |
| nutrition        | 0             |
| n_steps          | 0             |
| steps            | 0             |
| description      | 114           |
| ingredients      | 0             |
| n_ingredients    | 0             |
| user_id          | 0             |
| recipe_id        | 0             |
| date             | 0             |
| rating           | 15035         |
| review           | 57            |
| missing_rating   | 0             |
| filled_rating    | 0             |
| avg_rating       | 0             |

**Missingness of 'name' and 'description'**: When we examine the imported dataset 'recipes', we observe that missing values in 'name' and 'description' reflect their absence in the original source. In other words, if a recipe's name or description is not provided on the website, it will be missing in our dataset. Therefore, the missing values in 'name' and 'description' are NMAR (Not Missing At Random) because the information was never recorded or displayed.

**Missingness of 'review'** In Step 2, when analyzing the missingness of 'rating', we found that missing ratings are not caused by the absence of user feedback—meaning, missing 'rating' values do not occur simultaneously with missing 'review' values. This suggests that even when a rating is given, the user may choose not to provide a review. Therefore, the missing values in 'review' are NMAR (Not Missing At Random) because the review was omitted despite the rating being provided.

In conclusion, the missingness in 'name', 'description', and 'review' is NMAR because these values are absent from the original data source or omitted by the user. Since these columns (name, description, review) are not central to our main analysis, therefore, dropping these columns won't have any effect or case any bias. We can safely use our cleaned_df which have 'description' and 'review' column dropped for the hypothesis test.

## Hypothesis Testing
**Question**: Is there a significant difference in ratings between quick (<30 min), moderate (30-60 min), long (60-120 min), and extra long (>120 min) recipes?

**Null Hypothesis (H0​)**: There is no significant difference in average ratings between Quick (<30 min), Moderate (30-60 min), Long (60-120 min), and Extra Long (>120 min) recipes.

**Alternative Hypothesis (H1​)**: At least one recipe category has a significantly different average rating compared to the others.

**

**Interpret results**
alpha = 0.05  # Significance level

if p_value < alpha:
    print("We reject the null hypothesis: At least one recipe category has a significantly different average rating.")
else:
    print("We fail to reject the null hypothesis: There is no significant difference in ratings among the recipe categories.")



    Observed F-statistic: 25.255264360614635
    Observed F-statistic: 25.2553
    P-value from permutation test: 0.0000
    We reject the null hypothesis: At least one recipe category has a significantly different average rating.


## Framing a Prediction Problem



## Baseline Model




## Final Model





## Fairness Analysis


