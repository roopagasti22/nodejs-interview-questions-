# Top 10 Node.js Interview Questions

This guide explains ten common Node.js interview questions with practical examples. The examples use modern JavaScript and assume Node.js is installed.

## 1. What is Node.js, and why is it used?

Node.js is an open-source JavaScript runtime. It lets you run JavaScript outside a web browser, for example on a server, in command-line tools, or in scripts. It is built on the V8 JavaScript engine and provides APIs for working with files, networks, processes, and other operating-system features.

Node.js is commonly used for:

- Web servers and REST APIs
- Real-time applications, such as chat and collaboration tools
- Command-line utilities and automation
- Services that make many I/O requests, such as database or network calls

Its event-driven, non-blocking I/O model can handle many concurrent connections without creating a separate thread for every request. This makes it particularly useful for I/O-heavy workloads. It does not mean that every task runs in parallel: JavaScript execution on a given event loop is normally single-threaded, and CPU-heavy work can delay other callbacks unless it is moved to worker threads or another process.

## 2. What is the difference between JavaScript and Node.js?

JavaScript is a programming language. Node.js is a runtime environment that executes JavaScript outside the browser.

| JavaScript in a browser | JavaScript in Node.js |
| --- | --- |
| Runs in a browser's JavaScript engine | Runs in Node.js, which uses the V8 engine |
| Can interact with browser APIs such as the DOM and `window` | Can interact with Node APIs such as `fs`, `path`, and `http` |
| Commonly used to build interactive web pages | Commonly used for servers, APIs, scripts, and tools |
| Restricted from direct access to the user's file system for security | Can access files and operating-system features, subject to permissions |

The language syntax and core concepts are shared, but the APIs available depend on the runtime. For example, `document.querySelector()` is a browser API and is not built into Node.js.

## 3. What is npm, and why do we use it?

npm is the package manager commonly used with Node.js. It includes a command-line tool and the npm package registry, where developers publish reusable packages.

npm helps you:

- Install and update project dependencies, such as Express
- Record dependencies and project metadata in `package.json`
- Lock exact dependency versions in `package-lock.json` for repeatable installs
- Run project scripts, such as tests or a development server
- Publish packages for other projects to use

For example:

```bash
npm init -y
npm install express
npm install --save-dev nodemon
```

`npm install express` adds Express as an application dependency. `npm install --save-dev nodemon` adds Nodemon as a development dependency. A script can be added to `package.json` and run by name:

```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  }
}
```

Run these with `npm start` and `npm run dev`, respectively. The lock file should generally be kept in version control so that collaborators and deployment environments install consistent dependency versions.

## 4. What is Express.js, and how does it work with Node.js?

Express.js is a lightweight web framework for Node.js. Node.js provides low-level networking APIs; Express builds on them to make common web-server tasks easier, including routing, handling HTTP requests and responses, and composing middleware.

Install it with `npm install express`. A minimal server looks like this:

```js
const express = require('express');

const app = express();
app.use(express.json());

app.get('/hello', (req, res) => {
  res.json({ message: 'Hello from Express!' });
});

app.listen(3000, () => {
  console.log('Server listening on port 3000');
});
```

Here, `app.get` registers a handler for `GET /hello`, `express.json()` parses JSON request bodies, and `app.listen` starts the HTTP server. Express does not replace Node.js; it is a library that runs on Node.js.

## 5. What is middleware in Express.js?

Middleware is a function that runs during the request-response cycle. It can inspect or modify the request and response, end the response, or pass control to the next middleware or route handler.

A middleware function typically receives `req`, `res`, and `next`:

```js
function requestLogger(req, res, next) {
  console.log(`${req.method} ${req.url}`);
  next();
}

app.use(requestLogger);
```

Calling `next()` passes the request to the next matching middleware. If middleware sends a response, it should not continue to the next handler. If it encounters an error, it can pass that error with `next(error)` so Express can handle it with error-handling middleware.

Common uses include logging, authentication, parsing request bodies, handling CORS, and validating input. Middleware order matters: it runs in the order it is registered, so parsing middleware should usually be registered before routes that read the parsed body.

## 6. What is the event loop in Node.js?

The event loop is the mechanism Node.js uses to schedule JavaScript callbacks and coordinate asynchronous work. When code starts an operation such as reading a file or making a network request, Node.js can initiate that work and continue executing other JavaScript instead of waiting for the operation to finish.

When the operation completes, its callback or promise continuation becomes eligible to run. The event loop processes queued work when the JavaScript call stack is clear. Node.js also has specific scheduling rules for timers, I/O callbacks, and promise microtasks; interview answers often describe these as event-loop phases.

For example:

```js
const fs = require('node:fs');

console.log('Before');
fs.readFile('notes.txt', 'utf8', (error, contents) => {
  if (error) {
    console.error(error);
    return;
  }
  console.log(contents);
});
console.log('After');
```

Usually, this prints `Before`, then `After`, and then the file contents because the file read completes asynchronously. The event loop does not make CPU-heavy JavaScript non-blocking: a long-running loop still prevents other JavaScript callbacks from running on that thread.

## 7. What is the difference between synchronous and asynchronous programming?

