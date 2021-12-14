https://www.twilio.com/blog/graphql-api-python-flask-ariadne

Download/clone the repository then go to the folder of the project and create a virtual environment, activate it and install the packages with pip:
```
pip install flask ariadne flask-sqlalchemy
```

You will need to create the database, so, Fire up the terminal and start the python prompt by running the Python interpreter:
```
python
```

*Create the database table as follows:*
```
from main import db
db.create_all()
```

create your first to-do item and save it to the database:
```
from datetime import datetime
from api.models import Todo
today = datetime.today().date()
todo = Todo(description="Run a marathon", due_date=today, completed=False)
todo.to_dict()  # {'id': None, 'completed': False, 'description': 'Run a marathon', 'due_date': '2020-10-22'}
db.session.add(todo)
db.session.commit()
exit()  # To close the python prompt.
```

Now you can run in your terminal:
```
export FLASK_APP=main.py
flask run
```

go to:
http://127.0.0.1:5000/
Visit http://127.0.0.1:5000/graphql

Now you should be able to use the graphql:
*Example of queries runned at http://127.0.0.1:5000/graphql:*
```
mutation newTodo {
  createTodo(description:"Go to the dentist", dueDate:"24-10-2020") {
    success
    errors
    todo {
      id
      completed
      description
    }
  }
}

query findAllTodos {
  todos {
    success
    todos {
      description
      completed
      id
      dueDate
    }
  }
}

query findTodo {
  todo(todoId: "1") {
    success
    errors
    todo { id completed description dueDate }
  }
}

mutation markDone {
  markDone(todoId: "1") {
    success
    errors
    todo { id completed description dueDate }
  }
}

mutation updateDueDate {
  updateDueDate(todoId: "1", newDate: "25-10-2020") {
    success
    errors
  }
}

mutation deleteone {
  deleteTodo(todoId: "1") {
    success
    errors
  }
}
```
If you modify something in your code, you should run again: *flask run*

**From here, we have the procedure to create the project from the beginnig, also you can follow the instructions in https://www.twilio.com/blog/graphql-api-python-flask-ariadne**


# Create a directory called todo_api and navigate to it.
```
mkdir todo_api
cd todo_api
```

# Create the virtual environment:
```
python3 -m venv todo_api_env
```

# If you are using a Mac or Unix computer, activate the virtual environment as follows:
```
source todo_api_env/bin/activate
```

# install the packages flask ariadne flask-sqlalchemy
```
pip install flask ariadne flask-sqlalchemy
```

# INTRO:
We can define a Todo type using the SDL as follows:
```
type Todo {
    id: ID!
    description: String!
    completed: Boolean!
    dueDate: String!
}
```
The ! after a type indicates that the field is non-nullable, or in other words, that it must always have a value.

# Fetching data
When working with REST, we usually fetch data by making HTTP GET requests to various endpoints. GraphQL works a little differently. We have *a single endpoint*, from where the client can request all the data that it needs. The client does this by posting a query.

A query to get all the Todo items can look as follows:
```
type Query {
    todos: [Todo]!
}
```

# Create and modify data
We create, update and delete data in GraphQL using mutations. We write mutations similar to how we write queries but we use the keyword mutation. A mutation for creating a Todo would look as follows:
```
 type Mutation {
    createTodo(description: String!, dueDate: String!): Todo!
}
```


# create the Schema
Create a new file called schema.graphql with:
```
schema {
    query: Query
    mutation: Mutation
}

type Todo {
    id: ID!
    description: String!
    completed: Boolean!
    dueDate: String!
}

type TodoResult {
    success: Boolean!
    errors: [String]
    todo: Todo
}

type TodosResult {
    success: Boolean!
    errors: [String]
    todos: [Todo]
}

type Query {
    todos: TodosResult!
    todo(todoId: ID!): TodoResult!
}

type DeleteTodoResult {
    success: Boolean!
    errors: [String]
}

type Mutation {
    createTodo(description: String!, dueDate: String!): TodoResult!
    deleteTodo(todoId: ID!): DeleteTodoResult!
    markDone(todoId: String!): TodoResult!
    updateDueDate(todoId: String, newDate: String!): TodoResult!
}
```

