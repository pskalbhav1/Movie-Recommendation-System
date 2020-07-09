# Movie-Recommendation
A Movie Management and Recommendation System ( as a part of DevSprint )

List of features to be included:

      1.Searching for a movie

      2.Filtering movies according to a given criteria

      3.Details about all the movies

      4.Implement a rating system for movies


Judging Criteria

  Frontend: 

      1.UI

      2.Intuitiveness

      3.Ease of Access

  Backend:

      1.Efficiency

      2.Structure(schemes, routes, workflow)


# Solution :

   Have you ever wondered how Netflix recommends you movies to watch?  What is the idea upon which this recommendation is done?
   
   It turns out that there are three types of recommendations possible. Let's see what they are.
   
   Three popular recommendation systems:
           
      1.Popularity based recommendation engine
      
      2.Content based recommendation engine
      
      3.Collaborative filtering based recommendation engine
      
   # Popularity based recommendation :
     
   A very good example of this is the "trending list" that you see on Netflix. It keeps a view count of the movies being watched and displays them in decreasing order of their view counts.
      
   # Content based recommendation:
   
   This type of recommendation systems, takes in a movie that a user currently likes as input. Then it analyzes the contents (rating , overview, genre, cast, director etc.) of the movie to find out other movies which have similar content. Then it ranks similar movies according to their similarity scores and recommends the most relevant movies to the user.
      
   # Collaborative filtering based recommendation:
   
   This algorithm at first tries to find similar users based on their activities and preferences.
   
  # Code : 
      
  Here, we have used content based recommendation to predict movies. A movie that the user likes is taken as input in webpage. Then its contents are analyzed from the available dataaset. We have made use of the contents - keywords, genre, cast and director from the given dataset.
  
     import pandas as pd
     df = pd.read_csv("movie_dataset.csv")
     features = ['keywords','cast','genres','director']
     def combine_features(row):
         return row['keywords']+" "+row['cast']+" "+row['genres']+" "+row['director']
     for feature in features:
         df[feature] = df[feature].fillna('') #filling all NaNs with blank string
         
  Now, we need to find a way to represent these texts as vectors. The CountVectorizer() class from sklearn.feature_extraction.text library can do this for us.
  
      from sklearn.feature_extraction.text import CountVectorizer
      cv = CountVectorizer() #creating new CountVectorizer() object
      count_matrix = cv.fit_transform(df["combined_features"]) #feeding combined strings(movie contents) to CountVectorizer() object
      indices = pd.Series(df.index, index=df['title'])
      df = df.reset_index() #To start indexing them from 0
      
  Now, we need to find cosine(or “cos”) similarity between these vectors to find out how similar they are from each other. We can calculate this using cosine_similarity() function from sklearn.metrics.pairwise library.
  
      from sklearn.metrics.pairwise import cosine_similarity
      cosine_sim = cosine_similarity(count_matrix)

 Next important thing is to get recommendations for the movie the user likes. We have a function get_recommendation(movie_user_likes) for the same. The details of these movies is also obtained here from the dataset. Also, the most similar movie to the movie that the user likes is itself. We have to make sure to exclude that movie.
 
     def get_recommendations(title):
        cosine_sim = cosine_similarity(count_matrix)
        idx = indices[title]
        similar_movies = list(enumerate(cosine_sim[idx])) #accessing the row corresponding to given movie to find all the similarity scores for that movie and then enumerating over it
        sim_scores = sorted(similar_movies,key=lambda x:x[1],reverse=True) 
        sim_scores = sim_scores[1:11]
        movie_indices = [i[0] for i in sim_scores]
        tit = df['title'].iloc[movie_indices]
        dat = df['overview'].iloc[movie_indices]
        dire = df['director'].iloc[movie_indices]
        cas = df['cast'].iloc[movie_indices]
        vote_avg = df['vote_average'].iloc[movie_indices]
        genre = df['genres'].iloc[movie_indices]
        link = df['homepage'].iloc[movie_indices]
        return_df = pd.DataFrame(columns=['title','overview','director','cast','vote_average','genres','homepage'])
        return_df['title'] = tit
        return_df['overview'] = dat
        return_df['director'] = dire
        return_df['cast'] = cas
        return_df['vote_average'] = vote_avg
        return_df['genres'] = genre
        return_df['homepage'] = link
        return return_df
     
  Next step is setting up the main route using flask to render the HTML pages and act as a link between the frontend and backend. The following code helps in doing the same.
  
     import flask
     app = flask.Flask(__name__, template_folder='templates')
     # Set up the main route
     @app.route('/', methods=['GET', 'POST'])

     def main():
        if flask.request.method == 'GET':
            return(flask.render_template('index.html'))
            
        if flask.request.method == 'POST':
            m_name = flask.request.form['movie_name']
            m_name = m_name.title()
        
            if m_name not in all_titles:
                return(flask.render_template('negative.html',name=m_name))
            else:
                result_final = get_recommendations(m_name)
                names = []
                dates = []
                director = []
                cast = []
                rating = []
                genres = []
                links = []
                for i in range(len(result_final)):
                    names.append(result_final.iloc[i][0])
                    dates.append(result_final.iloc[i][1])
                    director.append(result_final.iloc[i][2])
                    cast.append(result_final.iloc[i][3])
                    rating.append(result_final.iloc[i][4])
                    genres.append(result_final.iloc[i][5])
                    links.append(result_final.iloc[i][6])

                return     flask.render_template('positive.html',movie_names=names,movie_date=dates,movie_director=director,movie_cast=cast,movie_rating=rating,movie_genres=genres,movie_links=links,search_name=m_name)

        
    if __name__ == '__main__':
        app.run()
        
And yes, we are done with content based movie recommendation system using HTML, Flask and Python.

# Trending List:

  The trending list is created based on the viewer votes that are present in the dataset. The top 5 movie names will be displayed along with their genres. The list is created based on the decreasing order of the ratings and then the top 5 names are displayed. 
  
     def trending():
         sim_scores=df1.groupby('title')['vote_average'].count().sort_values(ascending=False)
         sim_scores = indices[all_titles] 
         movie_indices = sim_scores[0:5]
         tit = df1['title'].iloc[movie_indices]
         genre = df1['genres'].iloc[movie_indices]
         return_df = pd.DataFrame(columns=['title','genres'])
         return_df['title'] = tit
         return_df['genres'] = genre
         return return_df
         
 This output gets displayed on the webpage and the same is done using flask, as available in the code.
