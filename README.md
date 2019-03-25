# ![alt text](https://assets.breatheco.de/apis/img/images.php?blob&random&cat=icon&tags=breathecode,32) Academy API
  
Let's create am API that allows managing a simple academy system. All data must persist (Database).

Technologies: Flask, REST API, flask-SQLAlchemy, JSON.
  
## Data Models
  
The system needs two models to store information: *Studens* and *Courses*. 
+ A *Student* can be part of one *Course* at a time.
+ A *Course* can be related to several *Students* at the same time.

This is a Many-to-One (One-to-Many) relationship structure. All the fields must be not nullable (all required).

### Students
  
+ id (Primary Key)
+ Name
+ Last Name
+ Age
+ Course
  
### Courses
  
+ id (Primary Key)
+ Name


## Instructions
  
Create an API with these endpoints:
  
1. [POST] */courses/add* - Add a new *Course*.
2. [POST] */students/add* - Add a new *Student* (must have a *Course*).
3. [GET] */courses* - Show a list of *Courses* objects.
4. [GET] */courses/:int* - Show a specific *Course*, including the list of *Students*.
5. [GET] */students* - Show a list of *Students*.
6. [GET] */students/:int* - Show a specific *Student*, including the *Course* information (object).
7. [PUT] */students/:int* - Change a specific *Student* to a different *Course*

## Hints
  
All endpoints must return a **valid** JSON.
  
To set a relationship between two tables using Flask-SQLAlchemy we must first locate the table that represents the *one* in the *one-to-many* relationship:
  
One *Student* can be in ***one*** *Course*  **<-**
  
One *Course* can have ***one or more*** *Students*
  
  
Is in this Model (table) where we must set the new relational field: *Student.course_id*. We can also create a new field in the *Course* to make it easier to gather the list of the *Students* in that *Course*:
  
*models.py*
```python
from flask_sqlalchemy import SQLAlchemy
  
db = SQLAlchemy()
  
class Courses(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(80), unique=False, nullable=False)
    students = db.relationship("Students") #List of students
  
  
class Students(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    first_name = db.Column(db.String(80), unique=False, nullable=False)
    last_name = db.Column(db.String(80), unique=False, nullable=False)
    age = db.Column(db.Integer, unique=False, nullable=False)
    course_id = db.Column(db.Integer, db.ForeignKey('courses.id')) #Related to ONE course
    
```
  
Remember to run the migration in order to create/upgrade the SQLite Database.

### Serialization
  
Now we're facing a *serialization* problem if we try to ```jsonify()``` a *Course* with the *Students*' information or a *Student* with a *Course*'s information: not valid format for JSON.
  
This happens because of the default representation format of Models in flask-SQLAlchemy, which cannot be serialized/jsonify.
  
We now must create a class method that returns a serializable format:

```python
from flask_sqlalchemy import SQLAlchemy
  
db = SQLAlchemy()
  
class Courses(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(80), unique=False, nullable=False)
    students = db.relationship("Students")
  
    def __repr__(self):
        return 'Course: %s' % (self.name)
  
    def to_dict(self):
        students = []
        for s in self.students:
            students.append(s.to_dict())
        return { 
          "id": self.id, 
          "name": self.name, 
          "students": students 
        }
  
  
class Students(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    first_name = db.Column(db.String(80), unique=False, nullable=False)
    last_name = db.Column(db.String(80), unique=False, nullable=False)
    age = db.Column(db.Integer, unique=False, nullable=False)
    course_id = db.Column(db.Integer, db.ForeignKey('courses.id'))
  
    def __repr__(self):
        return 'Student: %s' % self.first_name
  
    def to_dict(self):
        return { 
          "id": self.id, 
          "first_name": self.first_name, 
          "last_name": self.last_name, 
          "course": self.course_id,
          "age": self.age 
        }
  
```
  
+ ```__repr__(self):``` : Is the default representation of the Model, a String (not serializable).
+ ```to_dict(self):``` : custom method to format the Model into a *dictionary* (serializable). Note how we can cross call ```to_dict()``` to get a serializable value that we can later ```jsonify()```.
  
  
Finally we can set an endpoint in ```app.py``` like this:
  
*app.py*
```python
...
  
  
@app.route('/courses')
def courses():
    courses = Course.query.all()
    response = []
    for c in courses:
        course = c.to_dict()
        response.append(course)
    
    return jsonify({"data": response})
  
  
...
  
  
```