We will build our API using Ariadne, which is a popular Python library for building GraphQL servers. Ariadne is a schema-first library, which means that the schema written in the SDL is the ultimate source of truth.

# Creating a Flask project
Now that we have already defined our schema, let’s implement it and put together our GraphQL API.

The code for our api will live inside a package called api. Inside todo_api, create a directory called api and inside it create a file called __init__.py with:
```
from flask import Flask

app = Flask(__name__)


@app.route('/')
def hello():
    return 'Hello!'
```

In the project root, create another file called main.py with:
```
from api import app #  , db
```

Now we need to tell Flask where to find the app application instance. We do that by setting the environment variable FLASK_APP to the name of the top-level Python file that has the app, which in our case is main.py. We do by running:
```
export FLASK_APP=main.py
```

Start the Flask server by running the following command:
```
flask run
```

go to:
http://127.0.0.1:5000/


# Add the database
Add the configuration to the api/__init__.py file:
```
import os
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config["SQLALCHEMY_DATABASE_URI"] = f"sqlite:///{os.getcwd()}/todo.db"
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False
db = SQLAlchemy(app)


@app.route('/')
def hello():
    return 'Hello!'
```

The SQLALCHEMY_DATABASE_URI setting tells Flask-SQLAlchemy where the database file is located. In our case, we will store it in the project directory with the name todo.db.

Setting SQLALCHEMY_TRACK_MODIFICATIONS to False disables tracking modifications of objects and sending signals to the application for every database change. It is a useful feature but can cause memory overhead, so it should only be used when necessary.


# Create the Model
Our database will have one table, called Todo.
Create a new file called models.py inside the api package and define the Todo model as shown below:

```
from main import db


class Todo(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    description = db.Column(db.String)
    completed = db.Column(db.Boolean, default=False)
    due_date = db.Column(db.Date)

    def to_dict(self):
        return {
            "id": self.id,
            "completed": self.completed,
            "description": self.description,
            "due_date": str(self.due_date.strftime('%d-%m-%Y'))
        }
```

The models file needs to be imported into the application. Edit your main.py file so that it looks as follows:
```
from api import app, db
from api import models
```

# Create some Todos
Fire up the terminal and start the python prompt by running the Python interpreter:
```
python
```

*Create the database table as follows:*
```
from main import db
db.create_all()
```

create your first to-do item and save it to the database:
```
from datetime import datetime
from api.models import Todo
today = datetime.today().date()
todo = Todo(description="Run a marathon", due_date=today, completed=False)
todo.to_dict()  # {'id': None, 'completed': False, 'description': 'Run a marathon', 'due_date': '2020-10-22'}
db.session.add(todo)
db.session.commit()
```

# Queries and Mutations
After creating a GraphQL schema, we need to create functions (resolvers) that return values for the different fields defined in it. Inside api, create two files called queries.py and mutations.py.

Let us write a resolver to fetch all the Todo items. Add the following to api/queries.py:
```
from .models import Todo


def resolve_todos(obj, info):
    try:
        todos = [todo.to_dict() for todo in Todo.query.all()]
        payload = {
            "success": True,
            "todos": todos
        }
    except Exception as error:
        payload = {
            "success": False,
            "errors": [str(error)]
        }
    return payload
```

# Binding a resolver
Once we write a resolver, we need to tell Ariadne which field it corresponds to from the schema, so we need to bind the resolve_todos function to the field todos in our GraphQL schema.

Add the following at the bottom of main.py:


```
from ariadne import load_schema_from_path, make_executable_schema, \
    graphql_sync, snake_case_fallback_resolvers, ObjectType
from ariadne.constants import PLAYGROUND_HTML
from flask import request, jsonify
from api.queries import resolve_todos

query = ObjectType("Query")

query.set_field("todos", resolve_todos)

type_defs = load_schema_from_path("schema.graphql")
schema = make_executable_schema(
    type_defs, query, snake_case_fallback_resolvers
)
```

# Exploring our API
Ariadne ships with GraphQL Playground, which is a graphical user interface that we can run to test our queries interactively. Let’s set that up so that we can begin testing our queries.

