# Flask API and Databases with Python Workshop
Hosted by Amine M. Boulouma, contact and questions: [amine.boulouma.com](https://amine.boulouma.com)
- [Learn Flask API and Databases with Python](https://www.youtube.com/watch?v=IrWAHTEtqgg&ab_channel=AmineM.Boulouma)
- [Source code](https://github.com/amboulouma/flask-api-workshop)

## Install flask
```pip install flask```


The flask.Flask object is a WSGI application, not a server. Flask uses Werkzeug's development server as a WSGI server when you call python -m flask run in your shell. It creates a new WSGI server and then passes your app as paremeter to werkzeug.serving.run_simple. Maybe you can try doing that manually:
Learn more about it: https://stackoverflow.com/questions/41831929/debug-flask-server-inside-jupyter-notebook/41833123

### Import WSGI Configuration for Jupyter Notebook

Not mandatory for python without Jupyter Notebook.


```python
from werkzeug.wrappers import Request, Response
from werkzeug.serving import run_simple
```

### Import SQLite for some database usage


```python
import sqlite3
```

### Import and configure Flask


```python
import flask
from flask import request, jsonify

app = flask.Flask(__name__)
app.config["DEBUG"] = True
```

### Root route


```python
@app.route('/', methods=['GET'])
def home():
    return """<h1>Book Library</h1>

            <p>This site is fictional book library.</p>"""
```

### Route to print text at specific route


```python
@app.route('/print-hello-world', methods=['GET'])
def print_hw():
    return """Hello World"""
```

Route: http://localhost:8000/print-hello-world

If on Jupyter Notebook we must run the code below to avoir any server running issues:


```python
#run_simple('localhost', 8000, app)
```

Flask.run() calls run_simple() internally, so there should be no difference here.

If not on Jupyter Notebook we can run just this:

```app.run()```

### Catalog Data


```python
books = [
    {
        "author": "Mark Clifton and Frank Riley",
        "first_sentence": "Just ahead, on Third Street, the massive facade of San Francisco's Southern Pacific depot loomed, half hidden in the swirling fog and January twilight.",
        "id": 1,
        "published": 1955,
        "title": "They'd Rather Be Right"
    },
    {
        "author": "Ray Bradbury",
        "first_sentence": "It was a pleasure to burn.",
        "id": 2,
        "published": 1954,
        "title": "Fahrenheit 451"
    },
    {
        "author": "Alfred Bester",
        "first_sentence": "Explosion!",
        "id": 3,
        "published": 1953,
        "title": "The Demolished Man"
    }
]
```


```python
@app.route('/books', methods=['GET'])
def books_api():
    return jsonify(books)
```

Route: http://127.0.0.1:5000/books

### Get a book by ID


```python
@app.route('/getbooks', methods=['GET'])
def getbooks_api():
    if 'id' in request.args:
        id = int(request.args['id'])
    else:
        return "Error: No id field provided. Please specify an id."

    results = []

    for book in books:
        if book['id'] == id:
            results.append(book)
            
    if results == []:
        return "Error: No such id in the book database."
    
    return jsonify(results)
```

Route: http://127.0.0.1:5000/getbooks?id=1

### Connecting API and a Database
Download database here: https://github.com/amboulouma/flask-api-workshop/raw/main/books.db

#### Function utility for database dictionary convestion


```python
def dict_factory(cursor, row):
    d = {}
    for idx, col in enumerate(cursor.description):
        d[col[0]] = row[idx]
    return d
```

### New route for getting the books


```python
@app.route('/getbooks_db', methods=['GET'])
def getbooks_db_api():
    query_parameters = request.args

    id = query_parameters.get('id')
    published = query_parameters.get('published')
    author = query_parameters.get('author')

    query = "SELECT * FROM books WHERE"
    to_filter = []

    if id:
        query += ' id=? AND'
        to_filter.append(id)
    if published:
        query += ' published=? AND'
        to_filter.append(published)
    if author:
        query += ' author=? AND'
        to_filter.append(author)
    if not (id or published or author):
        return page_not_found(404)

    query = query[:-4] + ';'

    conn = sqlite3.connect('books.db')
    conn.row_factory = dict_factory
    cur = conn.cursor()

    results = cur.execute(query, to_filter).fetchall()

    return jsonify(results)
```

Route: http://127.0.0.1:5000/getbooks_db?author=Connie+Willis&published=1999

### Update our get all books with the database


```python
@app.route('/books_db', methods=['GET'])
def books_db_api():
    conn = sqlite3.connect('books.db')
    conn.row_factory = dict_factory
    cur = conn.cursor()
    all_books = cur.execute('SELECT * FROM books;').fetchall()

    return jsonify(all_books)
```

Route: http://127.0.0.1:5000/books_db

### Add an error handler


```python
@app.errorhandler(404)
def page_not_found(e):
    return "<h1>404</h1><p>The resource could not be found.</p>", 404
```


```python
run_simple('localhost', 5000, app)
```

     * Running on http://localhost:5000/ (Press CTRL+C to quit)
    127.0.0.1 - - [01/Jun/2021 19:53:53] "[37mGET /books HTTP/1.1[0m" 200 -
    127.0.0.1 - - [01/Jun/2021 19:53:54] "[37mGET /books_db HTTP/1.1[0m" 200 -
    127.0.0.1 - - [01/Jun/2021 19:54:13] "[37mGET /getbooks HTTP/1.1[0m" 200 -
    127.0.0.1 - - [01/Jun/2021 19:54:27] "[37mGET /getbooks_db?author=Connie+Willis&published=1999 HTTP/1.1[0m" 200 -

