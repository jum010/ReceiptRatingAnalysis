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

    Users don't review certain recipes, usually because none have tried it or they don't have a strong opinion about it and didn't submit any interaction(neither reviews nor rates). Since this is caused by an unobserved value, we conclude this missingness is NMAR and drop this recipe from our dataset.

- Investigating the rest of missingness

    We want to find out whether users choose not to rate a recipe if they had an extremely negative or extremely positive experience. To see the rating distribution of these reviews, we select a random sample of reviews with missing ratings. Then, we give these chosen reviews a properly predicted rating based on our knowledge.

    Based on the description of each review, I gave each one a reasonable rating and put them in an array called 'rating_sample_reviews'.

    We draw the distribution of these predicted ratings and see if there is any significant bumps at both edges of the plot.

![Alt text](<images/Distribution of Imputed Ratings.svg>)


The plot shows that the imputed ratings are more likely a normal distribution, and it suggests that the missing ratings tend to cluster around an average value rather than being skewed toward very high or very low ratings. In other words, it indicates that—based on the review text—the missing ratings do not appear to be systematically biased, which means that these users chose not to review a recipe not because they had an extremely negative or extremely positive experience.

This evidence is consistent with the missingness being either Missing Completely at Random (MCAR) or Missing at Random (MAR), rather than NMAR (Not Missing At Random). 

So, a normal (bell-shaped) distribution of imputed ratings supports the idea that the missingness in ratings is not being driven by the underlying sentiment or the value of the rating itself(not NMAR).

#### Missingness Dependency of ratings: 
In this section, we investigate whether the missingness in the rating column is systematically related to certain recipe attributes. Specifically, we test the dependency of the missing ratings on two numeric columns: minutes (cooking time) and n_ingredients (number of ingredients). For each test, we use a permutation test with the following setup:
- Test Statistic: 
As our test statistic, we use the absolute difference in the mean of the numeric column between recipes with missing ratings and those with non-missing ratings.

- Significance Level: We set our significance level at 0.05.

**Permutation Test Implementation**


Dependency on Cooking Time (minutes)

- **Null Hypothesis (H₀)**:
The rating's missingness does not depend on the recipe’s cooking time. In other words, the mean cooking time is the same for recipes with missing and non-missing ratings.

- **Alternate Hypothesis (H₁)**:
The missingness in rating does depend on the recipe’s cooking time; that is, the mean cooking time differs between recipes with missing and non-missing ratings.

- **Results**: Dependency test for 'minutes': Observed difference = 51.46, p-value = 0.1020

- **Conclusion**: The observed difference in mean cooking time between recipes with missing ratings and those with non-missing ratings is 51.46 minutes. However, the permutation test yields a p-value of 0.1020, which is greater than the significance level of 0.05. Therefore, we **fail to reject** the null hypothesis. This suggests that the missingness in rating does not significantly depend on the recipe’s cooking time.

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

- **Conclusion**: In this test, the observed difference in the mean number of ingredients between recipes with missing and non-missing ratings is 0.16. The permutation test produces a p-value of 0.0000, which is below our significance level of 0.05. Thus, we **reject** the null hypothesis. This result indicates that the missingness in rating is significantly associated with the number of ingredients: recipes with missing ratings tend to have a different number of ingredients compared to those with complete ratings.

<iframe
    src="images/Permutation Distribution of Mean Differences for 'n_ingredients'.html"
    width="800"
    height="600"
    frameborder="0"
></iframe>

**Summary of Findings**:

These results suggest that the mechanism for missing ratings is likely MAR (Missing At Random) rather than MCAR (Missing Completely At Random), therefore, instead of directly dropping the missing roles, we suggested to imputate those missing rates.


**Missing rating imputation Strategy**

The dataset utilized in this study has missing ratings for some of the recipes. As user ratings are one of the main indicators of recipe quality, these missing values need to be addressed in a manner that properly represents the corresponding information regarding each recipe. To do this, we employed a hybrid imputation approach that utilizes global and recipe-level information.