Add the following routes at the bottom of main.py:
```
@app.route("/graphql", methods=["GET"])
def graphql_playground():
    return PLAYGROUND_HTML, 200


@app.route("/graphql", methods=["POST"])
def graphql_server():
    data = request.get_json()

    success, result = graphql_sync(
        schema,
        data,
        context_value=request,
        debug=app.debug
    )

    status_code = 200 if success else 400
    return jsonify(result), status_code
```

Start the Flask server with:

```
flask run
```

Visit 127.0.0.1:5000/graphql and if everything is setup correctly.

# *Let’s write our first query. Paste the query below in the editor on the left side of the page:*

```
query fetchAllTodos {
  todos {
    success
    errors
    todos {
      description
      completed
      id
    }
  }
}
```
We have named our query fetchAllTodos and requested for the fields id, completed, dueDate and description from the query todos. Hit the play button to the right of the editor, you should see something like:
```
{
  "data": {
    "todos": {
      "errors": null,
      "success": true,
      "todos": [
        {
          "completed": false,
          "description": "Run a marathon",
          "id": "1"
        }
      ]
    }
  }
}
```

# Fetching a single item
To fetch a single Todo item, we will need to write a special kind of resolver; one that takes arguments. Example:

```
query fetchTodo {
  todo(todoId: "1") {
    success
    errors
    todo { id completed description dueDate }
  }
}
```

*when we execute it we get an obscure response that includes some error messages, among them one that reads “Cannot return null for non-nullable field Query.todo.” This happens because we haven’t written a resolver to resolve the todo field of the schema*

Let’s update api/queries.py to add our second resolver:
```
from ariadne import convert_kwargs_to_snake_case

...

@convert_kwargs_to_snake_case
def resolve_todo(obj, info, todo_id):
    try:
        todo = Todo.query.get(todo_id)
        payload = {
            "success": True,
            "todo": todo.to_dict()
        }

    except AttributeError:  # todo not found
        payload = {
            "success": False,
            "errors": [f"Todo item matching id {todo_id} not found"]
        }

    return payload
```

Next add the code to bind the resolver to main.py:

```
from api.queries import resolve_todos, resolve_todo

...

query.set_field("todos", resolve_todos)
query.set_field("todo", resolve_todo)
...

```
Now restart the Flask server and then run the query above once again and you should get back the Todo item matching the given id.

Note that we decorated our resolver with convert_kwargs_to_snake_case. This is because the argument is passed in as todoId on the query, but the corresponding argument on the resolver is named todo_id. We could define our resolver as def resolve_todo(obj, info, todoId), but to avoid having to mix snake case and camel case we use the convert_kwargs_to_snake_case decorator to convert the incoming arguments to snake case.


# Mutations
We write a mutation resolver in a similar way to how we have written the query resolvers above. The mutation resolver function takes in the obj and info arguments and any other arguments that are defined in the schema.

Let’s write our first mutation. As defined in the schema, our createTodo mutation takes two arguments: description and dueDate. Add the code below to api/mutations.py:

```
from datetime import datetime

from ariadne import convert_kwargs_to_snake_case

from api import db
from api.models import Todo


@convert_kwargs_to_snake_case
def resolve_create_todo(obj, info, description, due_date):
    try:
        due_date = datetime.strptime(due_date, '%d-%m-%Y').date()
        todo = Todo(
            description=description, due_date=due_date
        )
        db.session.add(todo)
        db.session.commit()
        payload = {
            "success": True,
            "todo": todo.to_dict()
        }
    except ValueError:  # date format errors
        payload = {
            "success": False,
            "errors": [f"Incorrect date format provided. Date should be in "
                       f"the format dd-mm-yyyy"]
        }

    return payload
```

If there was an error parsing the date string, the striptime function throws a ValueError and we return an error message prompting the user to provide a date in the format dd-mm-yyyy.

Let’s bind the mutation resolver. To do this we need to update main.py.

```
from api import app, db
from api import models
from ariadne import load_schema_from_path, make_executable_schema, \
    graphql_sync, snake_case_fallback_resolvers, ObjectType
from ariadne.constants import PLAYGROUND_HTML
from flask import request, jsonify
from api.queries import resolve_todos, resolve_todo
from api.mutations import resolve_create_todo

query = ObjectType("Query")

query.set_field("todos", resolve_todos)
query.set_field("todo", resolve_todo)

mutation = ObjectType("Mutation")
mutation.set_field("createTodo", resolve_create_todo)

type_defs = load_schema_from_path("schema.graphql")
schema = make_executable_schema(
    type_defs, query, mutation, snake_case_fallback_resolvers
)

...

```