Synchronous code runs step by step. Each operation must finish before the next one begins. Asynchronous code lets a program start an operation and continue doing other work while it waits for the result.

Synchronous example:

```js
const fs = require('node:fs');

const contents = fs.readFileSync('notes.txt', 'utf8');
console.log(contents);
console.log('This runs after the file has been read');
```

Asynchronous example using promises and `async`/`await`:

```js
const fs = require('node:fs/promises');

async function readNotes() {
  try {
    const contents = await fs.readFile('notes.txt', 'utf8');
    console.log(contents);
  } catch (error) {
    console.error('Could not read notes:', error);
  }
}

readNotes();
console.log('This can run while the file read is in progress');
```

In Node.js servers, asynchronous I/O is often preferred because a request handler can wait for file, database, or network operations without blocking the JavaScript thread. `async`/`await` makes promise-based code read in a sequential style, but it does not make CPU-intensive work asynchronous by itself.

## 8. What is a REST API, and how do you create one using Node.js?

REST (Representational State Transfer) is an architectural style often used for HTTP APIs. A REST-style API exposes resources through URLs and uses HTTP methods to describe operations. For example, a `tasks` resource might use:

- `GET /tasks` to list tasks
- `GET /tasks/:id` to retrieve one task
- `POST /tasks` to create a task
- `PUT /tasks/:id` or `PATCH /tasks/:id` to update a task
- `DELETE /tasks/:id` to delete a task

Responses commonly use JSON and appropriate HTTP status codes, such as `200 OK`, `201 Created`, `400 Bad Request`, and `404 Not Found`.

This small Express example demonstrates listing and creating in-memory tasks:

```js
const express = require('express');

const app = express();
app.use(express.json());

const tasks = [];
let nextId = 1;

app.get('/tasks', (req, res) => {
  res.json(tasks);
});

app.post('/tasks', (req, res) => {
  const title = req.body.title;

  if (typeof title !== 'string' || title.trim() === '') {
    return res.status(400).json({ error: 'A non-empty title is required' });
  }

  const task = { id: nextId++, title: title.trim() };
  tasks.push(task);
  return res.status(201).json(task);
});

app.listen(3000, () => {
  console.log('API listening on port 3000');
});
```

This example stores data in memory, so tasks disappear when the server stops. A real application should use persistent storage, validate all relevant inputs, handle errors consistently, and apply authentication and authorization where required.

## 9. What is JWT authentication, and how does it work in Node.js?

JWT stands for JSON Web Token. It is a compact token format used to carry claims, such as a user's identifier. A signed JWT has three Base64URL-encoded parts: a header, a payload, and a signature. The signature lets the server detect changes to the token; **signing does not encrypt the payload**, so sensitive information should not be placed in it.

A typical bearer-token flow is:

1. The user sends credentials to a login endpoint over HTTPS.
2. The server verifies the credentials against a trusted user store.
3. If valid, the server signs a short-lived token and returns it.
4. The client sends the token in an `Authorization: Bearer <token>` header on later requests.
5. Middleware verifies the token's signature and relevant claims, such as expiration, before allowing access.

Using the `jsonwebtoken` package, a simplified example is:

```js
const jwt = require('jsonwebtoken');

function createAccessToken(user) {
  return jwt.sign(
    { sub: String(user.id) },
    process.env.JWT_SECRET,
    { expiresIn: '15m' }
  );
}

function requireAuth(req, res, next) {
  const authorization = req.get('authorization');
  const [scheme, token] = authorization?.split(' ') ?? [];

  if (scheme !== 'Bearer' || !token) {
    return res.status(401).json({ error: 'Authentication required' });
  }

  try {
    req.auth = jwt.verify(token, process.env.JWT_SECRET);
    return next();
  } catch {
    return res.status(401).json({ error: 'Invalid or expired token' });
  }
}
```

In an application, install the package with `npm install jsonwebtoken`, set `JWT_SECRET` to a strong secret in the deployment environment, and fail startup if it is missing. Do not hard-code secrets or accept a token merely because it can be decoded: verify its signature and expiration. Use HTTPS, validate login credentials securely, choose an appropriate token lifetime, and check authorization separately—an authenticated user is not automatically allowed to access every resource.

## 10. How do you connect Node.js to a MySQL database?

One common option is the `mysql2` package, which supports promises and connection pooling. A pool reuses database connections rather than opening a new connection for every query.

Install the driver with `npm install mysql2`, then configure connection details through environment variables:

```js
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
  host: process.env.DB_HOST,
  port: Number(process.env.DB_PORT ?? 3306),
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  waitForConnections: true,
  connectionLimit: 10
});

async function findUserById(id) {
  const [rows] = await pool.execute(
    'SELECT id, name, email FROM users WHERE id = ?',
    [id]
  );
  return rows[0] ?? null;
}

module.exports = { pool, findUserById };
```

The `?` placeholder is bound to the supplied value by the driver. Use parameterized queries like this instead of concatenating user input into SQL; concatenation can introduce SQL injection vulnerabilities. In a complete application, validate configuration at startup, handle database errors, close the pool gracefully when the process shuts down, and use migrations to manage schema changes.