- Global and Recipe-Level Statistics

    First, we computed the global mean and standard deviation of the ratings across all recipes with available ratings. We then grouped the data by recipe ID to calculate recipe-specific statistics – namely, the mean rating, standard deviation, and the count of ratings for each recipe. For recipes that had no observed ratings, we used the global mean as a fallback. Additionally, any missing recipe-level statistics were replaced with appropriate global values (e.g., a missing mean was set to the global mean, and a missing standard deviation was set to 0).

- Hybrid Imputation

    Then, we created a hybrid imputation function to address the issue of missing ratings on a recipe-by-recipe basis:

    - Adequate Data (≥5 evaluations):
    
        For recipes that have at least five observed ratings, we fill in missing values with the recipe's average rating. The assumption here is that a recipe with many ratings has a good estimate of its average quality.

    - Sparse Data (<5 ratings):

        For recipes with fewer than five ratings, we adopt a probabilistic approach. If there is variation in the available ratings (i.e., a non-zero standard deviation), we sample a rating from a normal distribution centered on the recipe's mean with the corresponding standard deviation. If no variation is present, we simply use the mean. This method helps capture the uncertainty inherent in having limited data for a recipe.

    - Clipping:
    
        Finally, the imputed ratings are clipped to ensure they fall within the acceptable range (which should be 1 to 5), maintaining consistency with the original rating scale.

By combining these techniques, our imputation method preserves the integrity of the original data distribution and minimizes bias in subsequent analyses. The resulting imputed ratings (stored as filled_rating in our dataset) are then used for further tasks such as predictive modeling and hypothesis testing.

#### Add column 'average_rating' containing average rating per recipe.
Using our filled_ratings, we computed the mean rating for each recipe and added it as a new column, "average_rating", to our dataset.

#### Create a new 'calories' column
Since we are also curious about the relationship between ratings and calories, we will take the first element in the 'nutrition' column, which represents the amount of calories, and store it in a new column, 'calories'. 

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
    </tr>A
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
- Explore the distribution of the average rating (avg_rating)

    In this analysis, we examined the distribution of average recipe ratings across our dataset. As shown in the histogram below, the ratings are highly concentrated in the upper range (around 4.5 to 5), indicating that the majority of recipes receive favorable evaluations from users. This trend suggests that, overall, recipes on our platform are well-regarded.

<iframe
    src="images/Distribution of Average Recipe Ratings.html"
    width="600"
    height="400"
    frameborder="0"
></iframe>

- Explore the distribution of the recipe preparation time (minutes)

    We also explored the distribution of recipe preparation times. The initial histogram reveals a right-skewed distribution with a long tail, indicating that while many recipes can be prepared relatively quickly, there are a few outliers with extremely long preparation times. A violin plot further confirms the presence of these outliers. After filtering the data to remove extreme values (using the 1st and 99th percentiles), the resulting histogram provides a clearer view of the typical cooking times, showing that most recipes fall within a more reasonable range.

<iframe
    src="images/Distribution of Recipe Preparation Time.html"
    width="600"
    height="400"
    frameborder="0"
></iframe>

<iframe
    src="images/Distribution of Recipe Preparation Time violin plot.html"
    width="600"
    height="400"
    frameborder="0"
></iframe>

<iframe
    src="images/Distribution of Recipe Preparation Time without outliers.html"
    width="600"
    height="400"
    frameborder="0"
></iframe>


### Bivariate Analysis
To better understand how recipe quality relates to various attributes, we explored the relationship between the average recipe rating and several key variables and computed each group's Pearson correlation coefficient. These could help us better understand our data.

- Explore the relationship between the average rating of recipes and the cooking time (minutes).

    The scatter plot below shows the relationship between average rating and cooking time without outliers. It suggests that—after filtering out extreme cooking times—the ratings tend to remain consistently high regardless of how long the recipe takes. The Pearson correlation coefficient between cooking time and average rating without outliers is around -0.03, which indicates that there is virtually no linear relationship between cooking time (after removing outliers) and average recipe ratings. In other words, variations in cooking time are not associated with systematic changes in how users rate the recipes. This suggests that other factors may be more influential in determining recipe ratings.

<iframe
    src="images/Average Rating vs Cooking Time without outliers.html"
    width="600"
    height="400"
    frameborder="0"
