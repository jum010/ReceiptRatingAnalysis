# Cooking Clock: Analyzing the Relationship Between Recipe Duration and Ratings

**Name(s)**: Cecilia Li & Marina Ma

**Website Link**: https://jum010.github.io/ReceiptRatingAnalysis/

**Overview**:
This is the DSC80 project at UCSD, which is based on exploring the relationship between the average receipt ratings and different periods of cooking time.

<iframe
  src="images/Average Rating vs Calories without outliers.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


## Step 1: Introduction

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

## Step 2: Data Cleaning and Exploratory Data Analysis
### Data Cleaning
#### Left merge the recipes and interactions datasets together.
- This step helps us align each review to its corresponding recipe. 
#### Check the columns of merged_df and their data types
This step helps us evaluate what data-cleaning steps are appropriate for the dataset and if we need to conduct data type conversion.
Our merged_df has 234429 rows × 17 columns. 

#### Fill all ratings of 0 with np.nan
We replace all instances of rating 0 with np.nan because a 0 rating might be a placeholder for “no rating” rather than an actual score. When we go to https://www.food.com/ and try to rate one of the recipes, we find that the minimal rate is 1, and users can choose to give a review without rating. Moreover, when we look into the reviews where the rating is 0, we can find out that some reviews are positive and give compliments rather than negative complaints, which gives us more confidence to say that those 0s are not actual ratings but missing values. We found out that there are a total 15036 of missing ratings. We also create a Boolean column, missing_rating, that indicates whether the rating is missing for further assessment of missingness.

#### NMAR Analysis on these missing ratings
NMAR (Not Missing At Random) means the probability that a missing rating may be related to the unobserved value or other factors. For instance, there could be no rating for a specific receipt, or it could simply be because of forgetting (if there is neither review nor rating for a specific user interaction), or users might choose not to rate a recipe if they had an extremely negative or extremely positive experience.

