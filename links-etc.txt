#### Web app examples

- Surgeon Scorecard: https://projects.propublica.org/surgeons/
- Dollars for Docs: https://projects.propublica.org/docdollars/
- Shale Play: http://stateimpact.npr.org/pennsylvania/drilling/
- Guantanamo Docket: https://www.nytimes.com/interactive/projects/guantanamo

#### SQLite

DB Browser: https://sqlitebrowser.org/

#### Web apps in Python

Flask: http://flask.pocoo.org/
Django: https://www.djangoproject.com/
SQLAlchemy: https://www.sqlalchemy.org/

#### Model for School

  class School(db.Model):
    __tablename__ = 'schools'
    __table_args__ = {
      'autoload': True,
      'autoload_with': db.engine
    }
    dbn = db.Column(db.String, primary_key=True)

#### Model for Score

  class Score(db.Model):
    __tablename__ = 'sat_scores'
    __table_args__ = {
      'autoload': True,
      'autoload_with': db.engine
    }
    dbn = db.Column(db.String, primary_key=True)

#### Using Flask templates

in layout.html:

    {% block content %}{% endblock %}

in the template:

    {% extends 'layout.html' %}

    {% block content %}
    blah blah your content goes here
    {% endblock %}

#### Freezing

    from flask_frozen import Freezer
    from app import app, School

    # app.config['FREEZER_RELATIVE_URLS'] = True
    # app.config['FREEZER_DESTINATION'] = 'docs'

    freezer = Freezer(app)

    if __name__ == '__main__':
        freezer.freeze()