></iframe>

- Explore the relationship between the average rating of recipes and the cooking steps (n_steps).

    The scatter plot below illustrates the relationship between average recipe rating and the number of cooking steps. It shows that the ratings remain relatively stable across different numbers of steps, indicating that recipe complexity (as measured by the number of steps) does not have a strong influence on how users rate the recipes. In addition, the Pearson correlation coefficient between the number of cooking steps and the average rating is 0.005, which is close to 0, which confirms that there is virtually no linear relationship between these two variables.

<iframe
    src="images/Average Rating vs Number of Cooking Steps.html"
    width="600"
    height="400"
    frameborder="0"
></iframe>

- Explore the relationship between recipes' average rating (avg_rating) and the number of ingredients (n_ingredients).

    The scatter plot below illustrates the relationship between the average rating and the number of ingredients in a recipe. The Pearson correlation coefficient is approximately -0.004, which indicates that there is virtually no linear relationship between the number of ingredients and the average rating. In other words, the number of ingredients does not systematically affect how users rate recipes, suggesting that other factors may be more influential in determining recipe quality.

<iframe
    src="images/Average Rating vs Number of Ingredients.html"
    width="600"
    height="400"
    frameborder="0"
></iframe>

- Explore the relationship between recipes' average rating (avg_rating) and the calories (first element from the 'nutrition' column).

    The scatter plot below illustrates the relationship between the average recipe rating and calories (after removing outliers). The Pearson correlation coefficient is approximately -0.0034, which indicates that there is virtually no linear relationship between the calorie content and the average rating. In other words, the calorie levels of recipes do not systematically influence how users rate them, implying that other factors are likely more important in determining recipe quality.

<iframe
    src="images/Average Rating vs Calories without outliers.html"
    width="600"
    height="400"
    frameborder="0"
></iframe>

### Interesting Aggregates

Below is a table summarizing the average recipe ratings by cooking time categories. We first categorized recipes into four groups based on their preparation time:

- Quick (<30 min)
- Moderate (30-60 min)
- Long (60-120 min)
- Extra Long (>120 min)

The table below shows the mean rating, standard deviation, and the number of recipes in each category:

| time_category           | mean | std  | count |
|-------------------------|------|------|-------|
| Extra Long (>120 min)   | 4.59 | 0.67 | 8428  |
| Long (60-120 min)       | 4.62 | 0.64 | 15570 |
| Moderate (30-60 min)    | 4.61 | 0.64 | 28665 |
| Quick (<30 min)         | 4.65 | 0.61 | 30055 |

This aggregate table is significant because it reveals that recipes tend to be highly rated regardless of their preparation time. Although the “Quick” recipes have a slightly higher mean rating compared to “Extra Long” recipes, the differences are very subtle. Additionally, the variation (as measured by the standard deviation) is similar across groups. These insights suggest that while cooking time might have a small effect on user ratings, other factors are likely more influential in determining recipe quality. 

Below is an embedded box plot that further visualizes the distribution of ratings across these time categories:

<iframe
    src="images/Comparison of Ratings by Time Category.html"
    width="600"
    height="400"
    frameborder="0"
></iframe>


## Assessment of Missingness
When we look at our original dataset, merged_df, to see if there are any missing values in each column that could affect the bias in our analysis, we find that four columns total contain missing values: 'name', 'description', 'rating', and 'review'.

According to previous analysis, missingness in “rating” is mostly MAR and was handled by imputation. Now, we are going to look at the other three. 

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

**Missingness of 'review'** In previous analyses, when analyzing the missingness of 'rating', we found that the absence of user feedback does not cause missing ratings—meaning, missing 'rating' values do not occur simultaneously with missing 'review' values. This suggests that the user may choose not to provide a review even when a rating is given. Therefore, the missing values in 'review' are NMAR (Not Missing At Random) because the review was omitted despite the rating being provided.

In conclusion, the missingness in 'name', 'description', and 'review' is NMAR because these values are absent from the original data source or omitted by the user. Since these columns (name, description, review) are not central to our main analysis, dropping them won't have any effect or cause any bias. We can safely use our cleaned_df, which has the 'description' and 'review' columns dropped, for the hypothesis test.

