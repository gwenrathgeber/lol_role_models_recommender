# League of Legends Role Models: A Recommendation System for Skill Training and Study

__Gwen Rathgeber | [GitHub](https://github.com/gwenrathgeber) | [LinkedIn](https://www.linkedin.com/in/gwenrathgeber/)__


Special thanks to [Uthgar](https://twitter.com/Dr_Uthgar) at Mobalytics.gg for suggesting this project!


The accompanying Flask app to this project can be viewed at https://github.com/gwenrathgeber/lol_role_models_app.
  
    
      
_N.B: this notebook is meant to summarize the code and logic behind developing the [Flask app](https://github.com/gwenrathgeber/lol_role_models_app) which is the true final product of the project, and is not meant to be run top-to-bottom locally (though it can be)._

_If you do wish to run this code locally, please be aware that querying the Riot API for our Role Models gameplay data takes over 24 hours on a basic developer key even with our implementation of multi-threading API requests by region, and you may wish to make alterations to the `scrape_seeds` function to limit the number of matches you are querying. The scraper function will save as it goes, so running the scraper intermittently and refreshing your API key every 24 hours is also viable._

## Problem Statement

League of Legends is a Multiplayer Online Battle Arena game known for attracting devoted players with a strong competitive drive. As one of the earliest games to popularize online video game streaming on platforms such as Twitch.tv, League demonstrated the appeal of not only playing games, but watching others compete. Due to the complexity of the game, a highly effective method of improving is studying the gameplay of higher-skilled players. We wish to improve this process for players by providing a recommendation system which matches players with Role Models: high-ranking players whose playstyles are similar to the user. Users can then use other tools to watch replays of recent games played by their recommended Role Models, simplifying and enhancing the process of finding high-level games to study through personalization and filtering.

## Executive Summary
In order to implement the recommender, we require a body of high-level players who will be recommended to users. All necessary data will be acquired from the Riot Games API, which can provide match-level data for games played by on a given account.

After scraping the data we need, we will need to combine each players' match history into a single row of aggregate statistics, using subject-matter knowledge to identify metrics which will be represent distinctive play patterns while also being comparable between lower- and higher-skilled players.

Finally, we will implement the recommendation system itself, which will involve pulling an individual player's match history, performing our aggregation, and comparing their statistics against our Role Model candidates.

Evaluation of this recommendation system is unfortunately limited to subjective interpretation of results by myself and test users, as it is not designed to drive a particular business objective or KPI. However, if a similar tool was deployed on one of the many stat-tracking and personal improvement websites which exist for League of Legends Players, some important target metrics would include the page bounce rate for the tool, churn and retention rates of tool users vs. non-users, and user feedback in the form of qualitative and quantitative satisfaction surveys that could be added to the results page. 

## Data Dictionary

The following endpoints of the Riot API were used over the course of this project, please consult their documentation for details of the fields:
 - [Summoner by Account ID Endpoint](https://developer.riotgames.com/apis#summoner-v4/GET_getByAccountId)
 - [Summoner by Account Name Endpoint](https://developer.riotgames.com/apis#summoner-v4/GET_getBySummonerName)
 - [Challenger Leagues Endpoint](https://developer.riotgames.com/apis#league-v4/GET_getChallengerLeague)
 - [Match Histories Endpoint](https://developer.riotgames.com/apis#match-v4/GET_getMatchlist)
 - [Match Data Endpoint](https://developer.riotgames.com/apis#match-v4/GET_getMatch)
 - [Timeline Data Endpoint](https://developer.riotgames.com/apis#match-v4/GET_getMatchTimeline)

The following is a representation of the custom fields we generated in our game data analysis:


Column | Data Type| Description
-|-|-
csd_10 | float|Average CS (Creep Score) Difference Between Player and Opponent at 10 minutes
gold_d_10 | float| Average Gold Difference Between Player and Opponent at 10 minutes
xpd_10| float|Average Experience Difference Between Player and Opponent at 10 minutes
dmg_share| float| Average Percent of Team's Total Damage to Champions Dealt by Player
dmg_taken_share| float| Average Percent of Team's Total Damage Taken Received by Player
vision_score| float| Average Vision Score of Player
kill_participation|float|Average Percent of Team's Kills a Player was Involved In
obj_dmg_share|float| Average Percent of Team's Total Damage to Epic Monsters and Structures by Player
dragons|float|Average Percent of Dragons Taken by Player's Team
barons|float|Average Percent of Barons Taken by Player's Team
wards_cleared |float | Average Ratio of Wards Cleared to Wards Placed by Player
vision_wards_purchased |float | Average Ratio of Vision Wards Purchased to Wards Placed by Player
kda_early | float| Average Ratio of Kills + Deaths + Assists Occurring before 10 minutes
kda_mid |float | Average Ratio of Kills + Deaths + Assists Occurring between 10 and 20 minutes
kda_late |float| Average Ratio of Kills + Deaths + Assists Occurring after 20 minutes
solo_kills |float | Average Ratio of Kills with no Assists by Teammates
teamfight_kills |float | Average Ratio of Kills with two or more Assists by Teammates
skirmish_kills |float | Average Ratio of Kills with only one Assist by Teammates
wards_early |float | Average Ratio of Wards Placed before 10 minutes
wards_mid |float |  Average Ratio of Wards Placed between 10 and 20 minutes
wards_late |float |  Average Ratio of Wards Placed after 20 minutes
most_played_champ |string | Most Frequently Played Champion
most_played_role |string | Most Frequently Played Role
most_played_lane | string| Most Frequently Played Lane
role |string | Simplified Most Played Role
op_gg | string| URL of player's op.gg account profile
similarity|float|Euclidean Distance from Role Model Player to User

## Evaluation and User Feedback
As previously mentioned, our model can't be objectively evaluated easily. However, here are a summary of pros and cons in its subjective performance:

**Pros:**
- The highest-similarity results are almost always players with the same main role, and will often include multiple players with the same main champion. Insofar as role and champion are useful heuristics for playstyle, this is promising.
- We can readily observe major differences in the stat lines between good and poor recommendations, especially and most importantly on stats which are not in reference to the team, such as laning stats and warding by game phase. 

**Cons:**
- The main issue we've seen in testing is that lower-ranked input players often tend to receive recommendations who are in the lower skill range of our Role Models. This reflects the aforementioned "apples to oranges" issues that come with comparing players across large rank gaps. This isn't necessarily a problem, as it's quite possible that a low-rank player can more readily compare their gameplay to extremely good players than the very best, and it was a concession we opted into in order to keep features which we felt had meaningful contributions to a player's "fingerprint."

Finally, here is a bit of user feedback from one of our testers:

"Personally, I found the champion stuff the most interesting part of the data, especially identifying trends across multiple players. I feel like the whole thing gave me some interesting ideas and allowed me to look at the way I play in a different way." 

This tester was what's known as a "One-Trick Pony" in League, which is to say that they almost exclusively play the same champion in every game. They've historically had trouble finding new champions to play which appeal to them given a finite amount of time to experiment outside of work, and so they were somewhat less interested in reviewing replays of similar players, and more interested in finding out what champions players like them enjoyed playing. 

Our methodology can easily be extended to recommending champions who have similar overall playstyles to a given player, and will be in the final Flask App. While users can certainly explore their Role Model recommendations' most-played champions, improving the user experience by suggesting champions which are a good fit would be a easily-implementable value add.

## Future Goals
- Deploy Flask App on Heroku (Main Hurdles: Managing API Rate Limits across asynchronous users, adding styling)
- Add Champion Recommendation component: what champions are most similar to your playstyle?
- Visualize similarity for the user to enhance sense of personalization and give feedback on playstyle


## References
- [Riot Games API](https://developer.riotgames.com/apis)
- [How Well does Personalized Marketing Work?](https://knowledge.wharton.upenn.edu/article/recommended-for-you-how-well-does-personalized-marketing-work/)
- [Recommender Systems in Python 101](https://www.kaggle.com/gspmoreira/recommender-systems-in-python-101)
- [Building the Next New York Times Recommendation Engine](https://open.blogs.nytimes.com/2015/08/11/building-the-next-new-york-times-recommendation-engine)
- [The Netflix Recommender System: Algorithms, Business Value, and Innovation](https://dl.acm.org/doi/10.1145/2843948)
- [How Machine Learning Fuels Your Netflix Addiction](https://www.rtinsights.com/netflix-recommendations-machine-learning-algorithms/)
- [The Treasure Chest of League of Legends: Riot’s APIs](https://medium.com/@montane/the-treasure-chest-of-league-of-legends-riots-apis-64816f4d3f84)
- [Part 1 of Riot API: Data Downpour](https://towardsdatascience.com/data-downpour-b1c4b41d7862)