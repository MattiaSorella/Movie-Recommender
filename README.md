# Movie-Recommender
It's a programm that Reccomends a list of movies that I got from data that I got from the TasteDive API combined with data that I got from the OMDB API

import requests_with_caching
import json

def get_movies_from_tastedive(movie_name):
    baseurl = "https://tastedive.com/api/similar"
    para_dict = {} #Setting up the empty dict
    para_dict["q"] = movie_name #We want our variable name to be the movie
    para_dict["type"] = "movies" # We want only movies
    para_dict["limit"] = 5 # We want the top 3 results
    resps = requests_with_caching.get(baseurl, params = para_dict)
    
    movie_name_ds = resps.json()
    return movie_name_ds


def extract_movie_titles(dic):
    list_of_titles = [i['Name'] for i in dic['Similar']['Results']]
    print(list_of_titles)
    return list_of_titles


def get_related_titles(lst_of_movies):
    ls = []
    for movie in lst_of_movies:
        ls.extend(extract_movie_titles(get_movies_from_tastedive(movie)))
    return list(set(ls))


def get_movie_data(movie_title):
    baseurl = "http://www.omdbapi.com/"
    para_dict = {}
    para_dict["t"] = movie_title
    para_dict["r"] = "json"
    response = requests_with_caching.get(baseurl, params = para_dict)
    
    movie_title_d = response.json()
    return movie_title_d


def get_movie_rating(d):
    rating = d["Ratings"]
    for item in rating:
        if item["Source"] == "Rotten Tomatoes":
            return int(item["Value"][:-1])
    return 0


def get_sorted_recommendations(list_of_movies):
    new_lst = get_related_titles(list_of_movies)
    nd = {}
    for item in new_lst:
        rating = get_movie_rating(get_movie_data(item))
        nd[item] = rating
    print(nd)
    return [item[0] for item in sorted(nd.items(), key = lambda x : (x[1], x[0]), reverse = True)]