## Hypothesis Testing

**Research Question**: Is there a significant difference in ratings between quick (<30 min), moderate (30-60 min), long (60-120 min), and extra long (>120 min) recipes?

**Null Hypothesis (H0​)**: There is no significant difference in average ratings between Quick (<30 min), Moderate (30-60 min), Long (60-120 min), and Extra Long (>120 min) recipes.

**Alternative Hypothesis (H1​)**: At least one recipe category has a significantly different average rating compared to the others.

**Test Statistic and Significance Level**: 
We used the F-statistic from a one-way ANOVA as our test statistic. The F-statistic measures the variance ratio between group means to variance within the groups. We set our significance level (α) at 0.05.

**Interpret results**
- Observed F-statistic: 25.255264360614635
- Observed F-statistic: 25.2553
- P-value from permutation test: 0.0000
- We **reject** the null hypothesis: At least one recipe category has a significantly different average rating.

Since the p-value is less than 0.05, we reject the null hypothesis. This result suggests that there is statistically significant evidence that the average ratings differ among at least one of the cooking time categories. While the test provides strong evidence of a difference, it does not definitively prove causation but indicates that cooking time may be associated with how recipes are rated.

The interactive histogram below displays the permutation distribution of F-statistics, with the observed F-statistic marked by a red dashed line.

<iframe
    src="images/Permutation Distribution of F-statistics.html"
    width="800"
    height="600"
    frameborder="0"
></iframe>


## Framing a Prediction Problem
- Prediction Problem and Type

    The objective of our prediction task is to predict the average rating of a recipe based on recipe characteristics available at the time of posting. Since the response variable, average rating (avg_rating), is a continuous numerical value (typically on a scale of 1 to 5), this is a regression problem rather than a classification problem.

- Response Variable and Justification

    The response variable for this model is the average rating (avg_rating), which represents the average user rating assigned to a recipe. This variable was chosen because it directly reflects user preferences and provides a quantitative measure of recipe success. By predicting avg_rating, we aim to understand how certain recipe attributes influence user satisfaction.

- Features Available at Prediction Time

    Only recipe-level attributes available at the time of posting can be used at the time of prediction. This ensures that our model reflects a real-world scenario where recipe ratings must be predicted before any user interactions occur. The features used in our model are:
    - n_steps: Number of steps required to prepare the recipe.
    - n_ingredients: Number of ingredients used in the recipe.
    - calories: Estimated calorie content of the recipe.

    These three features were selected based on insights from our Bivariate Analysis, where they exhibited some level of correlation with the average rating. They are quantitative, consistently available, and interpretable, making them suitable for building a robust predictive model.

- Why Other Features Were Not Included

    While other factors, such as cuisine type, cooking method, or user-submitted metadata, could potentially impact recipe ratings, they were excluded for the following reasons:
    Availability at Prediction Time: Some features (e.g., user reviews, number of ratings) are only generated after a recipe is posted, making them unsuitable for a predictive model that must function at the time of posting.

    Complexity & Encoding Requirements: Categorical features like cuisine type would require encoding, which adds complexity that may not be necessary for our initial regression model. Future work could explore their impact by incorporating them as additional predictors.

- Evaluation Metric and Justification

    To assess the accuracy of our model, we use three key evaluation metrics:
    - Mean Absolute Error (MAE): Measures the average absolute difference between predicted and actual ratings. MAE is intuitive and easy to interpret, directly measuring how close predictions are to actual ratings.
    Root Mean Squared Error (RMSE): This is similar to MAE but penalizes larger errors more heavily, making it useful for minimizing extreme mispredictions.
    - R² (Coefficient of Determination): Evaluates how well the model explains the variance in recipe ratings, providing a general measure of model performance.

    We chose RMSE over MAE in particular because it emphasizes significant errors, which is helpful in ensuring that high deviations in predicted ratings are minimized. However, MAE is also reported for better interpretability. Accuracy and F1-score were not chosen as they are only applicable to classification problems, whereas our task is regression.


