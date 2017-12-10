# CREATE AND RUN A NODE.JS APP ON HEROKU
* register on heroku
* login with:
```		
heroku login
```
* download ready-baked code with:
```
git clone https://github.com/heroku/node-js-getting-started.git
cd node-js-getting-started
```
* or create your own repository with the app code, and make that a git repository
* Heroku recognizes an app as Node.js by the existence of a 'package.json' file in the root directory. The defauly app has already one. If you are creating your own app, then create one by:
```
npm init --yes
```
* deploy the app with:
```
heroku create
```
(Your app will get a random name. 
Also, a git remote called 'heroku' will be created and associated
with your local git repository.
Save the url of your app.)
* push your code to the 'heroku' git remote:
```
git push heroku master
```
* set one web dyno up:
```
heroku ps:scale web=1
```
(A heroku's dyno is one container that has all the code needed
for making your webapp running. When you put a dyno up, you will 
be able to visit your page. A free dyno has about 500h of free time
in which it can stay up. A free dyno automatically dies after a long
period of inactivity/no visits/no pinging. You can upgrade your profile
on heroku to be able to get multiple dynos on. In this way, you can scale your app.)
* visit your site with:
```
heroku open
```
(or visit the url of your app, if you saved it.)

# RUN THE APP LOCALLY
* make sure you have the 'package.json' file
* install the dependencies (specified in package.json) with:
```
npm install
```
* run the app locally:
```
heroku local web
```
* visit the app at port 5000:
```
http://localhost:5000
```
* if you want to stop the app:
```
CTRL + C
```

# PROPAGATE LOCAL CHANGES TO THE APPLICATION
* you can modifiy dependencies in package.json
* you can modify index.js (you can require new modules, add new routes, etc.)
* always test your app locally with:
```
npm install
heroku local
visit http://localhost:5000
```
* to finally deploy your updates, do:
```
git add .
git commit -m "updating changes remotely!"
git push heroku master
```
* check that everything works online:
```
heroku open
```

# ADDING A FREE POSTGRES DATABASE
* Add the database as an addon (more on addons later):
```
heroku addons:create heroku-postgresql:hobby-dev
```
* Check that the DATABASE_URL environment variable was set (more on env variables later)
```
heroku config
```
* Edit your package.json file to add the pg module to your dependencies:
```
"dependencies": {
    ...
    "pg": "6.x",
    ...
}
```
* Install the new module for the local app
```
npm install
```
* Now you can edit your index.js file to use the database through the module!
* Example of syntax and something you could do (in this example we add the route /db which will display all the rows of our database when visited):
```javascript
var pg = require('pg');

app.get('/db', function (request, response) {
  pg.connect(process.env.DATABASE_URL, function(err, client, done) {
    client.query('SELECT * FROM test_table', function(err, result) {
      done();
      if (err)
       { console.error(err); response.send("Error " + err); }
      else
       { response.render('pages/db', {results: result.rows} ); }
    });
  });
});
```
* Now install Postgres locally (if you haven't yet):
* export the DATABASE_URL environment variable
```
-- for Mac and Linux

export DATABASE_URL=postgres://$(whoami)
-- for Windows

set DATABASE_URL=postgres://$(whoami)
```
(this tells Postgres to connect locally to the database matching your user account name)
* install Postgres via your package manager. For example
```
sudo apt-get install postgresql
```
* check that the command works
```bash
$ psql
```
* If you have Mac or Windows, follow different instructions at: [Postgres Local Setup](https://devcenter.heroku.com/articles/heroku-postgresql#local-setup)
* Now you can connect to the remote database:
```
heroku pg:psql
```
* And from there you can interactively work with the SQL database. For example, you can create tables, insert rows, etc.
```
psql (9.5.2, server 9.6.2)
SSL connection (cipher: DHE-RSA-AES256-SHA, bits: 256)
Type "help" for help.
=> create table test_table (id integer, name text);
CREATE TABLE
=> insert into test_table values (1, 'hello database');
INSERT 0 1
=> \q
```
* Now if you access /db you will see the rows in your database!


# USEFUL STUFF
* **OPEN YOUR APP**
```
heroku open
```
(if the web dyno is off, it will automatically set up)
* **OPEN YOUR APP LOCALLY**
```
heroku local web
```
then visit
```
http://localhost:5000
```
To stop:
```
CTRL + C
```
* **LOGS**
```
heroku logs
```
(try to visit your site, you will see the logs changing.
CTRL+C to stop the stream of logs)
* **PROCFILE**

A file called 'Procfile' in your app's root directory.
It contains what commands should be executed when your app starts.
(For example, for a default node.js app it contains just:
web: node index.js
Which tells heroku to setup a web dyno that runs your app.
'web' is the name of the process.)
* **PROCESSES**
```
heroku ps
```
lets you see what are the running processes.
* **CHANGE DYNOS**
```
heroku ps:scale web=N
```
sets N dynos up. For example, if N is 0, then you shut everything off.
Try to type 'heroku ps' after, and you will see no processes running.
Also, try to visit your page, and you will get an error.
To set the page again up, type 'heroku ps:scale web=1'
* **ADDONS**

Add an addon (you may need to verify it):
``` 
heroku addons:create <name-of-the-addon>
```
List the addons of your app:
```
heroku addons
```
See the addon in action
```
heroku addons:open <name-of-the-addon>
```
Visit the [Addons Marketplace](https://elements.heroku.com/addons/categories/data-stores?_ga=2.67872276.484524067.1512910053-2080128028.1512414352) for a list of all the available addons.
* **FORMATION DYNOS, ONE-OFF DYNOS**
Formation dynos are the dynos declared in your Procfile.
They are run via
```
heroku ps:scale
```
One-off dynos are dynos that you can run independently, that behave like interactive processes attached to your terminal.
They are run via
```
heroku run <name-of-the-process>
```
* **ONE-OFF DYNOS EXAMPLES**
- Run bash process (from which you can interactively run bash commands)
```
heroku run bash
```
- Run node process (from which you can interactively run node.js commands)
```
heroku run node
```
- Execute a script with the given command (python example)
```
heroku run python myscript.py
```
- Execute a script with the given command (node example)
```
heroku run node myscript.js
```
- Execute a script by supplying a process name declared in Procfile:
  add in your procfile:
```
myworker: python myscript.py
```
  then run:
```
heroku run myworker 42
```
  notice that you can also give additional parameters to the script in this manner.
- Initialize or migrate database
```
rake db:migrate
```
```
node migrate.js migrate
```
* **CONFIG VARS**
You can setup configuration variables.
For your local version, change them in the file
```
.env
```
To set the config var on Heroku instead, do
```
heroku config:set NAME=2
heroku config
git add .
git commit -m "config var changed"
git push heroku master
```
(replace NAME with the name of the config var.
'heroku config' lets you just check the variables.)

# ADVANCED TOPICS
* [How Heroku Works](https://devcenter.heroku.com/articles/how-heroku-works) for a techical overview
* [Node.js Apps on Heroku](https://devcenter.heroku.com/articles/deploying-nodejs) to understand how to deploy an existing Node.js app on Heroku
* [Node.js category](https://devcenter.heroku.com/categories/nodejs) to learn more
