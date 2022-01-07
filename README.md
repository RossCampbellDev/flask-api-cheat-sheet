# Using Python to make a REST API
## REST API
Representative State Transfer Application Programming Interface
The various parts of the URI that we request from the API are representative of the state of what we're looking at.  Change the URI, you are looking at a different state.

G - GET -> retrieve resources from an endpoint
P - POST -> write a new resource
D - DELETE
P - PUT -> overwrite an existing resource at the endpoint (idempotent)

## GET request
`import requests
import json`

we need both of these libraries so that we can make the web requests and retrieve responses, and to then parse the JSON resources more easily

`response = requests.get(\<URI\>)
json_data = response.json()`

## Serving an API
### Flask environment
`pip3 install flask
pip3 install flask-sqlalchemy
pip3 freeze > requirements.txt` - automatically makes our requirements file for dependencies

### Flask app
`from flask import flask
app = Flask(__name__)`

This sets up the flask application.  Next we start creating 'routes'

A 'route' is going to be an endpoint in our API.  If someone makes a request of that URI, the function below the route is called.

`@app.route('/')
def index():
	return "hello!"`

### Test this app
[in linux/mac CLI]
`export FLASK_APP=serverAPI.py
export FLASK_ENV=development
flask`

## Adding routes
```
@app.route('/drinks')
def get_drinks():
	# do something
	return result
```

## SQL Alchemy
```
To create our database
from flask_sqlalchemy import SQLAlchemy
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///data.db' # this sets up the connection to the database
db = SQLAlchemy(app) # pass in our flask app
```

Then build the database
`db.create_all() # this will also generate the db if it doesn't exist`

Create our database model

```
class Drink(db.Model):
	id = db.Column(db.Integer, primary_key=True)
	name = db.Column(db.String(50), unique=True, nullable=False)
	description = db.Column(db.String(200))`

	# override method
	def __repr__(self):
		return f"{self.name} - {self.description}" # formatted string
```
		
Now add some drinks

```
drink = Drink(name="cola", description="delicious drink")
db.session.add(drink)
db.session.commit()

Drink.query.all() # to retrieve all drinks from the db
```

now finishing our endpoint for 'getting all drinks':

```
@app.route('/drinks')
def get_drinks():
	drinks = Drink.query.all()
	result = []
	
	for drink in drinks:
		data = {'name': drink.name, 'desc': drink.description}
		result.append(data)
		
	return {"drinks": result}
```

since we want the output to be in a JSON-friendly format, we return a dictionary of dictionaries, in effect

## More specific queries
Now let's make it return a specific drink based on ID.  We need a new route:

```
@app.route('/drinks/<id>')
def get_drink(id):
	drink = Drink.query.get_or_404(id) # this will return 404 if it doesn't find a match for ID
	return {"name": drink.name, "description": drink.description}
```

## Adding a drink
```
@app.route('/drinks', methods=['POST'])
def add_drink(name, description):
	drink = Drink(name=request.json['name'], description=request.json['description'])
	db.session.add(drink)
	db.session.commit

	return f"{drink.id} added"
```

Alternatively, if this doesn't work (presumably something to do with python versions)

```
@app.route('/drinks', methods=['POST'])
def add_drink(name, description):
	resp = json.load(request.data.decode())
	drink = Drink(name=resp['name'], description=resp['description'])
	db.session.add(drink)
	db.session.commit

	return f"{drink.id} added"
```

## Deleting a drink
```
@app.route('/drinks', methods=['DELETE'])
def delete_drink(id):
	drink = Drink.query.get_or_404(id)
	if drink is None:
		return {"error": "didn't find that drink to remove"}
	db.session.delete(drink)
	db.session.commit

	return f"{drink.name} deleted"
```

#coding #API #python #requests
