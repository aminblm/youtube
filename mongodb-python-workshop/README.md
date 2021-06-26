# MongoDB with Python Workshop
Hosted by Amin M. Boulouma, contact and questions: [amine.boulouma.com](https://amine.boulouma.com)
- [Learn MongoDB with Python](https://youtu.be/QgezT0KKu98)
- [Source code](https://github.com/amboulouma/mongodb-python-workshop)


## Prerequisites

PyMongo: https://pymongo.readthedocs.io/en/stable/installation.html

```pip install pymongo```

MongoDB: https://docs.mongodb.com/manual/installation/

```brew tap mongodb/brew```

```brew install mongodb-community@4.4```




## Start MongoDB

>$ mongod

## Making a Connection with MongoClient


The first step when working with PyMongo is to create a MongoClient to the running mongod instance. Doing so is easy:


```python
from pymongo import MongoClient

client = MongoClient()
```

Creating the client with port specification


```python
client = MongoClient('localhost', 27017)
```

### Getting a Database



```python
db = client.test_database
```

### Getting a Collection


```python
collection = db.test_collection
```

### Documents

Data in MongoDB is represented (and stored) using JSON-style documents. In PyMongo we use dictionaries to represent documents. As an example, the following dictionary might be used to represent a blog post:


```python
import datetime

post = {"author": "Amin",
        "text": "MongoDB with Python",
        "tags": ["mongodb", "python", "pymongo"],
        "date": datetime.datetime.utcnow()
       }
```

### Inserting a Document



```python
posts = db.posts
post_id = posts.insert_one(post).inserted_id
post_id
```




    ObjectId('60b7100120bfc60abf20bcbd')




```python
db.list_collection_names()
```




    ['posts']



### Getting a Single Document With find_one()


```python
import pprint

pprint.pprint(posts.find_one())
```

    {'_id': ObjectId('60b70b1a44ca0fefd174f09b'),
     'author': 'Amin',
     'date': datetime.datetime(2021, 6, 2, 4, 37, 46, 983000),
     'tags': ['mongodb', 'python', 'pymongo'],
     'text': 'MongoDB with Python'}



```python
pprint.pprint(posts.find_one({"text": "MongoDB with Python"}))
```

    {'_id': ObjectId('60b70b1a44ca0fefd174f09b'),
     'author': 'Amin',
     'date': datetime.datetime(2021, 6, 2, 4, 37, 46, 983000),
     'tags': ['mongodb', 'python', 'pymongo'],
     'text': 'MongoDB with Python'}



```python
posts.find_one({"author": "Somebody"})
```

### Querying By ObjectId



```python
post_id
```




    ObjectId('60b7100120bfc60abf20bcbd')




```python
pprint.pprint(posts.find_one({"_id": post_id}))
```

    {'_id': ObjectId('60b7100120bfc60abf20bcbd'),
     'author': 'Amin',
     'date': datetime.datetime(2021, 6, 2, 4, 58, 27, 840000),
     'tags': ['mongodb', 'python', 'pymongo'],
     'text': 'MongoDB with Python'}


Note that an ObjectId is not the same as its string representation:




```python
post_id == str(post_id)
```




    False




```python
str(post_id)
```




    '60b7100120bfc60abf20bcbd'




```python
posts.find_one({"_id": str(post_id)})
```

A common task in web applications is to get an ObjectId from the request URL and find the matching document. It’s necessary in this case to convert the ObjectId from a string before passing it to find_one:


```python
from bson.objectid import ObjectId

def get(post_id):
    document = client.db.collection.find_one({'_id': ObjectId(post_id)})
```

### Bulk Inserts



```python
new_posts = [{"author": "Amin",
              "text": "ElasticSearch With Python",
              "tags": ["elasticsearch", "python"],
              "date": datetime.datetime(2009, 11, 12, 11, 14)},
             {"author": "Amin",
              "title": "Flask with python",
              "text": "That is amazing!",
              "date": datetime.datetime(2009, 11, 10, 10, 45)}]

result = posts.insert_many(new_posts)
result.inserted_ids
```




    [ObjectId('60b7109120bfc60abf20bcbe'), ObjectId('60b7109120bfc60abf20bcbf')]



### Querying for More Than One Document


```python
for post in posts.find():
    pprint.pprint(post)

```

    {'_id': ObjectId('60b70b1a44ca0fefd174f09b'),
     'author': 'Amin',
     'date': datetime.datetime(2021, 6, 2, 4, 37, 46, 983000),
     'tags': ['mongodb', 'python', 'pymongo'],
     'text': 'MongoDB with Python'}
    {'_id': ObjectId('60b70c9c44ca0fefd174f09c'),
     'author': 'Amin',
     'date': datetime.datetime(2009, 11, 12, 11, 14),
     'tags': ['elasticsearch', 'python'],
     'text': 'ElasticSearch With Python'}
    {'_id': ObjectId('60b70c9c44ca0fefd174f09d'),
     'author': 'Amin',
     'date': datetime.datetime(2009, 11, 10, 10, 45),
     'text': 'That is amazing!',
     'title': 'Flask with python'}
    {'_id': ObjectId('60b7100120bfc60abf20bcbd'),
     'author': 'Amin',
     'date': datetime.datetime(2021, 6, 2, 4, 58, 27, 840000),
     'tags': ['mongodb', 'python', 'pymongo'],
     'text': 'MongoDB with Python'}
    {'_id': ObjectId('60b7109120bfc60abf20bcbe'),
     'author': 'Amin',
     'date': datetime.datetime(2009, 11, 12, 11, 14),
     'tags': ['elasticsearch', 'python'],
     'text': 'ElasticSearch With Python'}
    {'_id': ObjectId('60b7109120bfc60abf20bcbf'),
     'author': 'Amin',
     'date': datetime.datetime(2009, 11, 10, 10, 45),
     'text': 'That is amazing!',
     'title': 'Flask with python'}



```python
for post in posts.find({"text": "That is amazing!"}):
    pprint.pprint(post)
```

    {'_id': ObjectId('60b70c9c44ca0fefd174f09d'),
     'author': 'Amin',
     'date': datetime.datetime(2009, 11, 10, 10, 45),
     'text': 'That is amazing!',
     'title': 'Flask with python'}
    {'_id': ObjectId('60b7109120bfc60abf20bcbf'),
     'author': 'Amin',
     'date': datetime.datetime(2009, 11, 10, 10, 45),
     'text': 'That is amazing!',
     'title': 'Flask with python'}


### Counting


```python
posts.count_documents({})
```




    6




```python
posts.count_documents({"text": "ElasticSearch With Python"})
```




    2