##### Investigating missingness due to missing user interaction
    In our dataset, we found that there are 83781 receipts with user interactions, but we have 83782 recipes in our 'recipes' dataset. This shows that one recipe has zero user reviews.



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
      <th>name</th>
      <th>id</th>
      <th>minutes</th>
      <th>contributor_id</th>
      <th>...</th>
      <th>description</th>
      <th>ingredients</th>
      <th>n_ingredients</th>
      <th>missing_interaction</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>142782</th>
      <td>napa dave s individual breakfast casseroles</td>
      <td>314968</td>
      <td>45</td>
      <td>238966</td>
      <td>...</td>
      <td>these are great for guests to grab on the go! ...</td>
      <td>['country sausage', 'eggs', 'cheddar cheese', ...</td>
      <td>5</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
<p>1 rows × 13 columns</p>
</div>

Users don't review certain recipes usually because they don't have a strong opinion towards the recipes. Since this is caused by unobserved value, we conclude this missingness is NMAR, and we drop this recipe from our dataset.

##### Investigating the rest of missingness
We want to find whether the users choose not to rate a recipe if they had an extremely negative or extremely positive experience. In order to see the rating distribution of these reviews, we select a random sample of reviews with missing ratings. Then, we give these chosen reviews a properly predicted rating based on our knowledges.



Based on the description of each review, I give them each a reasonable rating, and put them in a array 'rating_sample_reviews'

| Reveiws | Predicted Rating |
| -------- | -------- |
| Do you need vanilla for this? cause I don't have any left after I last baked... ;-; EDIT: I made them, no vanilla. Still tastes delicious. | 4 |
| Do you use grated Parmesan cheese or shredded ?? | 2 |
| This does sound good, kinda like I smother chicken.  Will try soon. | 3 |
| These are delicious, I have been making them for years.  At Christmas, I use scallions & roasted red peppers along with the bacon. I have never tried the parmesan cheese.   \n\nWhat no one mentioned is that these freeze beautifully.  Make up to instruction #8, put on a waxed paper lined cookie sheet & freeze for 60 minutes and then place in a freezer bag.  You then just have to take out as many as you want to bake & follow the remaining instructions, baking from a frozen state & just add a couple of minutes to the baking time.  Depending upon whether I am serving these in place of rolls for dinner or as an appetizer, depends on the side I roll from, bigger for dinner rolls and smaller for appetizers.  These are always a hit!!  Thank you for posting the recipe! | 5 |
| Surprisingly enough, Horse meat works fine as well.; and much leaner than beef. | 4 |
| Can you replace the Milk, with any other Milk products. I have Celiac's disease also Lactose intolerance. | 2 |
| My husband loved this, I thought it was ok. The stuffing flavor was quite good. I would suggest using a thinly sliced bacon if you like your bacon crisp. In the given 60 minutes, it was overcooked and the bacon was somewhat undercooked, still kind of rubbery. As a precaution I tied the rolled roast. Next time: Once assembled & tied, I will  sear the bacon to insure it is thoroughly cooked & more appealing. I will definitely adjust the baking time and use a thinner bacon. Also, I could find no possible way to use the dripping to make gravy. I'm a good gravy maker but just couldn't make it happen. | 3 |
| Did anyone Use all coconut flour no almond flour at all?. | 1 |
| sounds yummy , will have to make some up tonight.\r\nHave a question on how long these keep for ? | 3 |
| Fantastic...just as it is. I made this recently for my 50th B-day Party. I got one bite from a friends plate because the bowl was cleaned out before I could get a full serving. | 5 |


We draw the distribution of these predicted ratings and see if there is any significant bumps at both edges of the plot.


It turns out that the imputed ratings is more likely a normal distribution, and it suggests that the missing ratings tend to cluster around an average value rather than being skewed toward very high or very low ratings. In other words, it indicates that—based on the review text—the missing ratings do not appear to be systematically biased, which means that these users chose not to review a recipe wasn’t because they had an extremely negative or extremely positive experience.

This evidence is consistent with the missingness being either Missing Completely at Random (MCAR) or Missing at Random (MAR), rather than NMAR (Not Missing At Random). 

So, a normal (bell-shaped) distribution of imputed ratings supports the idea that the missingness in ratings is not being driven by the underlying sentiment or the value of the rating itself(not NMAR).

#### Missingness Dependency of ratings: Permutation Test Implementation

// TODO, need hypothesis,etc

    Dependency test for 'minutes': Observed difference = 51.46, p-value = 0.1020


**Test Results**:

The permutation tests indicate different relationships for each predictor:

For minutes, the observed difference between missing and non‐missing ratings is 51.46 with a p-value of 0.1020. This p-value is above the typical 0.05 threshold, suggesting that there isn’t strong evidence to conclude that missing ratings depend on how long a recipe takes. In other words, the cooking time (minutes) does not seem to be significantly related to whether a rating is missing.

For n_ingredients, the observed difference is 0.16 with a p-value of 0.0000. This very low p-value indicates a statistically significant dependency: the number of ingredients is associated with the missingness of the ratings. Although the numeric difference might be small, the significance implies that recipes with different numbers of ingredients have systematically different probabilities of having a missing rating.

These results suggest that the mechanism for missing ratings is likely MAR (Missing At Random) rather than MCAR (Missing Completely At Random), therefore, iinstead of directly dropping the missing roles, we suggested to imputate those missing rates.


#### Missing rating imputation

//TODO

```python
# Compute global statistics: global mean and standard deviation
global_mean_rating = merged_df['rating'].mean()
global_std_rating = merged_df['rating'].std()

# Compute recipe-level stats: mean, std, and count per recipe
recipe_stats = merged_df.groupby('id')['rating'].agg(['mean', 'std', 'count'])
# Replace missing means with the global mean and missing std with 0
recipe_stats['mean'] = recipe_stats['mean'].fillna(global_mean_rating)
recipe_stats['std'] = recipe_stats['std'].fillna(0)

def hybrid_impute_rating(row, min_rating=1, max_rating=5, threshold=5):
    """
    Impute a missing rating for a recipe using a hybrid approach:
    
    - For recipes with at least 'threshold' observed ratings, use the recipe's mean rating.
    - For recipes with fewer than 'threshold' ratings, use a probabilistic imputation:
         If there is variation (std > 0), sample from a normal distribution
         with the recipe's mean and std; if std is 0, just use the mean.
    - If the recipe is not in recipe_stats or has zero reviews, use the global mean.
    - Finally, clip the imputed value to be within the allowed range.
    """
    # If rating is not missing, return the original rating.
    if not np.isnan(row['rating']):
        return row['rating']
    
    recipe_id = row['id']
    
    # If the recipe is not in the computed stats, fallback to the global mean.
    if recipe_id not in recipe_stats.index:
        return global_mean_rating
    
    # Retrieve recipe-specific statistics.
    mean = recipe_stats.loc[recipe_id, 'mean']
    std = recipe_stats.loc[recipe_id, 'std']
    count = recipe_stats.loc[recipe_id, 'count']
    
    # If there are no observed ratings for this recipe, fallback to the global mean.
    if count == 0:
        return global_mean_rating
    
    # Case 1: Sufficient data (>= threshold): use the recipe's mean.
    if count >= threshold:
        imputed = mean
    # Case 2: Few ratings (< threshold): use a probabilistic imputation.
    else:
        if std > 0:
            imputed = np.random.normal(mean, std)
        else:
            imputed = mean
    
    # Ensure the imputed rating falls within the valid range (e.g., 1 to 5).
    imputed = np.clip(imputed, min_rating, max_rating)
    return imputed

# Apply the improved hybrid imputation function to each row.
merged_df['filled_rating'] = merged_df.apply(hybrid_impute_rating, axis=1)
merged_df.head()
```




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
      <th>name</th>
      <th>id</th>
      <th>minutes</th>
      <th>contributor_id</th>
      <th>...</th>
      <th>rating</th>
      <th>review</th>
      <th>missing_rating</th>
      <th>filled_rating</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1 brownies in the world    best ever</td>
      <td>333281</td>
      <td>40</td>
      <td>985201</td>
      <td>...</td>
      <td>4.0</td>
      <td>These were pretty good, but took forever to ba...</td>
      <td>False</td>
      <td>4.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1 in canada chocolate chip cookies</td>
      <td>453467</td>
      <td>45</td>
      <td>1848091</td>
      <td>...</td>
      <td>5.0</td>
      <td>Originally I was gonna cut the recipe in half ...</td>
      <td>False</td>
      <td>5.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>412 broccoli casserole</td>
      <td>306168</td>
      <td>40</td>
      <td>50969</td>
      <td>...</td>
      <td>5.0</td>
      <td>This was one of the best broccoli casseroles t...</td>
      <td>False</td>
      <td>5.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>412 broccoli casserole</td>
      <td>306168</td>
      <td>40</td>
      <td>50969</td>
      <td>...</td>
      <td>5.0</td>
      <td>I made this for my son's first birthday party ...</td>
      <td>False</td>
      <td>5.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>412 broccoli casserole</td>
      <td>306168</td>
      <td>40</td>
      <td>50969</td>
      <td>...</td>
      <td>5.0</td>
      <td>Loved this.  Be sure to completely thaw the br...</td>
      <td>False</td>
      <td>5.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 19 columns</p>
</div>




```python
merged_df.isna().sum().to_dict()
```




    {'name': 1,
     'id': 0,
     'minutes': 0,
     'contributor_id': 0,
     'submitted': 0,
     'tags': 0,
     'nutrition': 0,
     'n_steps': 0,
     'steps': 0,
     'description': 114,
     'ingredients': 0,
     'n_ingredients': 0,
     'user_id': 0,
     'recipe_id': 0,
     'date': 0,
     'rating': 15035,
     'review': 57,
     'missing_rating': 0,
     'filled_rating': 0}




```python
# Find the average rating per recipe, as a Series.
avg_rating_per_recipe = merged_df.groupby('id')['filled_rating'].mean()

# Map the average rating to merged_df
merged_df['avg_rating'] = merged_df['id'].map(avg_rating_per_recipe)

# Drop the duplicates and name it cleaned_df so we don't modify orginal merged_df in case further needed.
cleaned_df = merged_df.drop_duplicates(subset=['id'], keep='last')

# Drop the meaningless columns 
cleaned_df = cleaned_df.drop(columns = ['contributor_id', 'submitted', 'description', 'recipe_id', 'rating', 'review', 'missing_rating', 'filled_rating'])
display(cleaned_df.head())
print(cleaned_df.info())
cleaned_df.isna().sum().to_dict()
```


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
      <th>name</th>
      <th>id</th>
      <th>minutes</th>
      <th>tags</th>
      <th>...</th>
      <th>n_ingredients</th>
      <th>user_id</th>
      <th>date</th>
      <th>avg_rating</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1 brownies in the world    best ever</td>
      <td>333281</td>
      <td>40</td>
      <td>['60-minutes-or-less', 'time-to-make', 'course...</td>
      <td>...</td>
      <td>9</td>
      <td>3.87e+05</td>
      <td>2008-11-19</td>
      <td>4.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1 in canada chocolate chip cookies</td>
      <td>453467</td>
      <td>45</td>
      <td>['60-minutes-or-less', 'time-to-make', 'cuisin...</td>
      <td>...</td>
      <td>11</td>
      <td>4.25e+05</td>
      <td>2012-01-26</td>
      <td>5.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>412 broccoli casserole</td>
      <td>306168</td>
      <td>40</td>
      <td>['60-minutes-or-less', 'time-to-make', 'course...</td>
      <td>...</td>
      <td>9</td>
      <td>5.21e+05</td>
      <td>2017-10-17</td>
      <td>5.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>millionaire pound cake</td>
      <td>286009</td>
      <td>120</td>
      <td>['time-to-make', 'course', 'cuisine', 'prepara...</td>
      <td>...</td>
      <td>7</td>
      <td>8.13e+05</td>
      <td>2008-04-09</td>
      <td>5.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2000 meatloaf</td>
      <td>475785</td>
      <td>90</td>
      <td>['time-to-make', 'course', 'main-ingredient', ...</td>
      <td>...</td>
      <td>13</td>
      <td>2.22e+06</td>
      <td>2012-03-21</td>
      <td>5.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 12 columns</p>
</div>


    <class 'pandas.core.frame.DataFrame'>
    Index: 83781 entries, 0 to 234428
    Data columns (total 12 columns):
     #   Column         Non-Null Count  Dtype  
    ---  ------         --------------  -----  
     0   name           83780 non-null  object 
     1   id             83781 non-null  int64  
     2   minutes        83781 non-null  int64  
     3   tags           83781 non-null  object 
     4   nutrition      83781 non-null  object 
     5   n_steps        83781 non-null  int64  
     6   steps          83781 non-null  object 
     7   ingredients    83781 non-null  object 
     8   n_ingredients  83781 non-null  int64  
     9   user_id        83781 non-null  float64
     10  date           83781 non-null  object 
     11  avg_rating     83781 non-null  float64
    dtypes: float64(2), int64(4), object(6)
    memory usage: 8.3+ MB
    None





    {'name': 1,
     'id': 0,
     'minutes': 0,
     'tags': 0,
     'nutrition': 0,
     'n_steps': 0,
     'steps': 0,
     'ingredients': 0,
     'n_ingredients': 0,
     'user_id': 0,
     'date': 0,
     'avg_rating': 0}



### Univariate Analysis

#### Explore the distribution of the average rating (avg_rating)


```python
# We first examined the distribution of the avg_rating column. The following Plotly histogram shows how the average ratings are distributed across all recipes.
ave_rating_fig = px.histogram(
    cleaned_df, 
    x='avg_rating', 
    nbins=20, 
    title='Distribution of Average Recipe Ratings',
    labels={'avg_rating': 'Average Rating'}
)
# ave_rating_fig.show()
```

#### Explore the distribution of the recipe preparation time (minutes)


```python
# Distribution of Recipe Preparation Time (minutes)
prep_time_fig = px.histogram(cleaned_df, 
                             x='minutes', 
                             nbins=50, 
                             title="Distribution of Recipe Preparation Time",
                             labels={'minutes': 'Cooking Time (months)'}
                             )
# prep_time_fig.show()

# Show the outliers clearly
prep_time_scatter_plt = px.scatter(cleaned_df, x='minutes', y='avg_rating',
                  title='Average Rating vs Cooking Time',
                  labels={'minutes': 'Cooking Time (months)', 'avg_rating': 'Average Rating'})
# prep_time_scatter_plt.show()

fig_violin = px.violin(
    cleaned_df, 
    y='minutes', 
    box=True,        # overlay a boxplot
    points='all',    # show all points
    title='Violin Plot of Cooking Time', 
    labels={'minutes': 'Cooking Time (months)'}
)
# fig_violin.show()
```


```python
filtered_mins = cleaned_df[cleaned_df['minutes'] <= 100]
print(filtered_mins.shape[0])

fig_violin = px.violin(
    filtered_mins, 
    y='minutes', 
    box=True,        # overlay a boxplot
    points='all',    # show all points
    title='Violin Plot of Cooking Time', 
    labels={'minutes': 'Cooking Time (minutes)'}
)
# fig_violin.show()

```

    73463



```python

```


```python
# Define lower and upper bounds based on percentiles for the 'minutes' column
lower_bound = cleaned_df['minutes'].quantile(0.01)
upper_bound = cleaned_df['minutes'].quantile(0.99)

# Filter the DataFrame to remove outliers in cooking time
filtered_df = cleaned_df[(cleaned_df['minutes'] >= lower_bound) & (cleaned_df['minutes'] <= upper_bound)]
fig = px.histogram(filtered_df, x='minutes', nbins=50, title="Distribution of Recipe Preparation Time")
# fig.show()
```

### Bivariate Analysis
Next, we want to explore the relationship between the avg_rating of recipes and some related columns. The following plots visualize the relationships.

#### Explore the relationship between the average rating of recipes and the cooking time (minutes).


```python
# Relationship between the the average rating of recipes and cooking time (minutes).
avg_rating_vs_minutes = px.scatter(cleaned_df, x='minutes', y='avg_rating',
                                   title='Average Rating vs Cooking Time',
                                   labels={'minutes': 'Cooking Time (months)', 'avg_rating': 'Average Rating'})
# avg_rating_vs_minutes.show()

# Calculate the Pearson correlation coefficient between cooking time and average rating
corr_value = cleaned_df[['minutes', 'avg_rating']].corr().iloc[0, 1]
print("Pearson correlation coefficient between cooking time and average rating:", corr_value)

# Relationship between the the average rating of recipes and cooking time (minutes) without outliers. 
avg_rating_vs_minutes_without_outlier = px.scatter(filtered_df, x='minutes', y='avg_rating',
                                                   title='Average Rating vs Cooking Time',
                                                   labels={'minutes': 'Cooking Time (months)', 'avg_rating': 'Average Rating'})
# avg_rating_vs_minutes_without_outlier.show()

# Calculate the Pearson correlation coefficient between cooking time and average rating
corr_value = filtered_df[['minutes', 'avg_rating']].corr().iloc[0, 1]
print("Pearson correlation coefficient between cooking time and average rating:", corr_value)

```

    Pearson correlation coefficient between cooking time and average rating: 0.0015293559496345089
    Pearson correlation coefficient between cooking time and average rating: -0.029727448059430006


#### Explore the relationship between the average rating of recipes and the cooking steps (n_steps).


```python
# Since n_steps is a relatively small integer value, we can aggregate by n_steps.
# Group by the number of steps and calculate the mean average rating.
steps_agg = cleaned_df.groupby('n_steps')['avg_rating'].mean().reset_index()

# Create a line plot with markers to show the trend.
fig_steps_scatter = px.scatter(
    steps_agg, 
    x='n_steps', 
    y='avg_rating', 
    title='Average Rating vs Number of Cooking Steps',
    labels={'n_steps': 'Number of Cooking Steps', 'avg_rating': 'Average Rating'},
    trendline='ols',         # optional, adds a linear trendline
    trendline_color_override='red'
)
# fig_steps_scatter.show()

# Compute the Pearson correlation coefficient between number os cooking steps and average rating
corr_value = cleaned_df[['n_steps', 'avg_rating']].corr().iloc[0, 1]
print("Pearson correlation coefficient between the number of cooking steps and average rating:", corr_value)
```

    Pearson correlation coefficient between the number of cooking steps and average rating: 0.004897061087248975


#### Explore the relationship between the average rating (avg_rating) of recipes and the number of ingredients (n_ingredients).


```python
# Group by the number of ingredients and calculate the mean average rating for each group
ingredients_agg = cleaned_df.groupby('n_ingredients')['avg_rating'].mean().reset_index()

# Create a line plot with markers to display the trend
fig_ingredients = px.line(ingredients_agg, x='n_ingredients', y='avg_rating', markers=True,
                          title='Average Rating vs Number of Ingredients',
                          labels={'n_ingredients': 'Number of Ingredients', 'avg_rating': 'Average Rating'})
# fig_ingredients.show()

# Calculate the Pearson correlation coefficient between the number of ingredients and average rating
corr_value = cleaned_df[['n_ingredients', 'avg_rating']].corr().iloc[0, 1]
print("Pearson correlation coefficient between the number of ingredients and average rating:", corr_value)
```

    Pearson correlation coefficient between the number of ingredients and average rating: -0.003959700246042612


#### Explore the relationship between the average rating (avg_rating) of recipes and the calories (first element from the 'nutrition' column).


```python
# Create a new 'calories' column by applying the function.
cleaned_df['calories'] = cleaned_df['nutrition'].apply(lambda x: float(x.strip('[]').split(',')[0]) if pd.notna(x) else np.nan)

# Plot the distribution of calories
fig_calories = px.histogram(
    cleaned_df,
    x='calories',
    nbins=30,
    title='Distribution of Calories in Recipes',
    labels={'calories': 'Calories'}
)
# fig_calories.show()

# Plot the relationship between calories and average rating
fig_calories_scatter = px.scatter(
    cleaned_df,
    x='calories',
    y='avg_rating',
    title='Average Rating vs Calories',
    labels={'calories': 'Calories', 'avg_rating': 'Average Rating'}
)
# fig_calories_scatter.show()

# Compute the Pearson correlation coefficient between calories and average rating
corr_value = cleaned_df[['calories', 'avg_rating']].corr().iloc[0, 1]
print("Pearson correlation coefficient between calories and average rating:", corr_value)
```

    Pearson correlation coefficient between calories and average rating: -0.0011735817307947881



```python
# Filter the DataFrame to remove outliers in calories
lower_bound = cleaned_df['calories'].quantile(0.01)
upper_bound = cleaned_df['calories'].quantile(0.99)

# Filter the DataFrame to keep only rows within the bounds
filtered_calories = cleaned_df[(cleaned_df['calories'] >= lower_bound) & (cleaned_df['calories'] <= upper_bound)]

# Plot the distribution of calories without outliers
fig_calories_without_outliers = px.histogram(
    filtered_calories,
    x='calories',
    nbins=30,
    title='Distribution of Calories without outliers in Recipes',
    labels={'calories': 'Calories'}
)
# fig_calories_without_outliers.show()

# Plot the relationship between average rating and calories without outliers
fig_scatter_without_outliers = px.scatter(
    filtered_calories,
    x='calories',
    y='avg_rating',
    title='Average Rating vs Calories without outliers',
    labels={'calories': 'Calories', 'avg_rating': 'Average Rating'}
)
# fig_scatter_without_outliers.show()
```

### Interesting Aggregates


```python
def categorize_time(minutes):
    if minutes < 30:
        return "Quick (<30 min)"
    elif minutes < 60:
        return "Moderate (30-60 min)"
    elif minutes < 120:
        return "Long (60-120 min)"
    else:
        return "Extra Long (>120 min)"

filtered_df['time_category'] = filtered_df['minutes'].apply(categorize_time)
grouped_stats = filtered_df.groupby('time_category')['avg_rating'].agg(['mean', 'std', 'count']).reset_index()
print("Aggregate statistics by time category:")
print(grouped_stats)

# Define the category order explicitly for sorting
category_order = ["Quick (<30 min)", "Moderate (30-60 min)", "Long (60-120 min)", "Extra Long (>120 min)"]

# Create a box plot with the specified category order
fig = px.box(
    filtered_df, 
    x='time_category', 
    y='avg_rating', 
    title="Comparison of Ratings by Time Category",
    labels={'time_category': 'Time Category', 'average_rating': 'Average Rating'},
    category_orders={"time_category": category_order}  # Ensure correct order
)

# Show the figure
# fig.show()
```

    Aggregate statistics by time category:
               time_category  mean   std  count
    0  Extra Long (>120 min)  4.59  0.67   8428
    1      Long (60-120 min)  4.62  0.64  15570
    2   Moderate (30-60 min)  4.61  0.64  28665
    3        Quick (<30 min)  4.65  0.61  30055


## Step 3: Assessment of Missingness


```python
# We want to look at our orignial dataset merged_df to see if there is any missing values in each columns that could effect the bias on our analysis. 
display(merged_df.isna().sum().to_dict())

# There are totoal four columns contains missing values, which are: 'name', 'description', 'rating', and 'review'.
# According to previous analysis in Step 2, missingness in “rating” is mostly MAR and were handled by imputation.
# We will then need to look at the other three. 
```


    {'name': 1,
     'id': 0,
     'minutes': 0,
     'contributor_id': 0,
     'submitted': 0,
     'tags': 0,
     'nutrition': 0,
     'n_steps': 0,
     'steps': 0,
     'description': 114,
     'ingredients': 0,
     'n_ingredients': 0,
     'user_id': 0,
     'recipe_id': 0,
     'date': 0,
     'rating': 15035,
     'review': 57,
     'missing_rating': 0,
     'filled_rating': 0,
     'avg_rating': 0}


**Missingness of 'name' and 'description'**: When we examine the imported dataset 'recipes', we observe that missing values in 'name' and 'description' reflect their absence in the original source. In other words, if a recipe's name or description is not provided on the website, it will be missing in our dataset. Therefore, the missing values in 'name' and 'description' are NMAR (Not Missing At Random) because the information was never recorded or displayed.

**Missingness of 'review'** In Step 2, when analyzing the missingness of 'rating', we found that missing ratings are not caused by the absence of user feedback—meaning, missing 'rating' values do not occur simultaneously with missing 'review' values. This suggests that even when a rating is given, the user may choose not to provide a review. Therefore, the missing values in 'review' are NMAR (Not Missing At Random) because the review was omitted despite the rating being provided.

In conclusion, the missingness in 'name', 'description', and 'review' is NMAR because these values are absent from the original data source or omitted by the user. Since these columns (name, description, review) are not central to our main analysis, therefore, dropping these columns won't have any effect or case any bias. We can safely use our cleaned_df which have 'description' and 'review' column dropped for the hypothesis test.

## Step 4: Hypothesis Testing
**Question**: Is there a significant difference in ratings between quick (<30 min), moderate (30-60 min), long (60-120 min), and extra long (>120 min) recipes?

**Null Hypothesis (H0​)**: There is no significant difference in average ratings between Quick (<30 min), Moderate (30-60 min), Long (60-120 min), and Extra Long (>120 min) recipes.

**Alternative Hypothesis (H1​)**: At least one recipe category has a significantly different average rating compared to the others.


```python
import scipy.stats as stats

# Step 1: Compute the observed F-statistic using one-way ANOVA
recipe_groups = [filtered_df[filtered_df['time_category'] == cat]['avg_rating'].dropna()
                 for cat in category_order]

observed_f_stat, _ = stats.f_oneway(*recipe_groups)
print("Observed F-statistic:", observed_f_stat)

# Step 2: Perform permutation test
n_permutations = 1000
perm_f_stats = []

for _ in range(n_permutations):
    shuffled_ratings = filtered_df['avg_rating'].sample(frac=1, replace=False).values  # Shuffle ratings
    shuffled_df = filtered_df.copy()
    shuffled_df['shuffled_rating'] = shuffled_ratings

    # Compute F-statistic for shuffled data
    perm_groups = [shuffled_df[shuffled_df['time_category'] == cat]['shuffled_rating'].dropna()
                   for cat in category_order]

    perm_f_stat, _ = stats.f_oneway(*perm_groups)
    perm_f_stats.append(perm_f_stat)

# Step 3: Compute p-value
perm_f_stats = np.array(perm_f_stats)
p_value = np.mean(np.array(perm_f_stats) >= observed_f_stat)

# Print results
print(f"Observed F-statistic: {observed_f_stat:.4f}")
print(f"P-value from permutation test: {p_value:.4f}")

# Step 4: Interpret results
alpha = 0.05  # Significance level

if p_value < alpha:
    print("We reject the null hypothesis: At least one recipe category has a significantly different average rating.")
else:
    print("We fail to reject the null hypothesis: There is no significant difference in ratings among the recipe categories.")

# # Create a histogram of the permutation F-statistics
# fig = px.histogram(
#     x=perm_f_stats, 
#     nbins=30, 
#     title="Permutation Distribution of F-statistics",
#     labels={"x": "F-statistic", "y": "Frequency"},
#     template="plotly_white"
# )

# # Overlay a vertical dashed line for the observed F-statistic with an annotation
# fig.add_vline(
#     x=observed_f_stat, 
#     line_dash="dash", 
#     line_color="red", 
#     annotation_text=f"Observed F = {observed_f_stat:.2f}", 
#     annotation_position="top right"
# )


# fig.update_layout(
#     xaxis_title="F-statistic",
#     yaxis_title="Frequency"
# )

# # fig.show()
```

    Observed F-statistic: 25.255264360614635
    Observed F-statistic: 25.2553
    P-value from permutation test: 0.0000
    We reject the null hypothesis: At least one recipe category has a significantly different average rating.


## Step 5: Framing a Prediction Problem

**Problem Statement**: 
Predict the average rating of recipes based on three features: number of steps, number of ingredients, and calories.
We want to predict the average rating of recipes based on information available at posting time. In our case, we use three quantitative features:
- n_steps: Number of cooking steps.
- n_ingredients: Number of ingredients.
- calories: Calorie content of the recipe.

**Type of Problem**: Since our response variable, avg_rating, is continuous (typically on a scale of 1–5), this is a **regression** problem.

**Information Known at Prediction Time**: At the time of prediction, we assume that only recipe-level features are available (cooking steps, ingredient count, and nutritional information like calories) before any user interactions or reviews are received. This ensures that our model uses only data that would be known when the recipe is posted.

**Evaluation Metric**: 
For our regression task, we use:
- Mean Absolute Error (MAE): For its interpretability (average absolute error in rating points).
- Root Mean Squared Error (RMSE): To penalize larger errors.
- We may also report R² (Coefficient of Determination) as a measure of explained variance.
These metrics provide an intuitive understanding of the model’s performance in predicting recipe ratings.



## Step 6: Baseline Model

We now build a baseline regression model using a scikit‑learn Pipeline. Our baseline model will include:
- Preprocessing: Standardize our three numerical features using a StandardScaler.
- Model: A simple RandomForestRegressor.


```python
# Use filtered_calories which has outliers removed
filtered_calories.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Index: 82111 entries, 0 to 234428
    Data columns (total 13 columns):
     #   Column         Non-Null Count  Dtype  
    ---  ------         --------------  -----  
     0   name           82110 non-null  object 
     1   id             82111 non-null  int64  
     2   minutes        82111 non-null  int64  
     3   tags           82111 non-null  object 
     4   nutrition      82111 non-null  object 
     5   n_steps        82111 non-null  int64  
     6   steps          82111 non-null  object 
     7   ingredients    82111 non-null  object 
     8   n_ingredients  82111 non-null  int64  
     9   user_id        82111 non-null  float64
     10  date           82111 non-null  object 
     11  avg_rating     82111 non-null  float64
     12  calories       82111 non-null  float64
    dtypes: float64(3), int64(4), object(6)
    memory usage: 8.8+ MB



```python
from sklearn.model_selection import train_test_split, GridSearchCV
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, QuantileTransformer, FunctionTransformer
from sklearn.ensemble import RandomForestRegressor
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score

# Step 1: Select the features and response variable.
# We use 'filtered_calories' which is a DataFrame that contains the three chosen features and 'avg_rating'.
feature_cols = ['n_steps', 'n_ingredients', 'calories']
X = filtered_calories[feature_cols]
y = filtered_calories['avg_rating']

# Step 2: Split the data into training and test sets (to evaluate generalization).
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Step 3: Create a Pipeline for preprocessing and modeling.
pipeline = Pipeline([
    ('preprocessor', ColumnTransformer(
        transformers=[
            ('num', StandardScaler(), feature_cols)
        ]
    )),
    ('regressor', RandomForestRegressor(n_estimators=100, random_state=42))
])

# Step 4: Train the baseline model.
pipeline.fit(X_train, y_train)

# Step 5: Make predictions on the test set.
y_pred = pipeline.predict(X_test)

# Step 6: Evaluate the model using MAE, RMSE, and R-squared
mae = mean_absolute_error(y_test, y_pred)
rmse = np.sqrt(mean_squared_error(y_test, y_pred))
r2 = r2_score(y_test, y_pred)

print("Baseline Model Performance:")
print(f"MAE: {mae:.4f}")
print(f"RMSE: {rmse:.4f}")
print(f"R-squared: {r2:.4f}")
```

    Baseline Model Performance:
    MAE: 0.4859
    RMSE: 0.6857
    R-squared: -0.1917


## Step 7: Final Model

* Feature Engineering:

    We add two new features:
    - cal_per_ing: Calories per ingredient, helps understand the calorie density of a recipe by normalizing calories by the number of ingredients.
    - steps_per_ing: Number of steps per ingredient, captures the complexity of the recipe relative to its ingredient count.
    These features are motivated by the idea that a recipe’s complexity (or simplicity) might be better captured by how many calories or steps are associated with each ingredient rather than the raw totals.
* Preprocessing with a ColumnTransformer:
    - We apply a StandardScaler on the features that are expected to be roughly on the same scale once engineered: n_steps, n_ingredients, cal_per_ing, and steps_per_ing.
    - We apply a QuantileTransformer on the calories column to reduce skewness and force a normal-like distribution (with the output distribution set to 'normal').
* Hyperparameter Tuning:

    We use GridSearchCV to tune key hyperparameters of the RandomForestRegressor:
    - n_estimators: Number of trees in the forest.
    - max_depth: Maximum depth of the tree.
    - min_samples_split: The minimum number of samples required to split an internal node.
    We choose these hyperparameters because they directly control model complexity and overfitting. The grid search uses 5‑fold cross‑validation on the training set, optimizing for Mean Absolute Error (MAE).
* Evaluation:

    The model is trained on the training set and evaluated on the test set (the same split as used in the baseline) using MAE, RMSE, and R². This allows us to compare performance improvements with our baseline.


```python
# --- Step 1: Feature Engineering ---
def add_new_features(X):
    X = X.copy()
    # Avoid division by zero by replacing zeros in n_ingredients with NaN
    X['cal_per_ing'] = X['calories'] / X['n_ingredients'].replace(0, np.nan)
    X['steps_per_ing'] = X['n_steps'] / X['n_ingredients'].replace(0, np.nan)
    # Fill potential NaN values (if n_ingredients was 0) with 0
    X['cal_per_ing'] = X['cal_per_ing'].fillna(0)
    X['steps_per_ing'] = X['steps_per_ing'].fillna(0)
    return X

# --- Step 2: Define the final pipeline ---
# We'll use the baseline features: 'n_steps', 'n_ingredients', 'calories'
# After feature engineering, the dataset will include two new features: 'cal_per_ing' and 'steps_per_ing'
base_features = ['n_steps', 'n_ingredients', 'calories']

final_pipeline = Pipeline([
    ('feature_engineering', FunctionTransformer(add_new_features, validate=False)),
    ('preprocessor', ColumnTransformer(
        transformers=[
            # Apply StandardScaler to these features (baseline + engineered)
            ('std', StandardScaler(), ['n_steps', 'n_ingredients', 'cal_per_ing', 'steps_per_ing']),
            # Apply QuantileTransformer to the 'calories' column
            ('quant', QuantileTransformer(output_distribution='normal'), ['calories'])
        ],
        remainder='passthrough'
    )),
    ('regressor', RandomForestRegressor(random_state=42))
])

# --- Step 3: Prepare the data ---
# Assume filtered_calories is a DataFrame containing 'n_steps', 'n_ingredients', 'calories', and 'avg_rating'
X_final = filtered_calories[base_features]  # our input features
y_final = filtered_calories['avg_rating']     # our response variable

# Split data for training and testing
X_train, X_test, y_train, y_test = train_test_split(X_final, y_final, test_size=0.2, random_state=42)

# --- Step 4: Hyperparameter Tuning using GridSearchCV ---
# Define a grid for key hyperparameters
param_grid = {
    'regressor__n_estimators': [100, 200],
    'regressor__max_depth': [None, 5, 10, 20],
    'regressor__min_samples_split': [2, 5, 10]
}

grid_search = GridSearchCV(final_pipeline, param_grid, cv=5, scoring='neg_mean_absolute_error', n_jobs=-1)
grid_search.fit(X_train, y_train)

print("Best hyperparameters:", grid_search.best_params_)

# --- Step 5: Evaluate the Final Model ---
y_pred_final = grid_search.predict(X_test)
mae_final = mean_absolute_error(y_test, y_pred_final)
rmse_final = np.sqrt(mean_squared_error(y_test, y_pred_final))
r2_final = r2_score(y_test, y_pred_final)

print("Final Model Performance:")
print(f"MAE: {mae_final:.4f}")
print(f"RMSE: {rmse_final:.4f}")
print(f"R-squared: {r2_final:.4f}")
```


    ---------------------------------------------------------------------------

    KeyboardInterrupt                         Traceback (most recent call last)

    Cell In[33], line 48
         41 param_grid = {
         42     'regressor__n_estimators': [100, 200],
         43     'regressor__max_depth': [None, 5, 10, 20],
         44     'regressor__min_samples_split': [2, 5, 10]
         45 }
         47 grid_search = GridSearchCV(final_pipeline, param_grid, cv=5, scoring='neg_mean_absolute_error', n_jobs=-1)
    ---> 48 grid_search.fit(X_train, y_train)
         50 print("Best hyperparameters:", grid_search.best_params_)
         52 # --- Step 5: Evaluate the Final Model ---


    File ~/miniforge3/envs/dsc80/lib/python3.12/site-packages/sklearn/base.py:1473, in _fit_context.<locals>.decorator.<locals>.wrapper(estimator, *args, **kwargs)
       1466     estimator._validate_params()
       1468 with config_context(
       1469     skip_parameter_validation=(
       1470         prefer_skip_nested_validation or global_skip_validation
       1471     )
       1472 ):
    -> 1473     return fit_method(estimator, *args, **kwargs)


    File ~/miniforge3/envs/dsc80/lib/python3.12/site-packages/sklearn/model_selection/_search.py:1019, in BaseSearchCV.fit(self, X, y, **params)
       1013     results = self._format_results(
       1014         all_candidate_params, n_splits, all_out, all_more_results
       1015     )
       1017     return results
    -> 1019 self._run_search(evaluate_candidates)
       1021 # multimetric is determined here because in the case of a callable
       1022 # self.scoring the return type is only known after calling
       1023 first_test_score = all_out[0]["test_scores"]


    File ~/miniforge3/envs/dsc80/lib/python3.12/site-packages/sklearn/model_selection/_search.py:1573, in GridSearchCV._run_search(self, evaluate_candidates)
       1571 def _run_search(self, evaluate_candidates):
       1572     """Search all candidates in param_grid"""
    -> 1573     evaluate_candidates(ParameterGrid(self.param_grid))


    File ~/miniforge3/envs/dsc80/lib/python3.12/site-packages/sklearn/model_selection/_search.py:965, in BaseSearchCV.fit.<locals>.evaluate_candidates(candidate_params, cv, more_results)
        957 if self.verbose > 0:
        958     print(
        959         "Fitting {0} folds for each of {1} candidates,"
        960         " totalling {2} fits".format(
        961             n_splits, n_candidates, n_candidates * n_splits
        962         )
        963     )
    --> 965 out = parallel(
        966     delayed(_fit_and_score)(
        967         clone(base_estimator),
        968         X,
        969         y,
        970         train=train,
        971         test=test,
        972         parameters=parameters,
        973         split_progress=(split_idx, n_splits),
        974         candidate_progress=(cand_idx, n_candidates),
        975         **fit_and_score_kwargs,
        976     )
        977     for (cand_idx, parameters), (split_idx, (train, test)) in product(
        978         enumerate(candidate_params),
        979         enumerate(cv.split(X, y, **routed_params.splitter.split)),
        980     )
        981 )
        983 if len(out) < 1:
        984     raise ValueError(
        985         "No fits were performed. "
        986         "Was the CV iterator empty? "
        987         "Were there no candidates?"
        988     )


    File ~/miniforge3/envs/dsc80/lib/python3.12/site-packages/sklearn/utils/parallel.py:74, in Parallel.__call__(self, iterable)
         69 config = get_config()
         70 iterable_with_config = (
         71     (_with_config(delayed_func, config), args, kwargs)
         72     for delayed_func, args, kwargs in iterable
         73 )
    ---> 74 return super().__call__(iterable_with_config)


    File ~/miniforge3/envs/dsc80/lib/python3.12/site-packages/joblib/parallel.py:2007, in Parallel.__call__(self, iterable)
       2001 # The first item from the output is blank, but it makes the interpreter
       2002 # progress until it enters the Try/Except block of the generator and
       2003 # reaches the first `yield` statement. This starts the asynchronous
       2004 # dispatch of the tasks to the workers.
       2005 next(output)
    -> 2007 return output if self.return_generator else list(output)


    File ~/miniforge3/envs/dsc80/lib/python3.12/site-packages/joblib/parallel.py:1650, in Parallel._get_outputs(self, iterator, pre_dispatch)
       1647     yield
       1649     with self._backend.retrieval_context():
    -> 1650         yield from self._retrieve()
       1652 except GeneratorExit:
       1653     # The generator has been garbage collected before being fully
       1654     # consumed. This aborts the remaining tasks if possible and warn
       1655     # the user if necessary.
       1656     self._exception = True


    File ~/miniforge3/envs/dsc80/lib/python3.12/site-packages/joblib/parallel.py:1762, in Parallel._retrieve(self)
       1757 # If the next job is not ready for retrieval yet, we just wait for
       1758 # async callbacks to progress.
       1759 if ((len(self._jobs) == 0) or
       1760     (self._jobs[0].get_status(
       1761         timeout=self.timeout) == TASK_PENDING)):
    -> 1762     time.sleep(0.01)
       1763     continue
       1765 # We need to be careful: the job list can be filling up as
       1766 # we empty it and Python list are not thread-safe by
       1767 # default hence the use of the lock


    KeyboardInterrupt: 



```python

```

## Step 8: Fairness Analysis

**Question**: 
Does our model perform worse (i.e. have higher RMSE) for recipes that are “simple” (with fewer cooking steps) than for recipes that are “complex” (with more cooking steps)?


```python

```
