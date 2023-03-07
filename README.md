# Steam Games Recommandation based on Game Content

So, when I browse Kaggle dataset, I found a Steam's dataset of their 40,000 games library. It contains information from the name of the games, the genre, even the price and discounted price are in the dataset. As a gamer myself, I admit I was eager to analyze this data and I think I can do something like this with this data:

![gambar](https://user-images.githubusercontent.com/52767438/223320001-fc5fb787-c96c-4237-b1a4-c61cbf094e07.png)

Yep, it is a recommendation game based on similarity of the game you played before. So, I searched around how to search context similarity between the games you have played and the rest of the games in Steam's Library. And I found the Content Based Recommendation system.

To put it simple, Content Based Recommendation (CBR) system gives you a recommendation based on similarity of items you have consumed before with others in the store/platform library. For example, if you played an action game like Devil May Cry, most likely Steam will recommend you a game similar to DMC like Bayonetta, Scarlet Nexus, Ninja Gaiden Sigma, or another action game with hack and slash combat. The similarity of these games comes from many factors, such as description, genre, publisher, developer, and many others. After that, the system can ranked the similarity between a game and another games in Steam Library.

## Data Preparation
You can download the dataset here: https://www.kaggle.com/datasets/trolukovich/steam-games-complete-dataset
Here is the preview I screenshot from my notebook:

![gambar](https://user-images.githubusercontent.com/52767438/223322308-d03e6a40-34b4-4c1f-93c1-289905342fc9.png)

The data preparation I did is to prepare the features that have a potential to be an identifier of similarity between games. Basically these features contains information about the game such as description, rating, genre, and any information we can get. In this project, I used features like description, genre, rating, and another details about the game.

### Types Games
![gambar](https://user-images.githubusercontent.com/52767438/223322390-554bbda5-fb4e-4127-bfd6-f854b3a43bac.png)

I am curious about this feature. Turns out, there are three different types of games in Steam. But, there is also nan type or null data.

![gambar](https://user-images.githubusercontent.com/52767438/223322426-ff47b031-cb6a-47fc-9d34-fce4355d4d95.png)

For the games that have no type, these games also did not have many information about theirself. Index 704 and 35169 does not have enough info to be analyzed, because most of the data of these rows are Null. So I drop it.

### Description
There are two columns that can be used as description games feature, these are desc_snippet and game_description. I need to compare between these two columns.

![gambar](https://user-images.githubusercontent.com/52767438/223322581-c99d018e-0087-42f2-8399-ec18c86fb034.png)

Columns `desc_snippet` and `game_description` has the same meaning, but I prefer `game_description` because this column has more data (words to be said) than `desc_snippet`. So I take `game_description` data, and if it is null, it will be replaced with `desc_snippet`

Next, I clean up the description. There are sentences that were repeated in almost every games, these are "About This Game" and "About This Content". Also, I want the description no longer contain stopwords such as "and", "or", etc.

### Reviews
I used `all_reviews` column to evaluate the review of the game. About the rating, I will use the predicate that steam used, such as Very Positive, Mostly Positive, etc. I got dictionary about rating predicate in steam in this discussion: https://steamcommunity.com/discussions/forum/0/1744483505466407549/

Then, I used that dictionary to be used in `all_reviews` predicate:
- Overhwelmingly Positive = 9
- Very Positive = 8
- Positive = 7
- Mostly Positive = 6
- Mixed = 5
- Mostly Negative = 4
- Negative = 3
- Very Negative = 2
- Overwhelmingly Negative = 1

The problem about this feature is the information about rating predicate and reviewers number of a games contained in a column. So I need to extract these two informations.

**Extracting Review Predicate**

![gambar](https://user-images.githubusercontent.com/52767438/223322779-cb000b7b-e2dd-4f21-b83f-79ee3fe0b67b.png)

Notice that every rating predicate is located in the first sentence/word before the first comma. I used split function to separate this element in the list. 

![gambar](https://user-images.githubusercontent.com/52767438/223322870-e5ae9f3a-cc50-40df-a09b-16439a1de024.png)

After I split the all_reviews column, this column will contain list of information about reviews separated by ",". Now, I just need to get the first element from the list in every row, and I will get the rating predicate.

![gambar](https://user-images.githubusercontent.com/52767438/223322924-f302902b-82bc-4c7f-a656-316130645f4e.png)

Next, I transform this predicate to integer rate according to the dictionary I used.

![gambar](https://user-images.githubusercontent.com/52767438/223322961-105374a7-46f5-465a-b0bd-d8e35c647e8e.png)

**Extracting Number of Reviewers**

![gambar](https://user-images.githubusercontent.com/52767438/223323008-a796ce3d-8482-48dc-aa8b-9c91d9f524bc.png)

Every reviewer number is stored in a parentheses bracket. I just need to detect the value in this parentheses.

![gambar](https://user-images.githubusercontent.com/52767438/223323136-848d4f70-dea3-41ff-a29a-4efccb54f320.png)

![gambar](https://user-images.githubusercontent.com/52767438/223323100-40eddb61-5f2a-4650-ab18-06245e52d937.png)

Need to delete the game that not qualify enough to be predicated (too few reviews), the value such as '7 user reviews,- Need more user reviews to generate a score', '1 user reviews,- Need more user reviews to generate a score', etc.

![gambar](https://user-images.githubusercontent.com/52767438/223323207-d33c2685-250d-470f-a402-73deb0e1c8b2.png)

![gambar](https://user-images.githubusercontent.com/52767438/223323257-c52baf8a-6b87-4029-9b0b-120d52170d01.png)

Then, I used a formula from IMDb to weight the reviews based on how many reviewers in a game. Source: https://math.stackexchange.com/questions/169032/understanding-the-imdb-weighted-rating-function-for-usage-on-my-own-website

![gambar](https://user-images.githubusercontent.com/52767438/223323289-47eb9853-eb4e-4c38-ac69-a74fdb729a92.png)

### Popular Tags, Game Details, and Genre

![gambar](https://user-images.githubusercontent.com/52767438/223323386-619a073c-533e-47f8-a8f6-52a3535a62b7.png)

I want to minimize the abusive of data from a game and the originality of the data of a game. Column `popular_tags` and `genre` have similar meaning. Column `genre` is originated from the developer/publisher of the game, meanwhile `popular_tags` originated from users/players who reviewed it. I do believe that users/players who reviewed the game have the knowledge of the game which they reviewed, but to be fair for I to calculate the similarity of the games, I believe only using `genre` column is suffice enough to calculating similarity between games. So, for this project I only use `genre` column instead of `popular_tags` to identify the genre of the game. 

### Used Features
Next I made a dataframe using only columns that needed in identify similarity between games.

![gambar](https://user-images.githubusercontent.com/52767438/223323469-5fe5b8c4-24fb-45c2-aa64-a6c33695b709.png)

Before finding similarity, I need to handle null data. If the name column is null or empty, I will exclude this game to the modelling because I can't identify which game is this. If the description_clean is null or empty and one of game_details and genre columns are empty or null, I will also exclude the game because I assume this game does not have enough information to find the similarity with other games.

![gambar](https://user-images.githubusercontent.com/52767438/223323549-60cd12c6-9ee7-47e1-9fa5-8dc43c35edea.png)

## Modelling
### Computing Cosine Similarity
I combine the three similarity identifier to be calculated in TF-IDF Vectorizer

![gambar](https://user-images.githubusercontent.com/52767438/223323672-e7ce6c00-7aa5-4aed-84dc-ed9369ae95bb.png)

### Content Based Recommendation Function

![gambar](https://user-images.githubusercontent.com/52767438/223323753-4d7edaab-56c1-4692-a8c0-0c21b19340be.png)

## Example Result
- Result 1 for game "DOOM":

![gambar](https://user-images.githubusercontent.com/52767438/223323825-aabf904f-f5f3-4a54-94d5-3bbc8ba1ef7c.png)

- Result 2 for game "Ni no Kuni™ II: Revenant Kingdom":

![gambar](https://user-images.githubusercontent.com/52767438/223323966-88d57560-5447-4fec-ac16-a8a1a557f226.png)

- Result 3 for game "FINAL FANTASY X/X-2 HD Remaster":

![gambar](https://user-images.githubusercontent.com/52767438/223324057-cb70993a-2881-4dd8-8011-7101b58613f0.png)

The recommendation result of the algoriithm, seems good enough. But because I more prefer to find the most similar games first then I sorted by weighted rating, the games that has been recommended are

## Notes
### Potential of Improvement
1. Try another similarity metrix out there. There are similar algorithm like cosine similarity such jaccard similarity, euclidean distance, manhattan distance, etc.
2. The data preparation process that I did is one of the rushing process because I did this project only in one day. Perhaps it could be enhanced to be better. The ones that I can think of are text cleaning process, adding data source such as number of players, popularity of the game could be added, and maybe the sorting preference can be based weighted rating first then sort by similarity.
3. This data can be given a separate ID to make it easier to identify game recommendations to be analyzed.
4. Try another null treatment to the data.

### References:
1. Steam discussion about rating: https://steamcommunity.com/discussions/forum/0/1744483505466407549/
2. IMDb Weighted Rating Formula: https://math.stackexchange.com/questions/169032/understanding-the-imdb-weighted-rating-function-for-usage-on-my-own-website
3. Another Content Based Recommendation Project with Same Dataset: https://www.kaggle.com/code/fetenbasak/content-based-recommendation-game-recommender
4. About Content Based Recommendation and TF-IDF: https://www.analyticsvidhya.com/blog/2015/08/beginners-guide-learn-content-based-recommender-systems/
5. Cosine Similarity Scikit-Learn: https://scikit-learn.org/stable/modules/metrics.html
6. Cosine Similarity Explanation: https://builtin.com/machine-learning/cosine-similarity
