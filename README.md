---
layout: single
title: 'Python Flask'
---

Using [this!](https://www.youtube.com/watch?v=Z1RJmh_OqeA&t=1325s)


## MAKE APP

1. Make & cd project directory 
2. Make and start virtual environment

```console
python3 -m venv v-env
source v-env/bin/activate
```
3. Install packages

```console
pip3 install flask flask-sqlalchemy
```
4. Make `app.py`

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return "Hello, world!"

if __name__ == "__main__":
    app.run(debug=True)
```

5. Create two folders: `static` and `templates`

5b. Add this line to `app.py` 

``` python
from flask import Flask, render_template
```

6. Add `index.html` to templates folder (with basic html) and update `app.py` to return `render_template` instead of just 'hellow world'

```python
def index():
    # return "Hello, world!"
    return render_template('index.html')
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    Hello cats!
</body>
</html>
```

7. Add `base.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    {% block head %}{% endblock %}
</head>
<body>
    {% block body %}{% endblock %}
</body>
</html>
```

7b. Update `index.html`

```html
{% extends 'base.html' %}

{% block head %}
<h1>What is this? Todos for cats?!</h1>
{% endblock %}

{% block body %}

{% endblock %}
```

8. Make a css file and folder within `static` -- `static/css/main.css`

```css
body {
    margin: 0;
    font-family: sans-serif;
}
```

9. Link the stylesheet

9a. Update `app.py`

```python
from flask import Flask, render_template, url_for
```

9b. Add link to `base.html`

```html
<link rel="stylesheet" href="{{ url_for('static', filename='css/main.css') }}">
```
## ADD DATABASE

10. Database time!

app.py
```python
from flask import Flask, render_template, url_for
from flask_sqlalchemy import SQLAlchemy
# NEW LINE!! ^^^

app = Flask(__name__)

app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///test.db'
db = SQLAlchemy(app)
# NEW LINES!! ^^^
# Telling our app where our DB is located

@app.route('/')
def index():
    # return "Hello, world!"
    return render_template('index.html')

if __name__ == "__main__":
    app.run(debug=True)
```
NOTE: sqlite:///
`////` = absolute path
`///` = relative path, what we want

11. Add models

```python
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///test.db'
db = SQLAlchemy(app)

class Todo(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.String(200), nullable=False)
    date_created = db.Column(db.DateTime, default=datetime.utcnow)

    def __repr__(self):
        return '<Task %r>' % self.id
```

and add this to the top
```python
from datetime import datetime
```

12. Activate environment

12a. Make sure you are still in virtual environment
```console
python3
>> from app import db
>> db.create_all()
```