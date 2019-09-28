# Creating a demo Flask Application with Postgresql

## Steps
	1. Install Postgresql & Create database
	2. Create Python virtual environment
	4. Install Flask
	5. Run database migration
	6. Finish Flask code
	7. Run the Flask application

### 1. Install Postgresql & Create Database
	- sudo apt-get install postgresql postgresql-contrib
	- To verify installation
		- sudo -u postgres psql -c "SELECT version();"
	- Create User and DB
		- sudo su - postgres -c "createuser devopsuser" 
		- sudo su - postgres -c "createdb devopsdb"
	- Grant privileges
		- sudo -u postgres psql
		- grant all privileges on database devopsdb to devopsuser;
		- alter user devopsuser with password 'devops';
	- Enable remote access to postgresql, edit 
		- sudo vi /etc/postgresql/10/main/postgresql.conf
		- Change the line "listen_addresses = 'localhost'" to "listen_addresses = '*'"
		- Save and exit the file
		- Restart postgresql
		- sudo /etc/init.d/postgresql restart

### 2. Create python virtual environment
	- pip3 install virtualenv
	- cd python-flask-pg
	- virtualenv env
	- Activate virtualenv with command 
		- source env/bin/activate

### Install Flask
	- Make sure virtualenv is enabled 
		- pip3 install Flask
	- Create file app.py and put the following code

```
from flask import Flask, request
import os

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello DevOps!"

@app.route("/name/<name>")
def get_book_name(name):
    return "name : {}".format(name)

@app.route("/details")
def get_book_details():
    author=request.args.get('age')
    published=request.args.get('company')
    return "Age : {}, Company: {}".format(author,published)

if __name__ == '__main__':
    app.run()
```

	- Save and run python3 app.py
	- Open localhost:500 in browser, you will see "Hello DevOps!"


### Create configurations
	- Create a file config.py and put the following code
	```
	import os
	basedir = os.path.abspath(os.path.dirname(__file__))

	class Config(object):
	    DEBUG = False
	    TESTING = False
	    CSRF_ENABLED = True
	    SECRET_KEY = 'this-really-needs-to-be-changed'
	    SQLALCHEMY_DATABASE_URI = os.environ['DATABASE_URL']


	class ProductionConfig(Config):
	    DEBUG = False


	class StagingConfig(Config):
	    DEVELOPMENT = True
	    DEBUG = True


	class DevelopmentConfig(Config):
	    DEVELOPMENT = True
	    DEBUG = True


	class TestingConfig(Config):
	    TESTING = True
    ```
    - Create a file .env and put the following in it
    	- export APP_SETTINGS="config.DevelopmentConfig"
		  export DATABASE_URL="postgresql://localhost/devopsdb"

### Database Migration
	- flask_sqlalchemy is needed for database migrations
	- pip3 install flask_sqlchemy
	- Modify app.py to include db variable created from sqlalchemy

	- Create a file models.py with the following content
	```
	from app import db

	class Employee(db.Model):
	    __tablename__ = 'employee'

	    id = db.Column(db.Integer, primary_key=True)
	    name = db.Column(db.String())
	    age = db.Column(db.String())
	    address = db.Column(db.String())

	    def __init__(self, name, age, address):
	        self.name = name
	        self.age = age
	        self.address = address

	    def __repr__(self):
	        return '<id {}>'.format(self.id)
	    
	    def serialize(self):
	        return {
	            'id': self.id, 
	            'name': self.name,
	            'age': self.age,
	            'address':self.address
	        }
        ```


        - Create a file manage.py with the following content
        ```
	    from flask_script import Manager
		from flask_migrate import Migrate, MigrateCommand

		from app import app, db

		migrate = Migrate(app, db)
		manager = Manager(app)

		manager.add_command('db', MigrateCommand)


		if __name__ == '__main__':
		    manager.run()
    	```
    	- Add below package to requirement.txt 
    		- flask_script
			- flask_migrate
			- psycopg2-binary
		- Now install these modules using below command
    		- pip3 install -r requirement.txt

    	- Migrate the Database now
    		- python3 manage.py db init
    		- python3 manage.py db migrate
    		- python3 manage.py db upgrade


# Create html page

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO"
        crossorigin="anonymous">
</head>

<body>
    <div class="container">

        <div class="container">
            <br>
            <br>

            <div class="row align-items-center justify-content-center">
                <h1>Add an employee</h1>
            </div>
            <br>

            <form method="POST">

                <label for="name">Employee Name</label>
                <div class="form-row">
                    <input class="form-control" type="text" placeholder="Name of Employee" id="name" name="name">
                </div>
                <br>
                <div class="form-row">
                    <label for="author">Age</label>
                    <input class="form-control" type="text" placeholder="Employee Age" id="age" name="age">
                </div>
                <br>
                <div class="form-row ">
                    <label for="published">Address</label>
                    <input class="form-control " type="text" placeholder="Address" id="address" name="address">
                </div>

                <br>
                <button type="submit " class="btn btn-primary " style="float:right ">Save</button>
            
            </form>
            <br><br>
        </div>
    </div>

    <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js " integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo "
        crossorigin="anonymous "></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js " integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49 "
        crossorigin="anonymous "></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js " integrity="sha384-ChfqqxuZUCnJSK3+MXmPNIyE6ZbWh2IMqE241rYiqJxyMiZ6OW/JmZQ5stwEULTy "
        crossorigin="anonymous "></script>
</body>

</html>
```



### Remove postgresql on ubuntu 18.04
	- sudo apt-get --purge remove postgresql postgresql-doc postgresql-common
