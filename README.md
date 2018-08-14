# Flask web app how-to, steps and troubleshooting

# Templates

## Basic `app.py` template

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"

if __name__ == '__main__':
  app.run(debug=True)

```

## Basic `freezer.py` template

```python
from flask_frozen import Freezer
from app import app, School

app.config['FREEZER_RELATIVE_URLS'] = True
app.config['FREEZER_DESTINATION'] = 'docs'

freezer = Freezer(app)

if __name__ == '__main__':
    freezer.freeze()
```

## Very basic HTML templates

These do NOT use any variables, but they do use `layout.html`. You should probably fill `layout.html` in with what we did in class, just steal it from the `templates` folder.

**show.html**

```html
{% extends 'layout.html' %}

{% block content %}
<h1>This is a thing</h1>
{% endblock %}
```

**list.html**

```html
{% extends 'layout.html' %}

{% block content %}
<ul>
  <li><a href="/something/A">Thing is something</a></li>
  <li><a href="/something/B">Thing is something</a></li>
  <li><a href="/something/C">Thing is something</a></li>
</ul>
{% endblock %}
```

**layout.html**

```html
<p>This is your header</p>
{% block content %}{% endblock %}
<p>This is your footer</p>
```

# Basic steps

1. Start from the default Flask template up above, which you run with `python app.py`. It's a slightly edited version of the one from the [Flask site](http://flask.pocoo.org/).
1. Add your routes and pages with fake data. Your `html` goes in `templates/`.
1. Add bootstrap + a `layout.html`. Make sure you're using `{% block content %}` and all of that!
1. Convert your CSV to SQLite
1. Add SQLAlchemy + a data class to your app
1. Use your SQLAlchemy model to replace the fake data
1. Freeze your app with `python freezer.py`
1. Upload to GitHub

# Flask stuff

## Figure out what your routes should be

Make sure they all end in `/` so they'll be frozen correctly!

## Flask error: `view function mapping is overwriting an existing endpoint function`

Make sure all route endpoints have **different function names**. For example, if we have two `def schools` we'll get an error. Function names actually don't matter, you can name when whatever you want, as long as they're different.

## Double-check your HTML links

Your links should **end** in `/`, just like your routes, but they should also **not start with `/`**. For example, we used `<a href="schools/{{ school.dbn }}/">` to link to our school.

# SQLAlchemy stuff

## Convert your CSV file to a SQLite database

It's nice easy with `.to_sql`. You can check out the Jupyter Notebook.

## Build your models

* Name them appropriately (We had a `School`)
* Dobule-check table name that `__tablename__` is pointing to
* Make sure you have a primary key - it has to be a **unique field**. Ours was the column called `dbn`, so we used `dbn = db.Column(db.String, primary_key=True)`.

# Freezing

## Freezing error: `Did you forget a URL generator?`

When we have a route like `"/schools/<dbn>/"` we need to tell Frozen Flask what all possible values of `dbn` might be. We do this with a URL generator, like this:

```python
@freezer.register_generator
def school():
    for school in School.query.all():
        yield { 'dbn': school.dbn }
```

It grabs all of the schools and loops through the dbs. See the next error for a little more info!

## Freezing error: `Could not build url for endpoint`

Make sure the function names in `freezer.py`  match the names in `app.py`.  For example, in `app.py` we have...

```python
@app.route("/schools/<dbn>/")
def school(dbn):
  school = School.query.filter_by(dbn=dbn).first()
  return render_template("show.html", school=school)
```

...and then in `freezer.py` we have...

```python
@freezer.register_generator
def school():
    for school in School.query.all():
        yield { 'dbn': school.dbn }
```

Notice how the function names - `def school` - and the variable name - `dbn` - both match.

# GitHub isn't updating

Every time you make a change, you need to freeze again and commit/push to GitHub again.

The biggest issue is **making sure you're freezing in the right directory**. If you have one directory where you're running your code and a different directory that's your GitHub repository, you're going to have to copy your code over again and again. You probably want to just have one place - the GitHub repo - where you're doing your freezing.

# Bootstrap

* Be sure to update the `href` values in your navbar
* Maybe delete the search bar, too!
* Play around with Bootstrap. [Look at all these components!](https://getbootstrap.com/docs/4.1/components/alerts/) All you have to do is cut and paste!

# Advanced: Relating tables to each other

If you have two tables that are related to each other - for example, `schools` with a lot of `sat_scores`, and each school has `dbn` and each sat score has a `dbn` that says which school it's related to...

```python
# A few extra imports
from sqlalchemy.orm import relationship
from sqlalchemy import ForeignKey

class School(db.Model):
  __tablename__ = 'schools'
  __table_args__ = {
    'autoload': True,
    'autoload_with': db.engine
  }
  dbn = db.Column(db.String, primary_key=True)
  # this model has a relationship with the Score model
  scores = db.relationship('Score')

class Score(db.Model):
  __tablename__ = 'sat_scores'
  __table_args__ = {
    'autoload': True,
    'autoload_with': db.engine
  }
  # You add ForeignKey(schools.dbn) when declaring a column
  # to say that the dbn column you're talking about (dbn = )
  # is connected to the dbn column in the schools table (schools.dbn)
  dbn = db.Column(db.String, ForeignKey('schools.dbn'), primary_key=True)
```

Once you've done this, you could do something like

```
{% for score in school.sat_scores %}
  <p>Writing score: {{ score.writing_mean }}</p>
{% endfor %}
```

To loop through every single score that a school has associated with it. It's like a `JOIN` without writing all of the `JOIN` stuff!