Restart the Flask server and then try the following mutation in the playground:

```
mutation newTodo {
  createTodo(description:"Go to the dentist", dueDate:"24-10-2020") {
    success
    errors
    todo {
      id
      completed
      description
    }
  }
}
```

The server should return the result below:

```
{
  "data": {
    "createTodo": {
      "errors": null,
      "success": true,
      "todo": {
        "completed": false,
        "description": "Go to the dentist",
        "id": "2"
      }
    }
  }
}
```

Let’s now add a resolver for the markDone mutation. Add the code below to api/mutations.py:
```
@convert_kwargs_to_snake_case
def resolve_mark_done(obj, info, todo_id):
    try:
        todo = Todo.query.get(todo_id)
        todo.completed = True
        db.session.add(todo)
        db.session.commit()
        payload = {
            "success": True,
            "todo": todo.to_dict()
        }
    except AttributeError:  # todo not found
        payload = {
            "success": False,
            "errors":  [f"Todo matching id {todo_id} was not found"]
        }

    return payload
```

Here we accept a todo_id argument which we use to query for the particular Todo item, and then set its completed field to True.

To make the mutation available on the GraphQL server, let’s bind it as follows in main.py:

```
from api.mutations import resolve_create_todo, resolve_mark_done

...

mutation.set_field("createTodo", resolve_create_todo)
mutation.set_field("markDone", resolve_mark_done)

...

```

To test it, send a mutation to the server such as this one:

```
mutation markDone {
  markDone(todoId: "1") {
    success
    errors
    todo { id completed description dueDate }
  }
}
```

# Delete items.
Next we want to be able to delete items from the database. Go ahead and add one more mutation to api/mutations.py:

This resolver function accepts a todo_id, queries the database for our Todo item and then deletes it if it exists. This one returns a success value with the type boolean, denoting whether the requested Todo was deleted or not and an errors value which is a list of any errors that happened during execution.

Let’s go ahead and bind our resolver as follows in main.py:
```
from api.mutations import resolve_create_todo, resolve_mark_done, \
    resolve_delete_todo

...

mutation.set_field("markDone", resolve_mark_done)
mutation.set_field("deleteTodo", resolve_delete_todo)

...
```

To test it, send a mutation like the following to the server:

```
mutation {
  deleteTodo(todoId: "1") {
    success
    errors
  }
}
```

# Update one.
It’s possible our users will want to change the due date of an item. We will do that through the last of our mutations, which is called updateDueDate. Let’s add a resolver for this mutation in api/mutations.py:

Like all other resolvers, let’s go ahead and bind it as follows in main.py:

```
from api.mutations import resolve_create_todo, resolve_mark_done, \
    resolve_delete_todo, resolve_update_due_date

...

mutation.set_field("deleteTodo", resolve_delete_todo)
mutation.set_field("updateDueDate", resolve_update_due_date)

...
```

Test the mutation on the server with the following example:
```
mutation updateDueDate {
  updateDueDate(todoId: "2", newDate: "25-10-2020") {
    success
    errors
  }
}
```


*Example of queries runned at http://127.0.0.1:5000/graphql:*
```
query findAllTodos {
  todos {
    success
    todos {
      description
      completed
      id
      dueDate
    }
  }
}

query findTodo {
  todo(todoId: "1") {
    success
    errors
    todo { id completed description dueDate }
  }
}

mutation newTodo {
  createTodo(description:"Go to the dentist", dueDate:"24-10-2020") {
    success
    errors
    todo {
      id
      completed
      description
    }
  }
}

mutation markDone {
  markDone(todoId: "1") {
    success
    errors
    todo { id completed description dueDate }
  }
}

mutation deleteone {
  deleteTodo(todoId: "1") {
    success
    errors
  }
}

mutation updateDueDate {
  updateDueDate(todoId: "2", newDate: "25-10-2020") {
    success
    errors
  }
}
```

# graphql_python
# graphql_python
# graphql_python
# graphql_python
