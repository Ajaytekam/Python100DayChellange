## Running the application

```
virtualenv -p python3 venv
. venv/bin/activate
pip3 install flask flask-sqlalchemy
python3 -m flask run --host=0.0.0.0
```

## Setting up environment 

* Setting up virtualenv  

```
pip3 install virtualenv 
virtualenv -p python3 venv
./venv/bin/activate 
```

* Installing flask modules   

```
pip3 install flask flask-sqlalchemy  
```  

## A simple Web Server 

```python  
from flask import Flask

# creating application flask instance
app = Flask(__name__)

@app.route('/')
def index():
    return "This is index page"

@app.route('/hello')
def hello():
    return "This is hello world page"

if __name__ == "__main__":
    app.run(debug=True)
```

Now run the app  

```
python3 app.py 
```

Then browse both of the pages using url `http://localhost:5000/` and `http://localhost:5000/hello`.   

## Template Page Randering  

* Import render_template library 
* Create templates folder in project root  
* Create an index.html with some sample code inside templates folder.  
* And the code will look like  

```python  
from flask import Flask, render_template

# creating application flask instance
app = Flask(__name__)

@app.route('/')
def index():
    return render_template("index.html")

if __name__ == "__main__":
    app.run(debug=True)
```

## Template Inheritence 

Its like creating a master html file which needs to be randered on each html page like page skeleton. 

Example: base.html-Contains the html template with header/body/footer and css code.  

```html 
<!DOCTYPE html>
<html lang="en">
<head>
<title>CSS Template</title>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
* {
  box-sizing: border-box;
}

body {
  font-family: Arial, Helvetica, sans-serif;
}

/* Style the header */
header {
  background-color: #666;
  padding: 30px;
  text-align: center;
  font-size: 35px;
  color: white;
}

/* Create two columns/boxes that floats next to each other */
nav {
  float: left;
  width: 30%;
  height: 300px; /* only for demonstration, should be removed */
  background: #ccc;
  padding: 20px;
}

/* Style the list inside the menu */
nav ul {
  list-style-type: none;
  padding: 0;
}

article {
  float: left;
  padding: 20px;
  width: 70%;
  background-color: #f1f1f1;
  height: 300px; /* only for demonstration, should be removed */
}

/* Clear floats after the columns */
section::after {
  content: "";
  display: table;
  clear: both;
}

/* Style the footer */
footer {
  background-color: #777;
  padding: 10px;
  text-align: center;
  color: white;
}

/* Responsive layout - makes the two columns/boxes stack on top of each other instead of next to each other, on small screens */
@media (max-width: 600px) {
  nav, article {
    width: 100%;
    height: auto;
  }
}
</style>
{% block head %}{% endblock %}
</head>

<body>

<header>
  <h2>TodoList</h2>
</header>

<section>
  <nav>
    <ul>
      <li><a href="#">Option1</a></li>
      <li><a href="#">Option2</a></li>
      <li><a href="#">Option3</a></li>
    </ul>
  </nav>

  <article>
    {% block body %}
    <!-- Body code starts and ends here -->
    {% endblock %}
  </article>
</section>

<footer>
  <p>Footer</p>
</footer>

</body>
</html>
```

There are two jinja statements 

```
\\ render data on head 
{% block body %}{% endblock %}

\\ render data on body 
{% block body %}{% endblock %}
```

Now in the renderer html page put the below blocks 

File: index.html 
```
{% extends 'base.html' %}

{% block head %}
<!-- Code for head block -->
{% endblock %}

{% block body %}
<h1>This is a Heading</h1>
<p>This is a paragraph.</p>
{% endblock %}
```  

By that way the html code from index.thml will going to render into base.html.  

## Using `url_for` method to include css file 

* Import `url_for`  
* Creat a folder css in static directory and copy the css code from base.html file into css/main.css 
* Delete the css code from base.thml and put the below code in base.html header field 

```html  
<link rel="stylesheet" href="{{ url_for('static', filename='css/main.css') }}">
```  

## Using sqlite database 

* Import sqlalchemy

```
from flask_sqlalchemy import SQLAlchemy  
```

* Initilize database with sqlalchemy

```
app.config['SQLALCHEMY_DATABASE_URL'] = 'sqlite:///test.db'
db = SQLAlchemy(app)
```

* Creating database model 

```  
class Todo(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.String(200), nullable=False)
    completed = db.Column(db.Integer, default=0)
    date_created = db.Column(db.DateTime, default=datetime.utcnow)

    # print 'Task: ID' on console whenever task is created
    def __repr__(self):
        return '<Task %r' % self.id
```  

* Creating the database file and models  

```  
flask shell 
db.create_all()
```   

Or open python terminal  

```  
from app import app, db
app.app_context().push()
db.create_all()
```   

## Creating Task 
 
```python      
@app.route('/', methods=['POST', 'GET'])
def index():
    if request.method == 'POST':
        task_content = request.form['content']
        new_task = Todo(content=task_content)
        try:
            db.session.add(new_task)
            db.session.commit()
            return redirect('/')
        except:
            return 'There was an issue adding your task'
    else:
        tasks = Todo.query.order_by(Todo.date_created).all()
        return render_template('index.html', tasks=tasks)
```    

index.html code for rendering and creating tasks 

```   
<div class="content">
    <h1 style="text-align: center">Task Master</h1>
    {% if tasks|length < 1 %}
    <h4 style="text-align: center">There are no tasks. Create one below!</h4>
    {% else %}
    <table>
        <tr>
            <th>Task</th>
            <th>Added</th>
            <th>Actions</th>
        </tr>
        {% for task in tasks %}
        <tr>
            <td>{{ task.content }}</td>
            <td>{{ task.date_created.date() }}</td>
            <td>
                <a href="/delete/{{task.id}}">Delete</a>
                <br>
                <a href="/update/{{task.id}}">Update</a>
            </td>
        </tr>
        {% endfor %}
    </table>
    {% endif %}

    <div class="form">
        <form method="POST" action="/">
            <input type="text" name="content" id="content">
            <input type="submit" value="Add Task">
        </form>
    </div>
</div>
```  

## Code for delete routine 

```
@app.route('/delete/<int:id>', methods=['GET'])
def delete(id):
    task_to_delete = Todo.query.get_or_404(id)
    try:
        db.session.delete(task_to_delete)
        db.session.commit()
        return redirect('/')
    except:
        return 'There was a problem deleting that task.!!'
```

## Updating Commnad  

Python flask code 

```
@app.route('/update/<int:id>', methods=['GET', "POST"])
def update(id):
    task = Todo.query.get_or_404(id)
    if request.method == 'POST':
        task.content = request.form['content']
        try:
            db.session.commit()
            return redirect('/')
        except:
            return 'There was an issue updating your task.!!'
    else:
        return render_template('update.html', task=task)
```

update.html code 

```  
{% extends 'base.html' %}

{% block head %}
<title>Update Task</title>
{% endblock %}

{% block body %}

<div class="content">
    <h1 style="text-align: center">Update Task</h1>
    <div class="form">
        <form method="POST" action="/update/{{task.id}}">
            <input type="text" name="content" id="content" value="{{task.content}}">
            <input type="submit" value="Update">
        </form>
    </div>
</div>

{% endblock %}
``` 