## Baseline Model
- Model Description

    For our baseline regression model, we use a Random Forest Regression model implemented within a scikit-learn Pipeline. Random Forest is a robust, nonparametric model that effectively captures nonlinear relationships between features and ratings. While simpler models like linear regression could have been considered, Random Forest provides a more reliable starting point due to its ability to handle feature interactions and nonlinearity.

- Features in the Model

    Our model includes three quantitative features mentioned above. Since our dataset consists only of numerical features, no ordinal or nominal features are included. As a result, no categorical encoding was required.

- Preprocessing Approach

    Although Random Forest does not require feature scaling, we applied StandardScaler() to normalize n_steps and n_ingredients. This transformation ensures consistency across different models if we compare performance with algorithms relying on distance-based calculations (e.g., linear regression or k-nearest neighbors). The calories feature was used after filtering out outliers to improve model stability.

- Model Performance
    After training and evaluating the baseline model on a train-test split (80%-20%), we obtained the following performance metrics:
    - Mean Absolute Error (MAE): 0.4859
    - Root Mean Squared Error (RMSE): 0.6857
    - R-squared (R²): -0.1917

- Assessment of Model Quality

    The negative R² score (-0.1917) indicates that the model performs worse than simply predicting the mean rating for all observations. This suggests that the features selected—n_steps, n_ingredients, and calories—may not be strong predictors of avg_rating on their own.

- Why the Model May Not Be “Good”

    The baseline model demonstrates low predictive power, as seen in its high RMSE (0.6857) and negative R² (-0.1917), meaning it performs worse than simply predicting the mean rating. This suggests that the chosen features—n_steps, n_ingredients, and calories—do not strongly correlate with recipe ratings, making it difficult for the model to capture meaningful patterns.

    A significant limitation is feature selection since only three numerical features are used, excluding key categorical attributes such as cuisine type or cooking method. These missing factors likely play a significant role in user preferences and could improve predictive accuracy. Additionally, data noise affects performance, as ratings are subjective and influenced by personal tastes, cultural factors, and expectations—elements that cannot be fully captured through numerical features alone.

- Next Steps for Improvement

    To improve performance, feature engineering should be considered by adding categorical variables like cuisine type or meal category using encoding techniques. Trying alternative models such as Gradient Boosting Machines (GBM) or Neural Networks may yield better results, as they can capture complex relationships more effectively. Lastly, hyperparameter tuning, optimizing parameters like the number of estimators and tree depth in Random Forest, could enhance model generalization. By refining feature selection and model parameters, we can work toward building a more accurate predictive model.


## Final Model
- Feature Engineering and Justification
    To enhance our model's predictive ability, we introduced two new features:
    -	cal_per_ing (Calories per ingredient): This feature normalizes calorie content relative to the number of ingredients, helping to assess a recipe's calorie density. We hypothesize that healthier recipes (fewer calories per ingredient) may receive higher ratings from users who prioritize nutrition.
    -	steps_per_ing (Steps per ingredient): This feature captures the complexity of a recipe relative to its ingredients. A recipe with many steps but few ingredients may indicate intricate techniques, while a recipe with many ingredients but few steps might be more straightforward to prepare. We hypothesized that higher complexity could lead to lower ratings if users find the recipe challenging to follow.

    These features are valuable additions because they introduce a refined understanding of recipe difficulty and nutritional content. Unlike raw calorie count or step count, these new variables provide contextual information about a recipe’s structure. This added context can help the model differentiate between similar recipes that may have different levels of complexity and healthiness, leading to better predictions.

- Modeling Algorithm and Hyperparameter Selection

    We used a Random Forest Regressionor, which is well-suited for handling non-linear relationships and interactions between features. Since Random Forests automatically capture complex patterns without requiring explicit feature transformations, it was a logical choice for improving upon our baseline model.

    To optimize the model, we conducted hyperparameter tuning using GridSearchCV with 5-fold cross-validation. We focused on tuning the following parameters:

    - n_estimators: Number of trees in the forest
    - max_depth: Maximum depth of each decision tree
    - min_samples_split: Minimum number of samples required to split a node

    After running the grid search, the best-performing hyperparameters were:

    - n_estimators = 100
    - max_depth = 5
    - min_samples_split = 2

    These values suggest that a relatively shallow Random Forest (max depth of 5) with a moderate number of estimators (100 trees) provides the best trade-off between bias and variance, preventing overfitting while capturing useful patterns.

