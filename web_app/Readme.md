# Machine Learning to Web application

A simple tutorial, using the example provided in this repository on converting a Machine Learnng project to a web application using Flask and deploying to the web on Heroku. For more extensive tutorials on Flask and Heroku please see:

1. https://xcitech.github.io/tutorials/heroku_tutorial/
2. https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world

#### Step 1. Understand what you want to create

This may seem as a silly step, but it is the most important. Having an idea on what you want to build will hlep ypu understand what 
you will need from the Machine Leanrning part of the project. In this tutorial I want to make a simple web application to provide 
movie recommendatiions, based on a movie the user liked. The input will be a name of the movie and the output recommendations for 
that movie. For this to work I will need: (1) a database and (2) a model with the recommendations. 

### Step 2. Build your model and save the important bits

This is where you work on your ML project and optimise your models. In this example I have decided to make a simple Movie recommender using the [TMDB database from Kaggle](https://www.kaggle.com/tmdb/tmdb-movie-metadata), which is detailed in a [jupyter notebook](https://github.com/hrampadarath/JBCA_Hack_Night_Dec/blob/master/web_app/Simple_Movie_Recommender.ipynb).

In most cases, you will not want to have the webapplication doing the training and data preprocessing everytime a user opens the webapplication (in some cases you may, if the application uses data from the user). Doing the data preprocessing and ML heavy lifting offline and exporting the model will make your application more efficient. From the ML script I exported two datasets:

1. A cleaned version of the dataset - 
Here, only the columns I am interested in ('original_title', 'genres','popularity') were exported, but once I have cleaned the data.
```python
#make new data frame
movies_new_df = movies[['original_title', 'genres','popularity']]
# save new file 
movies_new_df.to_csv('tmb_movies_clean.csv', index=False)
```
2. The ML model using pickle - recommender systems can either use content or collaborative filtering. COntent is the more simple version where it uses the genres and ratings/popularity. I did this using the unsupervised version of the k-nearest-neighbours, and the exported the model uing pickle:
```python
from sklearn.neighbors import NearestNeighbors
#build the model
nn_model = NearestNeighbors(n_neighbors=5,algorithm='auto').fit(features)

#Obtain the indices of and distances to the the nearest K neighbors of each point.
distances, indices = nn_model.kneighbors(features)

#Export model indices as a pickle file
import pickle
with open('movieindices.pkl', 'wb') as fid:
    pickle.dump(indices, fid,2)
```

### Step 3. Create a virtual environment
Before starting with Flask to build the web application, create and start a virtual environment (this is important for pushing the app to Heroku):

```bash
>conda create -n env_flask python=3.6
>source activate env_flask
```
Then install Flask and other prerequisites. (You will need to install all modules you plan to use in your Flask app, here we need only pandasto additionally)

```bash
>pip install flask
>pip install gunicorn
>pip install panda
```
### Step 4. Build the Flask app
The Flask app consists of 2 main components: the main 
[app.py](https://github.com/hrampadarath/JBCA_Hack_Night_Dec/blob/master/web_app/app.py) and HTML templates, which are saved in a folder called 
[templates](https://github.com/hrampadarath/JBCA_Hack_Night_Dec/tree/master/web_app/templates). A simple app.py contains code that returns a rendered version of the html files in the templates folder. For example:

```python
from flask import Flask, request, render_template
import pickle
import pandas as pd

# initilise Flask
app = Flask(__name__)

@app.route('/') # the webpage link/extension
def main():
    return render_template('home.html') # call to the html template named "home.html"
```

The home.html file:

```html
{% extends "layout.html" %}
{% block body %}

<div class="container" style="width:100%; height:60%">
<h1>Movie Recommender App</h1>

    <div>
        <form action = "/similarByName" method = 'POST'>
	    <p> <input name="name" type ="text" placeholder="Search by Name" />
        <input type ="submit" value="submit" /> </p>
        </form>
    </div>

</div>
{% endblock %}

```
The above html file has three important features:

1. extends "layout.html" - the [layout.html](https://github.com/hrampadarath/JBCA_Hack_Night_Dec/blob/master/web_app/templates/layout.html) is a general html file that dictates the look of the application. All html files/pages in the applcation is an extention of this page.
2. <form action = "/similarByName" method = 'POST'> here a form is used to create a query (i.e. name of movie), that makes a call to a function "/similarByName" in the [app.py](https://github.com/hrampadarath/JBCA_Hack_Night_Dec/blob/master/web_app/app.py). Once this quey is made, the function "/similarByName" searches the database for a movie with a smiliar name, and returns a dictionary. 
3. This is all wrapped in an html block that is passed to layout.html, creating the webpage. 

The overall aim of this web app is that user will search for the movie in a form provided (a query that is sent to 
"/similarByName" via  "similar.html") and for each item in the list there will be an associated recommend button (that sends a query to another 
function /similarByContent also by  "similar.html"). The "/similarByContent" function is where the pickle file is used. 
Confusing? Confuffled?

To complete my app.py, I added a random movie generator, and an about page.


**Note: while building your Flask app, you can test it by exectuting it via python. It will make a webpage at http://127.0.0.1:5000/**

### Step 5. Deploy to Heroku

Will need a [Heroku](https://www.heroku.com/) account and the [HerokuCLI](https://devcenter.heroku.com/articles/heroku-cli). For our tutorial, we can use the free version of Heroku.
Create the Procfile: A Procfile is a mechanism for declaring what commands are run by your application’s dynos on the Heroku platform. Create a file called “Procfile” and put the following in it:

	web: gunicorn app:app

Create the python requirements file by running the following at the command prompt (within the virtual environment)

 	pip freeze > requirements.txt

Set up HerokuCLI using the instructions here.
Create a new app on the Heroku Website by logging into your account. You can ignore add to pipelines. I named my app: movie-recommender-example.

Login to Heroku through the command prompt:

    heroku login

Upload to Heroku (the instructions will be listed in your Heroku app page):

    > git init
    > heroku git:remote -a movie-recommender-example
    > git add .
    > git commit -am "make it better"
    > git push heroku master

Your app should be live

    use the "Open app" link on the Heroku app page
    or https://YOURAPP.herokuapp.com/
    
You can view the final web-app from this tutorial at https://movie-recommender-example.herokuapp.com/    

