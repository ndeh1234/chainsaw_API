# chainsaw_API
from flask import Flask,g,jsonify, render_template
import requests
from peewee import *
from playhouse.shortcuts import model_to_dict



#Conguring database file name
DATABASE = 'chainsaw.db'

#Create a Flask web app
app = Flask(__name__)

#Connect to the database
database = SqliteDatabase(DATABASE)

#Model class, map fields to columns in the database
class Chainsaw(Model):
    name = CharField()
    country = CharField()
    catches = IntegerField()

    class Meta:
        database = database

# Create table for model
database.create_tables(Chainsaw)

# Connect to database before every request is handled
@app.before_request
def before_request():
    g.db = database
    g.db.connect()

# Close the DB connection after the request is handled
@app.after_request
def after_request(response):
    g.db.close()
    return response

#GET all the records
@app.route('/chainsaw_API/chainsaw')
def get_all():
    res = Chainsaw.select()

    #Convert each  record to a dict and convert the list of dicts to JSON
    return jsonify([model_to_dict(c) for c in res])

#POST to create a new record
@app.route('/chainsaw_API/chainsaw', methods=['POST'])
def add_new():
    with database.atomic():
        c =Chainsaw.create(**requests.form.to_dict())
        return jsonify(model_to_dict(c)), 201 # 201 means resource created.

#PATCH to modify an exixting record
@app.route('chainsaw_API/chainsaw/<catcher_id>', methods=['PATCH'])
def update_chainsaw(catcher_id):
    with database.atomic():
        Chainsaw.update(**requests.form.to_dict())\
        .where(Chainsaw.id == catcher_id\
        .execute())
        return 'ok', 200  # 200 status means ok, request successful

#DELETE to delete an exixting record
@app.route('/chainsaw_API/chainsaw/<catcher_id>', methods=['DELETE'])
def delete_chainsaw(catcher_id):
    with database.atomic():
        Chainsaw.delete().where(Chainsaw.id == catcher_id).execute()
        return 'ok', 200  # 200 status means ok, request successful

# Start the web app running. By default, on port 5000
if __name__ == '__main__':
    app.run()(debug=True)





