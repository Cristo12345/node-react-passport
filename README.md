We want a minimal example to demonstrate the use of the following:

* React
* Node
* Passport-local
* Postgres

We use Postgres here though it should be easy to do a MongoDB version.

# Postgres database setup

Some usual `psql` commands:
* `\du` to list all users
* `\l` to list all databases
* `\dt` to list all tables in current database

In case you haven't created a user:

```
sudo -u postgres psql

CREATE USER jchiu;
ALTER USER jchiu SUPERUSER CREATEDB;
```

Exit and login to postgres as the new user. Create a database just for this app.

```
psql template1
CREATE DATABASE db0000 WITH OWNER=jchiu;
```

Exit and login to the new database. Create our table. We add just the fields we need for this example. Some simple constraints are added.

```
psql db0000
CREATE TABLE table_users(
  id SERIAL PRIMARY KEY,
  salt VARCHAR(255) NOT NULL CHECK (char_length(username)>=4),
  username VARCHAR(255) UNIQUE NOT NULL CHECK (char_length(username)>=4),
  hash VARCHAR(1000) NOT NULL CHECK (char_length(hash)>=4));
```

Set a password for this database by modifying `/etc/postgresql/9.5/main/pg_hba.conf`. For example, add the following line to the end:

```
host db0000 jchiu 127.0.0.1/32 password
```

Then set the password.

```
psql db0000

ALTER ROLE jchiu UNENCRYPTED PASSWORD 'somepassword';
```

Try logging in:

```
psql -h localhost -d db0000
```

You will be prompted for a password. Try different possibilities to make sure it works.

This information should be consistent with what is specified in `models/pool.js`.

# Model

When a user is created, or when a password is updated, we create a new [salt](https://en.wikipedia.org/wiki/Salt_(cryptography)) for the user, which is really just some random bits.

`hash` is computed from hashing `salt+password`. The latter is string concatenation.


# Node setup

We assume you have installed Node and npm.

In case you don't have the following already installed on your machine, do:

```shell
sudo npm install express-generator supervisor -g
```

Let's put our Node code in the directory `backend`.

```shell
express backend
cd backend
npm install
```

In `package.json`, under `scripts.start`, change it from `node ./bin/www` to `supervisor ./bin/www`.

Try the app and see that it works by typing `npm start`. By default, for Express, the site would be available at `http://localhost:3000`.

Next, install all the packages we need.

```shell
npm install --save \
passport passport-local \
crypto underscore \
continuation-local-storage \
pg
```

We use Tape for unit testing.

```shell
sudo npm install tape -g
npm install tape --save-dev
```

Add `"test": "tape **/*_test.js"` to `package.json`.