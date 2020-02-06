# Kaggle - Data Science Bowl 2019

<p align="center">
  <img width="50%" height="50%" src="https://datasciencebowl.com/wp-content/uploads/2019/03/dsb-logo-400.png">
</p>

This is the first global competition that I participated with one of my friend. The optimal goal of joining this competition is to learn as much as possible from the Kaggle masters as well as other top-tier data scientists. Big kudos to [@Thinh Truong](https://github.com/joey234) for bringing up a lot of novel solutions and informative discussions with me.

The result is more than expected as we learnt a lot from different solutions and discussions, from so many participants. Kaggle has its best way to motivate people to share their knowledge and engage discussions. The competition has 3497 teams, therefore, with only a small change of score can upgrade or downgrade one team's position significantly.

We ended up being at top 34% (1155/3497) on private leaderboard, rocketed 524 positions compared to the public leaderboard. Although this is not a quite good position, the knowledge we received from other experts are more valuable than enough. 

Keys that we got out of this competition:
- The dataset is significantly hard to understand in a short time. The target, features and the data itself is complicated enough to challenge all the kagglers. On the other hand, this engaged a huge amount of discussions for everyone to learn from others. I personally cannot understand this problem completely (though i tried for days) without many discussions with my friend.  
- The target variable is an ordinal variable (a score with possible values of 0,1,2,3), therefore, the baseline model is a classifier. However, as discussed and experimented by other participants, the regression model can be used and yields a significantly better result.
- In order to convert the regression outputs back to ordinal variables, a postprocessing step is necessary. This makes not only the preprocessing, feature engineering, modeling but also the postprocessing step very important to have an optimal solution.
- The validation strategy, feature engineering are the keys of winning in this competition. For many solutions, the participant generates their own secret features based on the experience of playing the game, these features turned out to be most effective ones, and helped the models generalize well towards the target.
- Feature selection to select top k best features based on [null importances](https://www.kaggle.com/ogrellier/feature-selection-with-null-importances) is widely used among top solutions.
- The public test set contains only 1000 records, this turned out to be a huge challenge for all teams to not overfit the public test set. Another controversial topic is the assumption whether or not the training set has the same distribution with the public test set and more importantly, the private test set. This problem also leads me to know another concept, adversarial validation. As a result, many teams chose to trust the local CV score instead of public LB score, and climbed up from 500 to 1000 ranks in the private leaderboard.
- Ensembling SOTA models is still the optimal approach to get good results for competitions. Additionally, boosting models are still the most preferrable in tabular data competitions, some also implements neural networks as they may give a different perspective about the target, hence, increase the result of ensembled models.
- An elegant way of implementing models using OOP concepts makes the code amazingly readable. Kudos to the author of [this solution](https://www.kaggle.com/braquino/convert-to-regression) for sharing his codes.

## About the Competition
This is the 5th annual Data Science Bowl, presented by Booz Allen Hamilton and Kaggle.

The competition's topic of 2019 is surrounding PBS KIDS app, a trusted application that help children learn important skills in school and life. In this challenge, the competitors are given the history data of gameplays of individual kids, collected from the PBS KIDS Measure Up! mobile application. The competitors will be asked to predict the scores on in-game assessments. Furthermore, the solutions are expected to support discovering important relationships between different types of high-quality educational media and learning process.

## About the PBS KIDS Measure Up! App
<p align="center">
  <img width="100%" height="100%" src="https://i.ytimg.com/vi/1ejHigxuR2Q/maxresdefault.jpg">
</p>

From the description: "In the PBS KIDS Measure Up! app, children ages 3 to 5 learn early STEM concepts focused on length, width, capacity, and weight while going on an adventure through Treetop City, Magma Peak, and Crystal Caves. Joined by their favorite PBS KIDS characters, children can also collect rewards and unlock digital toys as they play." To know more how this app works, this is the [web version](https://measureup.pbskids.org) of it.

## About the Dataset
You need to agree with the policies of the competition to obtain this dataset on your own, so i do not include the data within this repo. You can join and download it [here](https://www.kaggle.com/c/data-science-bowl-2019/overview).


## Our Approach
Overall, we obtained quadratic weighted kappa score of 0.532 - Public LB and 0.525 - Private LB.
#### Validation strategy
Grouped-KFold with k=5, grouped by the `installation_id`, which is the id of each device that installed the app.

#### Feature Engineering
Group records by `installation_id`, then sum up the features and represented them as accumulated features, this way of feature engineering can model the historic data of one user.
For categorical features, they can be handled effectively by boosting models themselves.
For numerical features, they can be aggregated by using mean, standard deviation or median.
Based on the baseline, we also did:
- Convert `world` to numerical form
- Add corresponding world of assessment: `assessment_world_code`
- Accumulated number of different previous assessments:
`assessment_1`, `assessment_4`, `assessment_16`, `assessment_18`, `assessment_39`,
- Add `duration_std` along with `duration_mean`
- Add `accumulated_assessment_attemps`
- Add combination of `session_title_code` of an assessment with corresponding `accuracy_group`
assessment_accuracy_map
- Add `clip`/`activity`/`game` duration mean and std
- No need to feature scaling as tree-based models are not affected by different scales of features.

#### Feature Selection
Eliminate high correlated features (>0.99 correlated coefficient)

#### Modelling
Ensemble regression models of LightGBM, XgBoost, CatBoost, and Neural network, with 0.25 weighted each.

#### Postprocessing
The method of post-processing is implemented by the author of the baseline solution.
Assumption: Training and testing set have the same distribution.

From the label of the training set, we calculate the pmf. The way of converting continuous output back to discrete is based on calculating which percentile rank (which bound) that the predition belongs to. Based on the assumption that training and testing set have the same distribution, we can use the pmf of training set to calculate the percentile rank of the testing set.

After obtaining the percentile rank of testing set, we can calculate which percentile rank that the predicted value belongs to.