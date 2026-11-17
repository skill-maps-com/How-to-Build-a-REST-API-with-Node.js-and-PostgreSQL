# Build a REST API with Node.js and PostgreSQL

A simple REST API built with **Node.js**, **Express** and **PostgreSQL**. It serves a list of users and supports the four basic operations: **GET, POST, PUT and DELETE**.

> This repository accompanies the lesson **"Build a REST API With Node.js and PostgreSQL (GET, POST, PUT, DELETE) – Step by Step"** published on **The Genius Blog**.

## Author

**Kindson Munonye** – software engineer and technical author covering machine learning, statistics, REST APIs, Python and software engineering. He publishes free tutorials on The Genius Blog and runs live classes on Alkademy.

All credit for the original lesson and code goes to him. If this repo helps you, subscribe to his YouTube channel to get notified about new lessons.

## What you will build

A REST API that:

1. Connects to a PostgreSQL database
2. Lists all users
3. Fetches a single user by id
4. Adds a new user
5. Updates a user's details
6. Deletes a user

## Table of contents

1. [Setup PostgreSQL / Node.js](#1-setup-postgresql--nodejs)
2. [Create the database connection](#2-create-the-database-connection)
3. [Create the server and client](#3-create-the-server-and-client)
4. [Get all users](#4-get-all-users)
5. [Get user by id](#5-get-user-by-id)
6. [Add a new user](#6-add-a-new-user)
7. [Update user details](#7-update-user-details)
8. [Delete a user](#8-delete-a-user)
9. [Next steps](#9-next-steps)

## 1. Setup PostgreSQL / Node.js

Both PostgreSQL and Node.js are free to download and install:

- [Download PostgreSQL](https://www.postgresql.org/download/)
- [Download Node.js](https://nodejs.org/)

Then install the dependencies:

```bash
npm install pg express body-parser --save
```

Create a database and a `users` table. The lesson does this in the video; an example schema matching the code below:

```sql
CREATE TABLE users (
  id        SERIAL PRIMARY KEY,
  firstname VARCHAR(100),
  lastname  VARCHAR(100),
  location  VARCHAR(100)
);
```

## 2. Create the database connection

Create a file called `connection.js`. It holds the connection data:

```js
const { Client } = require('pg');

const client = new Client({
  host: "localhost",
  user: "postgres",
  port: 5432,
  password: "your_password",
  database: "postgres"
});

module.exports = client;
```

## 3. Create the server and client

Create a second file, `api.js` (you can name it anything). It creates a server listening on port 3300 and connects the database client:

```js
const client = require('./connection.js');
const express = require('express');
const app = express();

app.listen(3300, () => {
  console.log("Server is now listening at port 3300");
});

client.connect();
```

**Add the body parser**, used to handle conversion to and from JSON:

```js
const bodyParser = require("body-parser");
app.use(bodyParser.json());
```

## 4. Get all users

For GET requests we use `app.get()`. It takes two parameters: the route (`/users`) and a callback. The callback receives the request and response; inside it we use the client to query the database and send the result back.

```js
app.get('/users', (req, res) => {
  client.query(`SELECT * FROM users`, (err, result) => {
    if (!err) {
      res.send(result.rows);
    }
  });
});
```

## 5. Get user by id

Note how the parameter is passed in the URL (`/users/:id`):

```js
app.get('/users/:id', (req, res) => {
  client.query(`SELECT * FROM users WHERE id = $1`, [req.params.id], (err, result) => {
    if (!err) {
      res.send(result.rows);
    }
  });
});
```

## 6. Add a new user

Post a new user with:

```js
app.post('/users', (req, res) => {
  const user = req.body;
  const insertQuery = `INSERT INTO users(id, firstname, lastname, location)
                       VALUES($1, $2, $3, $4)`;

  client.query(
    insertQuery,
    [user.id, user.firstname, user.lastname, user.location],
    (err, result) => {
      if (!err) {
        res.send('Insertion was successful');
      } else {
        console.log(err.message);
      }
    }
  );
});
```

## 7. Update user details

The update code follows the same pattern:

```js
app.put('/users/:id', (req, res) => {
  const user = req.body;
  const updateQuery = `UPDATE users
                       SET firstname = $1,
                           lastname  = $2,
                           location  = $3
                       WHERE id = $4`;

  client.query(
    updateQuery,
    [user.firstname, user.lastname, user.location, req.params.id],
    (err, result) => {
      if (!err) {
        res.send('Update was successful');
      } else {
        console.log(err.message);
      }
    }
  );
});
```

## 8. Delete a user

```js
app.delete('/users/:id', (req, res) => {
  const deleteQuery = `DELETE FROM users WHERE id = $1`;

  client.query(deleteQuery, [req.params.id], (err, result) => {
    if (!err) {
      res.send('Deletion was successful');
    } else {
      console.log(err.message);
    }
  });
});
```

> **Note:** the queries above use parameterized values (`$1`, `$2`, …) instead of interpolating request data into the SQL string. This protects against SQL injection.

## Running the API

```bash
node api.js
```

Then try the endpoints, for example:

```bash
curl http://localhost:3300/users
curl http://localhost:3300/users/1
curl -X POST http://localhost:3300/users \
  -H "Content-Type: application/json" \
  -d '{"id":1,"firstname":"Ada","lastname":"Lovelace","location":"London"}'
curl -X PUT http://localhost:3300/users/1 \
  -H "Content-Type: application/json" \
  -d '{"firstname":"Ada","lastname":"King","location":"Surrey"}'
curl -X DELETE http://localhost:3300/users/1
```

## 9. Next steps

- Repeat the same exercise with **MySQL**. The procedure is very similar, with minor differences in the connection setup.
- Build a **UI** on top of the API. The author's next lessons cover building the UI with Angular 11 and Bootstrap.

## Credits

Original lesson: *Build a REST API With Node.js and PostgreSQL (GET, POST, PUT, DELETE) – Step by Step* by **Kindson Munonye (Kindson The Genius)**, [The Genius Blog](https://www.thegeniusblog.com) — powered by Datarmatics.

Thanks and happy learning!
