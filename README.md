# Movie Recommendation System (Collaborative Filtering with Interactive Input)

This project walks through building a **movie recommendation system** using **Python**, **Pandas**, and **Jupyter Notebook widgets**. By the end, you'll be able to type the name of a movie into an input box and instantly get recommendations for other movies you might enjoy, based on ratings from users with similar preferences.

## Project Description

In this project, we use a basic collaborative filtering technique to recommend movies. The logic is based on identifying users who highly rated a particular movie and finding what other movies they also liked.

The project is implemented entirely in a Jupyter Notebook using `pandas` and `ipywidgets`.

## How It Works

1. **User Input:**
   - An interactive text input box is shown using `ipywidgets.Text`.
   - As the user types a movie title (minimum 5 characters), the system searches for the closest match.

2. **Recommendation Logic:**
   - Find users who rated the selected movie highly (above 4 stars).
   - Look for other movies these users also rated highly.
   - Normalize and score these recommendations based on how unique they are to the similar users compared to all users.

3. **Scoring Mechanism:**
   ```
   score = (# of similar users who liked the movie) / (# of all users who liked the movie)
   ```
   - This boosts movies that are *popular among similar users but not necessarily among all users*.

4. **Top Recommendations:**
   - The top 10 recommended movies are displayed along with their titles, genres, and relevance score.

## Example Output

If the input is:
```
Toy Story
```

The system recommends:

| Score | Title                        | Genres                        |
|-------|------------------------------|------------------------------ |
| 5.225  | Toy Story 2                 | Animation\|Children\|Comedy   |
| 4.405  | A Bug's Life                | Animation\|Adventure\|Comedy  |
| 4.354  | Toy Story 3                 | Animation\|Adventure\|Children|
| ...   | ...                          | ...                           |

## Requirements

- Python 3.x
- Jupyter Notebook
- pandas
- ipywidgets


## Files Used

- `movies.csv` — Movie metadata including titles and genres.
- `ratings.csv` — User ratings for different movies.

You can download the required datasets from Google Drive:
- [Download movies.csv](https://drive.google.com/file/d/18agdNmeX2G0sxqQf4VFSGTFFDyyBs68w/view?usp=sharing)
- [Download ratings.csv](https://drive.google.com/file/d/1H8m4D1QHDSH5BnMJ-Ek09dtqdWlnAsnW/view?usp=sharing)


## Code Highlights

### Main Recommendation Function

```python
def find_similar_movies(movie_id):
    similar_users = ratings[(ratings["movieId"] == movie_id) & (ratings["rating"] > 4)]["userId"].unique()
    similar_user_recs = ratings[(ratings["userId"].isin(similar_users)) & (ratings["rating"] > 4)]["movieId"]
    
    similar_user_recs = similar_user_recs.value_counts() / len(similar_users)
    similar_user_recs = similar_user_recs[similar_user_recs > .1]
    
    all_users = ratings[(ratings["movieId"].isin(similar_user_recs.index)) & (ratings["rating"] > 4)]
    all_user_recs = all_users["movieId"].value_counts() / len(all_users["userId"].unique())
    
    rec_percentages = pd.concat([similar_user_recs, all_user_recs], axis = 1)
    rec_percentages.columns = ["similar", "all"]
    
    rec_percentages["score"] = rec_percentages["similar"] / rec_percentages["all"]
    
    rec_percentages = rec_percentages.sort_values("score", ascending = False)
    
    return rec_percentages.head(10).merge(movies, left_index = True, right_on = "movieId")[["score", "title", "genres"]]
```

### Interactive Input with Widgets

```python
movie_name_input = widgets.Text(
    value = "Toy Story",
    description = "Movie Title:",
    disabled = False
)

recommendation_list = widgets.Output()

def on_type(data):
    with recommendation_list:
        recommendation_list.clear_output()
        title = data["new"]
        if len(title) > 5:
            results = search(title)
            movie_id = results.iloc[0]["movieId"]
            display(find_similar_movies(movie_id))
            
movie_name_input.observe(on_type ,names="value")

display(movie_name_input, recommendation_list)
```

## Future Improvements

- Build a web interface using Flask or Streamlit.
- Add content-based filtering (genres, tags) as fallback.

