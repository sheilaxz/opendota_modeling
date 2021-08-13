

# OpenDota API Data Analysis and Modeling  
### Udacity Data Scientist Nanodegree Capstone Project

<img src="https://github.com/sheilaxz/opendota_modeling/blob/main/The_International_2019.png?raw=true" width="840" height="560">  


## Table of Contents

1. [Introduction](#intro)  
2. [Tasks](#tasks)
3. [Data Extraction](#dataextraction)  
4. [Analysis and Modeling](#modeling)
5. [File Description](#files)  
6. [Licensing, Authors, and Acknowledgements](#licensing)


## 1. Introduction  <a name="intro"></a>

Being one of the most famous RTS-styled (RTS: real-time strategy) MOBA-type (MOBA: multi-player online battle arena) competitive team game, DotA has been well-loved by its players all over the world for almost 20 years. Every day, millions of players worldwide enter the battle as one of over a hundred Dota Heroes in a 5v5 team clash (the Team Radiant and the Team Dire). Each player controls a single Hero in a game, a strategically powerful unit with unique abilities and characteristics that can be strengthened as the game continues.  

Having been a DotA fan myself, since I knew that there were a R package "RDota2" and a python package "dota2," and I could access Dota game data via API, I have always wanted to play with the API data. This project provides a great opportunity for me to make fun with the data and models.  

Thus, in this project, I presented a trial to build a model to predict the game results (i.e., which team wins) given the heroes picked by both teams.  

In fact, predicting the game result with only heroes would be difficult, as the heroes and characteristics in DotA2 are well-balanced (which is also why the game is popular). Besides the heroes, there would be many different factors that would impact how the game goes, and it is not likely that a specific hero combo would always win over others. However, it is possible to find some imbalance from the data exploration and modeling process, which would lead to meaningful and interesting findings.

This work is also summarized in [a blog](https://shxz.medium.com/dota-2-can-i-win-the-game-with-the-heroes-picked-by-my-team-840cd6bd466d).


## 2. Tasks <a name="tasks"></a>

This project aims to create a model that can predict game results from the heroes selected by both game teams. The tasks of this project include:  
- Extract match data from OpenDota API.
- Train a model that could predict the game result from heroes selected.
- Interpret the model result. 



## 3. Data Extraction <a name="dataextraction"></a>

OpenDota is an open-source Dota data API platform where one could access Dota related data, including match, hero, team, etc. OpenDota can be an ideal source to get Dota match data. My reference for the OpenDota API platform is the [OpenDota API doc]( https://docs.opendota.com/).  

To extract data from the OpenDota API platform, an API key is required. You may obtain an API key with your steam account.  Another helpful tool is a python package named "dota2."


### Get Public Matches  

I started with the /publicMatches endpoint to get public matches, which can return 100 randomly sampled public matches for each call.  With the following code in the terminal, I could extract and save the data in a .json file:  
```
curl https://api.opendota.com/api/publicMatches > publicMatches.json
```

Another useful query parameter that can be applied to this request is "less_than_match_id," which could help get matches with a match ID lower than a given value. With this query parameter, I could obtain matches before a certain match time. The query could be something like:  
```
curl https://api.opendota.com/api/publicMatches?less_than_match_id={A_MATCH_ID} > publicMatches.json
```

As match IDs are numbered according to the match time, I could submit more than 600 queries with different maximum match IDs and obtain more than 60,000 unique matches.  

### Get Hero Dataset  

In the dataset, each hero is represented by a unique integer, named "hero_id." To know more information about each hero, I also obtained the dataset for heroes from OpenDota:
```
curl https://api.opendota.com/api/teams > teams.json
```

The match dataset I extracted is the "match_df.cvs" file in this repository. 



## 4. Analysis and Modeling <a name="modeling"></a>

The data processing and modeling process is described more in details in the [report of this work](https://github.com/sheilaxz/opendota_modeling/blob/main/CapstoneProjectReport.pdf) and the ["Dota2_DataProcessing.ipynb."](https://github.com/sheilaxz/opendota_modeling/blob/main/Dota2_DataProcessing.ipynb) In this README file, I just briefly summarize the features and the model. 

The features I used for modeling included: heroes, the primary attributes of the heroes, roles or the heroes, match duration.  I used the Light Gradient Boosting Model (LGBM) to train and predict the game results. 

The dataset is firstly split into a training set (80% of the data) and a testing set (20% of the data). I use the training set to train the model (with cross-validation), and the test set to test the performance of the trained model. To train the model, I set an LGBM pipeline to train the model and test different parameters for the model.

Results and discussions can also be found in the [report of this work](https://github.com/sheilaxz/opendota_modeling/blob/main/CapstoneProjectReport.pdf) and the ["Dota2_DataProcessing.ipynb."](https://github.com/sheilaxz/opendota_modeling/blob/main/Dota2_DataProcessing.ipynb) The performance (metric: AUC) of the trained model for the training set reaches 0.70076, while the AUC for the testing set is 0.60341. The AUC values for both training and testing sets show that the data/model has a certain degree of predictability for my model. 

As expected, the model performance is poor if only using heroes selected to predict the game results. This is reasonable since a game result should not be purely determined by the players' heroes or how long the game lasts. However, the model may tell me some other information.  

By taking a look at the feature importance of the model, it can be noticed that "Duration" has the highest importance score. As a feature that could interact and work together with all other features, I would believe duration may contribute the most to the predictive model.  

It can also be noticed that almost all non-hero features have higher importance than hero features. While there seems no observable importance difference for hero roles, the "Strength" primary attribute seems to have higher importance than other primary attributes. However, this does not mean that the number of strength heroes itself would impact the game's result. The importance scores tell the predictive ability of variables in the model. It has no direct relationship with whether a single feature has a positive or negative effect on the game's result because the variables in the tree model all affect each other and play a role in the prediction.  But from the model, we might see that a balanced combination of roles or primary attributes would impact the game result more than certain heroes.  

Let's then take a look at the hero features. Some heroes show higher importance than other heroes. These heroes are:  

Anti-Mage, Luna,Orge Magi, Doom, Zeus, Drow Ranger, Templar Assassin, Silencer, Invoker, Spectre 

Among these heroes, Hero Anti-Mage and Hero Luna appear at the forefront of the importance ranking for both teams, indicating that the role of these heroes in improving the predictive ability of the model is stable, regardless of whether they appear in the Radiant side or the Dire side.  

Comparing the Dota Hero Meta Statistics for this month, we can see that Luna, Ogre Magi, Drow Ranger, Silencer, and Spectre are all heroes that are very commonly picked with high winning rates. Meanwhile, Invoker and Anti-mage are two heroes that are widely picked but with comparatively low winning rates. On the other hand, Doom, which has relatively high importance, has a low picking rate and a low winning rate. Even though we can not say that a single hero would definitely increase or decrease the winning probability, these statistics might reflect why they have a higher importance score in the model to some degree.



## 5. File Description <a name="files"></a>

### Datasets

- match_df.csv: A dataset of match data I extracted as described in ["DotA2_OpenDotaExtraction.ipynb"](https://github.com/sheilaxz/opendota_modeling/blob/main/DotA2_OpenDotaExtraction.ipynb). This is the dataset I used for analysis and modeling in this project.


### Data Processing and Other Files

"DotA2_OpenDotaExtraction.ipynb" is a Jupyter Notebook file showing how I extracted the dota match data from OpenDota API platform.  

"Dota2_DataProcessing.ipynb" is the Jupyter Notebook file showin the data exploration and cleaning, feature selection, model training and testing.  

"lgbm.pkl" is a pickle file of the trained LGBM model.  

"heroes.json" is the hero dataset with detailed hero information extracted from OpenDota API platform. 

"CapstoneProjectReport.pdf" is a pdf file of a report of this whole project. This report describes how each step in this project is taken. 



## 6. Licensing, Authors, and Acknowledgements <a name="licensing"></a>

The project is heavily inspired by Chengjun Hou's blog ["Ten lines of code to predict TI7,"](https://cosx.org/2017/05/rdota2-seattle-prediction/) in which he performed data extraction and analysis using OpenData API data and R language3.  

The project also borrows some thought from Bill Prin's blog ["Python and DotA2: Analyzing Team Liquid's Io success and failure."](https://medium.com/@waprin/python-and-dota2-analyzing-team-liquids-io-success-and-failure-7d44cc5979b2)

Also thank Udacity for providing helpful guidance and feedback.

