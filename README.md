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

13. Add methods to routes 

`app.py`
```python
# @app.route('/')
@app.route('/', methods=['POST', 'GET'])
def index():
    # return "Hello, world!"
    return render_template('index.html')

if __name__ == "__main__":
    app.run(debug=True)
```

14. Add actions to `index.html`

```html
    <form actions="/" method="POST">
        <input type="text" name="content" id="content">
        <input type="submit" value="Add Task">
    </form>
```

15. Add requests to `app.py`

```python

from flask import Flask, render_template, url_for, request
.
.
.
@app.route('/', methods=['POST', 'GET'])
def index():
    if request.method == 'POST':
        pass
    else:
        return render_template('index.html')

```

15b. Try hitting submit -- we get error because we're not passing anything (try `return 'hello'` instead of `pass`)

16. Add ability to add todo (and add redirect to the top) and...

```python
from flask import Flask, render_template, url_for, request, redirect
.
.
.
@app.route('/', methods=['POST', 'GET'])
def index():
    if request.method == 'POST':
        # get `content` from form
        task_content = request.form['content']
        # create new instance of model
        new_task = Todo(content=task_content)

        # add to db
        try:
            db.session.add(new_task)
            db.session.commit()
            return redirect('/')
        except:
            return 'There was an issue adding your task'
    else:
        return render_template('index.html')
```
17. ...add query for non-post requests (just arriving at the page)

```python
def index():
    if request.method == 'POST':
        # get `content` from form
        task_content = request.form['content']
        # create new instance of model
        new_task = Todo(content=task_content)

        # add to db
        try:
            db.session.add(new_task)
            db.session.commit()
            return redirect('/')
        except:
            return 'There was an issue adding your task'
    else:
        # querying our database and ordering them by date created 
        tasks = Todo.query.order_by(Todo.date_created).all()
        # pass this variable to our template
        return render_template('index.html', tasks=tasks)
```

18. Add more `Jinja` to index.html so we can view all of our tasks with this fun for-loop!

```html
    <h1>Task Master</h1>
    <table>
        <tr>
            <th>Task</th>
            <th>Added</th>
            <th>Actions</th>
        </tr>
        {% for task in tasks%}
            <tr>
                <td>{{ task.content }}</td>
                <td>{{ task.date_created.date() }}</td>
                <td>
                    <a href="">Delete</a>
                    <br>
                    <a href="">Update</a>
                </td>
            </tr>
        {% endfor %}
    </table>

```

19. Add delete route (& function!) to `app.py`

```python
@app.route('/delete/<int:id>')
def delete(id):
    task_to_delete = Todo.query.get_or_404(id)

    # delete from database
    try:
        db.session.delete(task_to_delete)
        db.session.commit()
        return redirect('/')
    except:
        return 'There was a problem deleting that task'
```

20. Add route to html, and while we're there, add the update route!

```html
        {% for task in tasks%}
            <tr>
                <td>{{ task.content }}</td>
                <td>{{ task.date_created.date() }}</td>
                <td>
                    <a href="/delete/{{ task.id }}">Delete</a>
                    <br>
                    <a href="/update/{{ task.id }}">Update</a>
                </td>
            </tr>
        {% endfor %}
```

21. Add update template `update.html`
(pretty much copied from `index.html` with the table removed)

```html
{% extends 'base.html' %}

{% block head %}
<title>Task Master</title>
{% endblock %}

{% block body %}

<div class="content">

    <h1>Update Task</h1>

    <form actions="/update/{{ task.id }}" method="POST">
        <input type="text" name="content" id="content">
        <input type="submit" value="Add Task">
    </form>
</div>


{% endblock %}
```

22. Add update route (& function!) to `app.py`

```python
@app.route('/update/<int:id>', methods=['GET', 'POST'])
def update(id):
    task = Todo.query.get_or_404(id)
    
    if request.method == 'POST':
        task.content = request.form['content']
        try:
            # db.session.update(task_to_update)
            # commit() does the update for us don't need ^^
            db.session.commit()
            return redirect('/')
        except:
            return 'There was a problem updating that task'
    else:
        return render_template('update.html', task=task)
```

23. Update the update form
```html
    <form actions="/update/{{ task.id }}" method="POST">
        <input type="text" name="content" id="content" value="{{ task.content }}">
        <input type="submit" value="Update">
    </form>
```