- Performance Improvement Over the Baseline Model

    The Final Model outperformed the Baseline Model in all evaluation metrics by introducing feature engineering and hyperparameter tuning.

    | Model                                        | MAE   | RMSE  | R²      |
    |----------------------------------------------|-------|-------|---------|
    | Baseline (Random Forest, no feature engineering) | 0.4859 | 0.6857 | -0.1917 |
    | Final Model (Random Forest with new features + tuning) | 0.4505 | 0.6277 | 0.0014  |

    The Mean Absolute Error (MAE) decreased from 0.4859 to 0.4505, and the RMSE improved from 0.6857 to 0.6277, indicating that our predictions are now closer to actual user ratings. The most significant change is in R², which improved from -0.1917 to 0.0014. It suggests that the model is at least explaining some variance in the data, unlike the baseline, which performed worse than predicting the mean rating.

- Comparison with Linear Regression
    We compared our final Random Forest model with a Linear Regression baseline to assess whether the improvements were due to feature engineering or model complexity. The results showed that the Random Forest model performed slightly better:

    | Model                     | MAE   | RMSE  | R²     |
    |---------------------------|-------|-------|--------|
    | Linear Regression         | 0.4512 | 0.6281 | 0.0001 |
    | Final Random Forest Model | 0.4505 | 0.6277 | 0.0014  |

    When we compare the data, we can conclude that our feature-engineered Random Forest model performed better, which means our new features contributed to improved predictions. The higher R² score indicates that non-linear interactions between features may be useful for predicting recipe ratings.


## Fairness Analysis
To assess whether our model exhibits fairness, we compare its performance for simple recipes (fewer cooking steps) and complex recipes (more cooking steps). We define these groups as follows:
Group X (Simple Recipes): Recipes with a number of steps (n_steps) less than or equal to the median value.
Group Y (Complex Recipes): Recipes with n_steps greater than the median value.
By using the median n_steps as a threshold, we ensure that both groups contain a balanced number of recipes, preventing an imbalanced comparison.

- Evaluation Metric

    We measure model performance using Root Mean Squared Error (RMSE) separately for both groups:
    RMSE for Simple Recipes: Measures how well the model predicts ratings for recipes with fewer cooking steps.
    RMSE for Complex Recipes: Measures model performance for recipes with more cooking steps.
    If the RMSE differs significantly between the two groups, it could indicate that the model systematically performs worse for one group, raising fairness concerns.

    The results are:

    - RMSE for Simple Recipes: 0.6155
    - RMSE for Complex Recipes: 0.6426

    At first glance, the model appears to have a slightly higher RMSE for complex recipes, suggesting it may struggle to predict ratings for these recipes. However, to be more precise, we conduct a statistical significance test to determine whether this difference is meaningful or due to random variability.

- Hypothesis Testing

    To formally evaluate fairness, we set up a permutation test with the following hypotheses:

    - Null Hypothesis (H₀): The model is fair, meaning any observed RMSE difference between simple and complex recipes is due to random variation.
    - Alternative Hypothesis (H₁): The model is unfair, meaning it systematically makes more significant errors (higher RMSE) for one group than the other.
    - test statistic： We use the observed RMSE difference between the two groups and get the observed difference is − 0.0271. This negative value indicates that the model performs slightly worse for complex recipes.

    We perform a permutation test with 1000 random shuffles of the ratings, recalculating the RMSE difference each time to create a distribution of RMSE differences under the null hypothesis. The significance level (α) is set to 0.05.

- Results and Conclusion

    The p-value obtained from the permutation test is 0.0980, which is greater than 0.05. Since the p-value is not small enough to reject the null hypothesis, we **fail to reject the null** and conclude that the observed RMSE difference is not statistically significant.

    Our results suggest that while the model’s RMSE is slightly higher for complex recipes, this difference is likely due to chance rather than a systematic bias in prediction performance. Thus, we do not find strong evidence that our model is unfair toward simple or complex recipes.

