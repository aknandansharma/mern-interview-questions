# Node.js Interview Questions — 300 Q&A (Basic → Super Advanced)

A complete interview-prep pack for cracking Node.js rounds at any company — startup to FAANG-level. Every question has a simple-words explanation and a code snippet where it helps.

**Structure**
- Part 1: Basic (Q1–75)
- Part 2: Mid-Level (Q76–150)
- Part 3: Advanced (Q151–225)
- Part 4: Super Advanced (Q226–300)

---

# PART 1: BASIC (Q1–Q75)

### Q1. What is Node.js?
Node.js is a runtime that lets you run JavaScript outside the browser. It's built on Google Chrome's V8 engine, and it uses an event-driven, non-blocking model, which makes it great for building fast network applications like APIs, chat apps, and streaming services.

```js
// server.js — run with: node server.js
console.log("Node.js is just JS running on your machine, not in a browser");
```

### Q2. Why is Node.js single-threaded?
Node.js runs your JavaScript code on one main thread to keep things simple and avoid the complexity of thread locks/race conditions. Heavy I/O work (file reads, DB calls, network requests) is handed off to the system (via libuv's thread pool or OS async APIs) so the single thread never sits idle waiting.

```js
console.log("1: start");
setTimeout(() => console.log("2: timeout"), 0);
console.log("3: end");
// Output: 1, 3, 2 — single thread doesn't wait for the timer
```

### Q3. What is the Event Loop?
The event loop is the mechanism that lets Node.js do non-blocking I/O even though JS runs on one thread. It constantly checks: "Is the call stack empty? If yes, take the next task from the queue and run it." This is what allows thousands of concurrent connections without spawning thousands of threads.

```js
console.log("A");
setImmediate(() => console.log("B"));
Promise.resolve().then(() => console.log("C"));
console.log("D");
// A, D, C, B
```

### Q4. What is npm?
npm (Node Package Manager) is the default tool to install, share, and manage JavaScript packages. It comes bundled with Node.js and reads/writes `package.json` to track a project's dependencies.

```bash
npm init -y
npm install express
npm install --save-dev nodemon
```

### Q5. What is `package.json`?
It's the manifest file of a Node project — it stores the project name, version, dependencies, scripts, and metadata. Every tool (npm, bundlers, CI) reads this file to understand your project.

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "scripts": { "start": "node index.js" },
  "dependencies": { "express": "^4.19.2" }
}
```

### Q6. What is `package-lock.json`?
It locks the exact version of every dependency (and sub-dependency) that got installed, so every machine and every `npm install` produces an identical `node_modules` tree. Without it, a project could work on your machine and break on a teammate's due to minor version drift.

```bash
# Always commit package-lock.json to git
npm ci   # installs strictly from the lock file, faster & reproducible
```

### Q7. What are modules in Node.js?
A module is just a reusable block of code in its own file. Node wraps every file in a function so variables stay private to that file unless you explicitly export them.

```js
// math.js
function add(a, b) { return a + b; }
module.exports = { add };

// app.js
const { add } = require('./math');
console.log(add(2, 3)); // 5
```

### Q8. Difference between `require` and `import`?
`require` is CommonJS — synchronous, works at runtime, can be called conditionally anywhere in code. `import` is ES Modules (ESM) — must be at the top level, is asynchronous under the hood, and supports static analysis (tree-shaking). Node supports both, but ESM files need `.mjs` extension or `"type": "module"` in `package.json`.

```js
// CommonJS
const fs = require('fs');
// ESM
import fs from 'fs';
```

### Q9. What is `module.exports` vs `exports`?
`exports` is just a shorthand reference to `module.exports`. If you reassign `exports = something`, you break that link. Always use `module.exports` when exporting a single value (like a class or function).

```js
exports.a = 1;          // works
module.exports = { a: 1 }; // also works
exports = { a: 1 };     // BROKEN — doesn't actually export anything
```

### Q10. What are Node's global objects?
Objects available in every file without requiring them: `global`, `process`, `console`, `__dirname`, `__filename`, `Buffer`, `setTimeout`, `setInterval`, `require`, `module`. They're injected by Node's module wrapper, not truly "global" like `window` in browsers.

```js
console.log(__dirname);  // absolute path of current folder
console.log(__filename); // absolute path of current file
```

### Q11. What is the `process` object?
`process` gives information about, and control over, the current running Node instance — command-line args, environment variables, memory usage, and the ability to exit the program.

```js
console.log(process.argv);      // CLI arguments
console.log(process.env.NODE_ENV);
process.exit(1);                // exit with error code
```

### Q12. How do you read environment variables?
Through `process.env`. In real projects, people use a `.env` file with a package like `dotenv` to load values without hardcoding secrets into source code.

```js
require('dotenv').config();
console.log(process.env.DB_URL);
```

### Q13. What is the `fs` module?
`fs` (File System) lets you read, write, delete, and watch files. It offers sync, callback-based async, and Promise-based (`fs.promises` / `fs/promises`) APIs.

```js
const fs = require('fs');
fs.readFile('data.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});
```

### Q14. Sync vs Async fs methods — which should you use?
Sync methods (`readFileSync`) block the entire event loop until done — fine for startup scripts, bad for a running server. Async methods (`readFile`, or `fs.promises.readFile`) don't block, so the server can keep handling other requests while waiting on disk I/O.

```js
// BAD in a live server:
const data = fs.readFileSync('big.txt'); // blocks everyone

// GOOD:
const data = await fs.promises.readFile('big.txt');
```

### Q15. What is the `http` module?
It's Node's built-in module to create web servers and make HTTP requests, without any external framework like Express.

```js
const http = require('http');
const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello World');
});
server.listen(3000);
```

### Q16. What is middleware in the context of Node/Express?
Middleware is a function that sits between the incoming request and the final response — it can inspect, modify, log, or reject the request before passing control to the next function using `next()`.

```js
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next(); // pass control forward
});
```

### Q17. What is callback hell and how do you avoid it?
Callback hell is deeply nested callbacks that become hard to read and maintain — often shaped like a pyramid. You avoid it using Promises or `async/await`, which flatten the structure.

```js
// Callback hell
getUser(id, (user) => {
  getPosts(user, (posts) => {
    getComments(posts, (comments) => {
      console.log(comments);
    });
  });
});

// Fixed with async/await
const user = await getUser(id);
const posts = await getPosts(user);
const comments = await getComments(posts);
```

### Q18. What is a Promise?
A Promise is an object representing the eventual result (success or failure) of an async operation. It has 3 states: pending, fulfilled, rejected.

```js
const p = new Promise((resolve, reject) => {
  setTimeout(() => resolve("done"), 1000);
});
p.then(result => console.log(result));
```

### Q19. What is `async/await`?
Syntactic sugar over Promises that lets asynchronous code read like synchronous code. An `async` function always returns a Promise, and `await` pauses execution until that Promise settles.

```js
async function getData() {
  try {
    const res = await fetch('https://api.example.com');
    const data = await res.json();
    return data;
  } catch (err) {
    console.error(err);
  }
}
```

### Q20. What is the difference between `setTimeout`, `setImmediate`, and `process.nextTick`?
`process.nextTick` runs before anything else — even before Promises — right after the current operation finishes. `setTimeout(fn, 0)` waits for the timers phase of the event loop. `setImmediate` runs in the "check" phase, right after I/O callbacks. Order can vary depending on context (main script vs inside I/O).

```js
process.nextTick(() => console.log('nextTick'));
setImmediate(() => console.log('immediate'));
setTimeout(() => console.log('timeout'), 0);
console.log('main');
// main, nextTick, timeout, immediate (typical order in main module)
```

### Q21. What is JSON and how do you work with it in Node?
JSON (JavaScript Object Notation) is a lightweight text format for data exchange. Node has built-in `JSON.stringify()` (object → string) and `JSON.parse()` (string → object).

```js
const obj = { name: "Aknandan", role: "SDE" };
const str = JSON.stringify(obj);
const back = JSON.parse(str);
```

### Q22. What is the `path` module used for?
It helps you build and manipulate file paths safely across operating systems (Windows uses `\`, Linux/Mac use `/`), instead of manually concatenating strings.

```js
const path = require('path');
console.log(path.join(__dirname, 'files', 'data.txt'));
console.log(path.extname('report.pdf')); // .pdf
```

### Q23. What is the `os` module?
It gives info about the operating system the code is running on — CPU cores, free memory, platform, home directory — useful for scaling decisions like how many worker processes to spawn.

```js
const os = require('os');
console.log(os.cpus().length); // number of CPU cores
console.log(os.freemem());
```

### Q24. What is REPL in Node.js?
REPL stands for Read-Eval-Print-Loop — it's the interactive shell you get when you type `node` in the terminal without a filename. It reads input, evaluates it, prints the result, and loops.

```bash
$ node
> 2 + 2
4
> const x = 10
undefined
> x * 2
20
```

### Q25. What is an EventEmitter?
It's a core Node class that implements the publish-subscribe (observer) pattern — you can emit named events and register listener functions for them. Most core Node objects (streams, servers) extend EventEmitter internally.

```js
const EventEmitter = require('events');
const emitter = new EventEmitter();
emitter.on('greet', (name) => console.log(`Hello, ${name}`));
emitter.emit('greet', 'World');
```

### Q26. What is a Buffer in Node.js?
A Buffer is a chunk of raw binary data stored outside V8's normal memory heap — used when handling things like file contents, TCP streams, or images, where data isn't plain text/UTF-8.

```js
const buf = Buffer.from('Hello');
console.log(buf);          // <Buffer 48 65 6c 6c 6f>
console.log(buf.toString()); // 'Hello'
```

### Q27. What is the difference between `null` and `undefined` in Node/JS?
`undefined` means a variable was declared but never assigned a value (or a function returned nothing). `null` is an intentional "no value" assigned by the developer. Node functions often use `null` for "no error" in error-first callbacks.

```js
let a;
console.log(a); // undefined
let b = null;
console.log(b); // null (explicitly empty)
```

### Q28. What is an error-first callback?
A Node.js convention where the first argument of a callback is reserved for an error (or `null` if none), and the second argument is the actual result. It became the de facto standard before Promises were widespread.

```js
fs.readFile('file.txt', (err, data) => {
  if (err) return console.error(err);
  console.log(data);
});
```

### Q29. How do you handle uncaught exceptions in Node?
Listen on `process.on('uncaughtException', ...)`, but it's mainly used to log and then gracefully shut down — you should NOT try to "resume" the app after one, since the process may be in an inconsistent state.

```js
process.on('uncaughtException', (err) => {
  console.error('Uncaught:', err);
  process.exit(1); // restart via process manager (PM2, etc.)
});
```

### Q30. What is the difference between `unhandledRejection` and `uncaughtException`?
`uncaughtException` fires for synchronous errors that were never caught by a try/catch. `unhandledRejection` fires when a Promise rejects and nobody attached a `.catch()` to handle it.

```js
process.on('unhandledRejection', (reason) => {
  console.error('Unhandled rejection:', reason);
});
Promise.reject(new Error('oops')); // triggers the handler above
```

### Q31. What is `__dirname` vs `process.cwd()`?
`__dirname` is the folder where the currently executing file lives — fixed at file location. `process.cwd()` is the folder from which the Node process was launched (the terminal's current directory) — it can differ if you run the script from elsewhere.

```js
console.log(__dirname);    // e.g. /project/src
console.log(process.cwd()); // e.g. /project (if you ran `node src/app.js` from /project)
```

### Q32. What is npx?
`npx` runs a package's binary without permanently installing it globally — it downloads it temporarily (or uses a local install) and executes it. Handy for one-off commands like scaffolding tools.

```bash
npx create-react-app my-app
npx nodemon app.js
```

### Q33. What are semantic versioning (semver) symbols in package.json?
`^1.2.3` allows minor and patch updates (up to `<2.0.0`). `~1.2.3` allows only patch updates (up to `<1.3.0`). An exact version like `1.2.3` locks it completely. This controls how much a package can drift on `npm install`/`update`.

```json
{ "dependencies": { "lodash": "^4.17.21" } }
```

### Q34. What is the difference between `dependencies` and `devDependencies`?
`dependencies` are needed for the app to actually run in production (e.g. express). `devDependencies` are only needed during development/testing (e.g. nodemon, jest, eslint) and are typically not installed in a production build.

```bash
npm install express --save
npm install jest --save-dev
```

### Q35. What is CORS and why does it matter in Node APIs?
CORS (Cross-Origin Resource Sharing) is a browser security rule that blocks a webpage from calling an API on a different domain unless the server explicitly allows it via response headers. Node/Express APIs commonly use the `cors` package to control this.

```js
const cors = require('cors');
app.use(cors({ origin: 'https://myfrontend.com' }));
```

### Q36. How does Node.js handle multiple concurrent requests with only one thread?
It doesn't block on I/O. When a request needs to read a file or query a database, Node delegates that work (to libuv/OS/thread pool) and immediately moves to the next request. Once the I/O finishes, its callback is queued and executed when the main thread is free.

```js
// Both requests get handled "at once" even on 1 thread,
// because neither one blocks while waiting for the DB.
app.get('/a', async (req, res) => res.json(await db.query('...')));
app.get('/b', async (req, res) => res.json(await db.query('...')));
```

### Q37. What is a Stream in Node.js?
A Stream is a way to read or write data piece by piece (in chunks) instead of loading everything into memory at once — critical for handling large files or live data like video.

```js
const fs = require('fs');
const readStream = fs.createReadStream('bigfile.txt');
readStream.on('data', (chunk) => console.log('chunk received:', chunk.length));
```

### Q38. What are the 4 types of Streams?
Readable (you read data from it, e.g. `fs.createReadStream`), Writable (you write data to it, e.g. `fs.createWriteStream`), Duplex (both readable and writable, e.g. a TCP socket), and Transform (a duplex stream that modifies data as it passes through, e.g. `zlib.createGzip()`).

```js
const zlib = require('zlib');
fs.createReadStream('file.txt')
  .pipe(zlib.createGzip())      // Transform stream
  .pipe(fs.createWriteStream('file.txt.gz'));
```

### Q39. What does `.pipe()` do?
It connects a readable stream's output directly to a writable stream's input, automatically managing the flow of data (including backpressure), so you don't have to manually listen for `data` events and call `.write()`.

```js
fs.createReadStream('input.txt').pipe(fs.createWriteStream('output.txt'));
```

### Q40. What is middleware order in Express and why does it matter?
Express runs middleware in the exact order you register it with `app.use()`. If you put an auth-check middleware after your route handler, it's useless — the route already ran. Order determines what each request "passes through" before reaching its final handler.

```js
app.use(logger);      // runs first
app.use(authenticate); // runs second
app.get('/profile', getProfile); // runs last
```

### Q41. What is the difference between `GET`, `POST`, `PUT`, `PATCH`, `DELETE`?
`GET` retrieves data (no body, safe/idempotent). `POST` creates a new resource. `PUT` replaces a resource entirely. `PATCH` updates part of a resource. `DELETE` removes a resource. REST APIs map these to CRUD operations.

```js
app.get('/users/:id', getUser);
app.post('/users', createUser);
app.put('/users/:id', replaceUser);
app.patch('/users/:id', updateUserField);
app.delete('/users/:id', deleteUser);
```

### Q42. What are HTTP status codes and common ones to know?
They're 3-digit codes a server sends to indicate the result of a request: 2xx = success (200 OK, 201 Created), 3xx = redirection (301, 304), 4xx = client error (400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found), 5xx = server error (500 Internal Server Error, 503 Service Unavailable).

```js
res.status(404).json({ message: 'User not found' });
res.status(201).json({ message: 'Created' });
```

### Q43. What is `express.json()` middleware for?
It parses incoming requests with a JSON body (`Content-Type: application/json`) and populates `req.body` with the parsed object. Without it, `req.body` would be `undefined`.

```js
app.use(express.json());
app.post('/data', (req, res) => {
  console.log(req.body); // now available as a JS object
});
```

### Q44. What is routing in Express?
Routing determines how an app responds to a client request to a specific URL path and HTTP method. Express lets you define routes on `app` directly or organize them into separate `Router` modules.

```js
const router = express.Router();
router.get('/', (req, res) => res.send('list users'));
app.use('/users', router);
```

### Q45. What is a route parameter vs a query parameter?
Route params are part of the URL path itself, marked with `:` (e.g. `/users/:id`), used for identifying a specific resource. Query params come after `?` (e.g. `/users?sort=asc`), used for optional filters/options.

```js
app.get('/users/:id', (req, res) => {
  console.log(req.params.id);  // route param
  console.log(req.query.sort); // query param, e.g. /users/5?sort=asc
});
```

### Q46. How do you handle errors centrally in Express?
By defining an error-handling middleware with 4 parameters `(err, req, res, next)`, placed after all your routes. Any `next(err)` call, or a thrown error inside an async route wrapped properly, lands here.

```js
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ message: 'Something broke!' });
});
```

### Q47. What is `nodemon` and why use it?
It's a dev tool that watches your files and automatically restarts the Node server whenever it detects a code change — saving you from manually stopping/restarting during development.

```bash
npm install -D nodemon
npx nodemon server.js
```

### Q48. How do you connect Node.js to MongoDB?
Typically via the `mongodb` native driver, or more commonly `mongoose`, an ODM (Object Data Modeling) library that adds schemas, validation, and easier query building on top of MongoDB.

```js
const mongoose = require('mongoose');
mongoose.connect('mongodb://localhost:27017/mydb')
  .then(() => console.log('DB connected'));
```

### Q49. What is a Mongoose Schema/Model?
A Schema defines the shape and rules of documents in a MongoDB collection (field types, required fields, defaults). A Model is a constructor compiled from that schema, used to create, read, update, and delete documents.

```js
const userSchema = new mongoose.Schema({
  name: { type: String, required: true },
  age: { type: Number, default: 18 }
});
const User = mongoose.model('User', userSchema);
```

### Q50. What is the difference between `SQL` and `NoSQL` databases (as relevant to Node choices)?
SQL databases (MySQL, PostgreSQL) store data in structured tables with fixed schemas and relations, great for complex queries/transactions. NoSQL (MongoDB) stores flexible, schema-less documents, great for fast iteration and horizontally scaling unstructured data. Node works well with both — the choice depends on data shape and consistency needs, not the language.

```js
// SQL (with a driver like 'pg')
await client.query('SELECT * FROM users WHERE id = $1', [id]);
// NoSQL (Mongoose)
await User.findById(id);
```

### Q51. What is JWT and how is it used for authentication?
JWT (JSON Web Token) is a compact, signed token containing claims (like user id) that a server issues after login. The client sends it back on future requests (usually in the `Authorization` header), and the server verifies its signature instead of hitting the database every time to check who's logged in.

```js
const jwt = require('jsonwebtoken');
const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET, { expiresIn: '1h' });
const decoded = jwt.verify(token, process.env.JWT_SECRET);
```

### Q52. What is the difference between authentication and authorization?
Authentication answers "who are you?" (login, verifying identity). Authorization answers "what are you allowed to do?" (checking roles/permissions after identity is confirmed). One always happens before the other.

```js
// Authentication: verifying the JWT identifies the user
// Authorization: checking role after identity is known
if (req.user.role !== 'admin') return res.status(403).send('Forbidden');
```

### Q53. How do you hash passwords in Node.js?
Never store plain-text passwords. Use a slow, salted hashing algorithm like bcrypt, which makes brute-forcing computationally expensive.

```js
const bcrypt = require('bcrypt');
const hash = await bcrypt.hash(plainPassword, 10); // 10 = salt rounds
const isMatch = await bcrypt.compare(plainPassword, hash);
```

### Q54. What are cookies and sessions in a Node app?
A cookie is a small piece of data stored in the browser and sent with every request to the same domain. A session stores user state on the server (or in a store like Redis), with only a session ID kept in the cookie — used for traditional server-rendered login flows (as opposed to stateless JWTs).

```js
const session = require('express-session');
app.use(session({ secret: 'keyboard cat', resave: false, saveUninitialized: false }));
```

### Q55. What is CSRF and how do you prevent it in Node apps?
CSRF (Cross-Site Request Forgery) tricks a logged-in user's browser into sending an unwanted request to your server (since cookies auto-attach). Prevention includes CSRF tokens, `SameSite` cookie attributes, and checking the request's origin/referrer header.

```js
res.cookie('token', value, { sameSite: 'strict', httpOnly: true });
```

### Q56. What is `helmet` in Express?
A middleware package that sets a bunch of security-related HTTP headers automatically (like disabling `X-Powered-By`, enabling `Content-Security-Policy`) to protect against common web vulnerabilities.

```js
const helmet = require('helmet');
app.use(helmet());
```

### Q57. What is rate limiting and why is it important?
It restricts how many requests a client can make in a time window, to prevent abuse, brute-force attacks, and server overload.

```js
const rateLimit = require('express-rate-limit');
app.use(rateLimit({ windowMs: 15 * 60 * 1000, max: 100 })); // 100 req / 15 min
```

### Q58. What is input validation and why does it matter?
It's checking incoming data (body, params, query) against expected rules (type, length, format) before processing it, to prevent bad data, crashes, and security issues like injection attacks.

```js
const { body, validationResult } = require('express-validator');
app.post('/signup',
  body('email').isEmail(),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
  }
);
```

### Q59. What is logging and why not just use `console.log` in production?
`console.log` is synchronous, unstructured, and gives no log levels or persistence. Production apps use loggers like `winston` or `pino` for structured (JSON) logs, log levels (info/warn/error), and writing to files or external log services.

```js
const winston = require('winston');
const logger = winston.createLogger({
  transports: [new winston.transports.Console()]
});
logger.info('Server started');
```

### Q60. What is `.env` and why shouldn't you commit it to git?
It's a file storing environment-specific config (API keys, DB URLs, secrets) loaded via `dotenv`. It shouldn't be committed because it contains sensitive credentials — instead you add it to `.gitignore` and provide an `.env.example` template.

```
# .env
DB_URL=mongodb://localhost:27017/mydb
JWT_SECRET=supersecret
```

### Q61. What is API versioning and why is it needed?
It's the practice of tagging your API (e.g. `/api/v1/users`) so you can release breaking changes in a new version (`/api/v2/...`) without breaking existing clients still using the old one.

```js
app.use('/api/v1/users', v1UserRouter);
app.use('/api/v2/users', v2UserRouter);
```

### Q62. What is pagination and how do you implement it in Node/MongoDB?
Pagination breaks a large result set into smaller pages, avoiding sending (or querying) massive amounts of data at once. Common approaches: offset-based (`skip`/`limit`) or cursor-based (using an ID as a marker).

```js
const page = parseInt(req.query.page) || 1;
const limit = 10;
const users = await User.find().skip((page - 1) * limit).limit(limit);
```

### Q63. What is the difference between `PUT` and `PATCH` in practice?
`PUT` expects the full resource representation and replaces it entirely (missing fields may get wiped/reset). `PATCH` sends only the fields that changed, updating them in place.

```js
// PUT — full replace
app.put('/users/:id', async (req, res) => {
  await User.findByIdAndUpdate(req.params.id, req.body, { overwrite: true });
});
// PATCH — partial update
app.patch('/users/:id', async (req, res) => {
  await User.findByIdAndUpdate(req.params.id, { $set: req.body });
});
```

### Q64. What is CommonJS?
It's the module system Node.js originally used — `require()` to import, `module.exports` to export. It's synchronous and loads the whole module at once, unlike ESM which supports async loading and static analysis.

```js
// CommonJS
const express = require('express');
module.exports = router;
```

### Q65. What is `Object.freeze()` used for in Node config?
It makes an object immutable — attempts to modify it silently fail (or throw in strict mode). Commonly used to lock configuration objects so they can't be accidentally changed at runtime.

```js
const config = Object.freeze({ port: 3000, env: 'production' });
config.port = 4000; // silently ignored (or throws in strict mode)
console.log(config.port); // still 3000
```

### Q66. What is the difference between `==` and `===` (relevant in Node too)?
`==` compares values after type coercion (converts types to match). `===` compares both value and type with no coercion — always the safer, more predictable choice.

```js
console.log(1 == '1');  // true (coerced)
console.log(1 === '1'); // false (different types)
```

### Q67. What is destructuring and how is it used in Node code?
It's a JS syntax to unpack values from arrays or properties from objects into distinct variables, making code cleaner — very common when pulling specific fields from `req.body` or config objects.

```js
const { name, email } = req.body;
const [first, second] = ['a', 'b'];
```

### Q68. What are template literals?
Strings wrapped in backticks (`` ` ``) that allow embedded expressions using `${}` and multi-line strings, replacing clunky string concatenation.

```js
const name = 'Aknandan';
console.log(`Hello, ${name}! Today is ${new Date().toDateString()}`);
```

### Q69. What is the spread operator (`...`) used for?
It expands an iterable (array/object) into individual elements — useful for copying arrays/objects, merging them, or passing array items as function arguments.

```js
const arr1 = [1, 2];
const arr2 = [...arr1, 3, 4]; // [1, 2, 3, 4]
const merged = { ...obj1, ...obj2 };
```

### Q70. What is a closure, and where is it used in Node apps?
A closure is a function that "remembers" variables from its outer scope even after that outer function has finished running. Used heavily in Node for things like middleware factories and private state.

```js
function rateLimiter(max) {
  let count = 0;
  return function () { // closure over `count` and `max`
    count++;
    return count <= max;
  };
}
const limiter = rateLimiter(3);
```

### Q71. What is the difference between `let`, `const`, and `var`?
`var` is function-scoped and gets hoisted (usable before declaration, as `undefined`). `let` and `const` are block-scoped and live in a "temporal dead zone" until their line runs. `const` additionally can't be reassigned (though object/array contents can still change).

```js
if (true) { var x = 1; let y = 2; }
console.log(x); // 1 (leaks out of block)
console.log(y); // ReferenceError — y is block-scoped
```

### Q72. What is `Array.map()`, `filter()`, `reduce()` — quick recall for interviews?
`map()` transforms each element into a new array (same length). `filter()` keeps only elements matching a condition (shorter or equal length). `reduce()` folds the array down into a single accumulated value.

```js
const nums = [1, 2, 3, 4];
nums.map(n => n * 2);        // [2, 4, 6, 8]
nums.filter(n => n % 2 === 0); // [2, 4]
nums.reduce((sum, n) => sum + n, 0); // 10
```

### Q73. What is `this` inside a regular function vs an arrow function?
In a regular function, `this` depends on how the function is called (can be dynamically rebound). In an arrow function, `this` is lexically inherited from the enclosing scope at definition time and never changes — this is why arrow functions are preferred for callbacks inside class methods.

```js
class Timer {
  constructor() { this.seconds = 0; }
  start() {
    setInterval(() => { this.seconds++; }, 1000); // arrow keeps `this` = Timer instance
  }
}
```

### Q74. What is a higher-order function?
A function that either takes another function as an argument, returns a function, or both. `map`, `filter`, `setTimeout`, and Express middleware factories are all higher-order functions.

```js
function withLogging(fn) {
  return function (...args) {
    console.log('Calling with', args);
    return fn(...args);
  };
}
```

### Q75. What is the difference between a library and a framework (e.g. Express vs raw Node `http`)?
A library (like a simple utility package) is something YOU call when you need it — you're in control of the flow. A framework (like Express, NestJS) calls YOUR code — it dictates the structure (routes, middleware) and you plug your logic into it ("inversion of control").

```js
// Library style: you call it
const _ = require('lodash');
_.chunk([1,2,3,4], 2);

// Framework style: it calls you
app.get('/users', (req, res) => { /* Express calls this */ });
```


---

# PART 2: MID-LEVEL (Q76–150)

### Q76. Explain the phases of the Node.js Event Loop in detail.
The event loop runs in phases, each with its own callback queue: **Timers** (runs `setTimeout`/`setInterval` callbacks whose time has elapsed), **Pending callbacks** (some system-level callbacks deferred from the previous cycle), **Poll** (retrieves new I/O events, executes I/O callbacks — the loop can block here waiting for events), **Check** (runs `setImmediate` callbacks), **Close callbacks** (e.g. `socket.on('close')`). Between every phase, all `process.nextTick()` and microtask (Promise) callbacks are drained first.

```js
// Rough mental model:
// timers -> pending -> poll -> check -> close  (repeat)
// microtasks (Promise/.then, process.nextTick) run between EVERY step
```

### Q76b. What is the difference between the poll phase and the check phase?
The poll phase is where Node actually waits for and processes new I/O events (like a completed file read); it can block here if there's nothing else to do and no timers pending. The check phase runs immediately after poll finishes, exclusively for `setImmediate()` callbacks — this is why `setImmediate` inside an I/O callback always fires before a `setTimeout(fn, 0)` registered at the same point.

```js
const fs = require('fs');
fs.readFile(__filename, () => {
  setTimeout(() => console.log('timeout'), 0);
  setImmediate(() => console.log('immediate')); // always fires first here
});
```

### Q77. What is the difference between microtasks and macrotasks?
Microtasks (`Promise.then`, `queueMicrotask`, `process.nextTick`) are executed immediately after the current operation, before the event loop moves to its next phase. Macrotasks (`setTimeout`, `setImmediate`, I/O callbacks) are each tied to a specific event loop phase and only run in their turn. All queued microtasks always run to completion before the next macrotask.

```js
setTimeout(() => console.log('macrotask'), 0);
Promise.resolve().then(() => console.log('microtask'));
// Output: microtask, macrotask
```

### Q78. What is `process.nextTick()` and how is it different from microtasks like Promises?
`process.nextTick()` queues a callback to run at the very end of the current operation, before the event loop continues — and it actually runs BEFORE Promise microtasks, in its own separate queue that's fully drained first. Overusing it can starve I/O, since it keeps letting more `nextTick` callbacks cut in line.

```js
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('promise'));
// Output: nextTick, promise
```

### Q79. What is `Promise.all()` vs `Promise.allSettled()`?
`Promise.all()` resolves when all promises succeed, but rejects immediately if any ONE fails (fail-fast). `Promise.allSettled()` waits for all promises to finish regardless of outcome and gives you an array of `{status, value/reason}` for each — useful when you want results even if some calls failed.

```js
const results = await Promise.allSettled([fetchA(), fetchB(), fetchC()]);
results.forEach(r => console.log(r.status, r.value || r.reason));
```

### Q80. What is `Promise.race()` and `Promise.any()`?
`Promise.race()` resolves/rejects as soon as the FIRST promise settles (success or failure) — often used for timeouts. `Promise.any()` resolves as soon as the FIRST one succeeds, and only rejects if ALL of them fail.

```js
const withTimeout = Promise.race([
  fetchData(),
  new Promise((_, reject) => setTimeout(() => reject('timeout'), 5000))
]);
```

### Q81. How do you run async operations in parallel vs sequentially?
Sequential: `await` each call one after another — total time is the sum. Parallel: start all the promises first (without awaiting individually), then `await Promise.all()` — total time is roughly the max of them, since they run concurrently.

```js
// Sequential (slow): 3 + 3 = 6 seconds total
const a = await taskA(); // takes 3s
const b = await taskB(); // takes 3s

// Parallel (fast): ~3 seconds total
const [a2, b2] = await Promise.all([taskA(), taskB()]);
```

### Q82. What is backpressure in streams and how do you handle it?
Backpressure happens when a writable stream can't consume data as fast as a readable stream produces it, causing memory buildup. `.write()` returns `false` when its internal buffer is full — you should pause the readable stream until the writable emits `'drain'`. Using `.pipe()` handles this automatically for you.

```js
readable.on('data', (chunk) => {
  if (!writable.write(chunk)) {
    readable.pause();
    writable.once('drain', () => readable.resume());
  }
});
```

### Q83. How do you create a custom Readable stream?
Extend `stream.Readable` and implement the `_read()` method, which pushes data using `this.push(chunk)`. Push `null` to signal the end of the stream.

```js
const { Readable } = require('stream');
class CounterStream extends Readable {
  constructor(max) { super(); this.i = 0; this.max = max; }
  _read() {
    if (this.i < this.max) this.push(`${this.i++}\n`);
    else this.push(null); // end of stream
  }
}
new CounterStream(3).pipe(process.stdout);
```

### Q84. What is the `child_process` module used for?
It lets Node spawn separate OS processes to run other programs or scripts — useful for offloading CPU-heavy work, running shell commands, or executing another language's code from Node.

```js
const { exec } = require('child_process');
exec('ls -la', (err, stdout) => console.log(stdout));
```

### Q85. What is the difference between `exec`, `execFile`, `spawn`, and `fork`?
`exec` runs a shell command and buffers all output in memory (bad for large output). `execFile` runs an executable directly without a shell (safer, avoids shell injection). `spawn` streams output instead of buffering it — good for long-running or large-output processes. `fork` is a special case of `spawn` specifically for launching another Node.js script, with a built-in IPC (inter-process communication) channel.

```js
const { spawn, fork } = require('child_process');
const ls = spawn('ls', ['-la']); // streams stdout in chunks
const child = fork('./worker.js'); // Node-to-Node with message passing
child.send({ task: 'compute' });
child.on('message', (msg) => console.log('From child:', msg));
```

### Q86. What is clustering in Node.js and why do you need it?
Since Node is single-threaded, one process can only use one CPU core. The `cluster` module lets you fork multiple worker processes (usually one per CPU core), all sharing the same server port, so your app can use all cores and survive individual worker crashes.

```js
const cluster = require('cluster');
const os = require('os');
if (cluster.isPrimary) {
  os.cpus().forEach(() => cluster.fork());
} else {
  require('./server'); // each worker runs the actual app
}
```

### Q87. What is the difference between the `cluster` module and Worker Threads?
`cluster` spawns full separate OS processes — each has its own memory and event loop, communicates via IPC/serialized messages, and is best for scaling network I/O across cores. `worker_threads` run inside the same process, can share memory directly (via `SharedArrayBuffer`), and are better suited for CPU-heavy tasks (like image processing) since it avoids process-spawn overhead.

```js
const { Worker } = require('worker_threads');
const worker = new Worker('./heavy-task.js');
worker.on('message', (result) => console.log(result));
```

### Q88. What is a memory leak in Node.js, and can you give a common cause?
A memory leak happens when memory that's no longer needed isn't released, because something still holds a reference to it, causing memory usage to grow indefinitely. A common cause: forgetting to remove event listeners, or an ever-growing array/cache/global variable that's never cleared.

```js
// LEAK: listeners pile up on every request, never removed
app.get('/', (req, res) => {
  emitter.on('data', () => {}); // never cleaned up
});
```

### Q89. How do you debug a Node.js application?
Using the built-in inspector (`node --inspect app.js`) connected to Chrome DevTools, or `node --inspect-brk` to pause at the first line. VS Code also has a built-in Node debugger. You can also add breakpoints with the `debugger;` statement.

```bash
node --inspect-brk server.js
# then open chrome://inspect in Chrome
```

### Q90. What is `console.log` vs `console.error` vs `console.table`?
`console.log` writes to stdout (standard output). `console.error` writes to stderr (standard error) — useful because logs/monitoring tools often separate these streams. `console.table` renders array/object data as a formatted table, handy for debugging structured data.

```js
console.table([{ name: 'A', age: 25 }, { name: 'B', age: 30 }]);
```

### Q91. What is the purpose of a `.gitignore` and `.npmignore` in a Node project?
`.gitignore` tells git which files/folders to skip when committing (like `node_modules`, `.env`, build output). `.npmignore` (or the `files` field in `package.json`) controls what gets included when you publish your package to npm — you don't want to ship tests or source maps to consumers.

```
# .gitignore
node_modules/
.env
dist/
```

### Q92. What is Jest / Mocha used for, and how do you write a basic test?
They're testing frameworks for writing and running unit/integration tests in Node. Jest is all-in-one (runner + assertions + mocking); Mocha is often paired with Chai for assertions.

```js
// sum.test.js (Jest)
const sum = (a, b) => a + b;
test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});
```

### Q93. What is mocking in testing, and why do you need it?
Mocking replaces a real dependency (like a database call or external API) with a fake version that returns controlled data, so tests are fast, deterministic, and don't depend on external systems being up.

```js
jest.mock('../db');
const db = require('../db');
db.getUser.mockResolvedValue({ id: 1, name: 'Test User' });
```

### Q94. What is the difference between unit tests, integration tests, and end-to-end (E2E) tests?
Unit tests check a single function/module in isolation (fast, mocked dependencies). Integration tests check how multiple pieces work together (e.g. route + DB). E2E tests simulate a real user flow through the whole running system (slowest, most realistic).

```js
// Unit: testing one function
test('add()', () => expect(add(1,2)).toBe(3));
// Integration: testing an actual route + real test DB
test('POST /users creates a user', async () => {
  const res = await request(app).post('/users').send({ name: 'A' });
  expect(res.status).toBe(201);
});
```

### Q95. What is `supertest` used for?
A library for testing HTTP servers (like Express apps) by making real HTTP-like requests against your app in-memory, without needing to actually start it on a port.

```js
const request = require('supertest');
const app = require('../app');
test('GET / returns 200', async () => {
  const res = await request(app).get('/');
  expect(res.statusCode).toBe(200);
});
```

### Q96. What is the difference between `PUT`-based idempotency and `POST` non-idempotency?
Idempotent means calling it multiple times has the same effect as calling it once. `PUT` and `DELETE` are meant to be idempotent (replacing/deleting the same resource repeatedly gives the same end state). `POST` is NOT idempotent — calling it twice usually creates two resources (e.g. two orders).

```js
// Calling this 5 times still results in the SAME final state:
app.put('/users/5', (req, res) => { /* replace user 5 with req.body */ });
// Calling this 5 times creates 5 different orders:
app.post('/orders', (req, res) => { /* creates a new order every time */ });
```

### Q97. What is connection pooling and why is it important for databases in Node?
Instead of opening a new database connection for every request (slow, resource-heavy), a connection pool keeps a set of reusable open connections that requests borrow and return. This drastically improves performance under load.

```js
const { Pool } = require('pg');
const pool = new Pool({ max: 20 }); // up to 20 reusable connections
const result = await pool.query('SELECT * FROM users');
```

### Q98. What is an ORM/ODM and why use one?
ORM (Object-Relational Mapping, e.g. Sequelize, Prisma for SQL) or ODM (Object-Document Mapping, e.g. Mongoose for MongoDB) lets you interact with a database using JS objects/classes instead of writing raw queries — adds validation, relationships, and migrations, at the cost of some performance/control.

```js
// Prisma example
const user = await prisma.user.create({ data: { name: 'A', email: 'a@x.com' } });
```

### Q99. What are database transactions and when do you need them in Node?
A transaction groups multiple database operations so they either ALL succeed together or ALL fail together (atomicity) — essential when an operation touches multiple records/collections that must stay consistent (e.g. transferring money between two accounts).

```js
const session = await mongoose.startSession();
session.startTransaction();
try {
  await Account.updateOne({_id: a}, {$inc: {balance: -100}}, {session});
  await Account.updateOne({_id: b}, {$inc: {balance: 100}}, {session});
  await session.commitTransaction();
} catch (e) {
  await session.abortTransaction();
}
```

### Q100. What is indexing in a database and how does it help Node apps at scale?
An index is a data structure (usually a B-tree) that lets the database find matching rows/documents without scanning the entire collection/table — massively speeding up queries on large datasets, at the cost of extra storage and slightly slower writes.

```js
userSchema.index({ email: 1 }, { unique: true }); // Mongoose
// or in SQL: CREATE INDEX idx_email ON users(email);
```

### Q101. What is the N+1 query problem and how do you avoid it in Node/Mongoose?
It happens when you fetch a list of N items, then run a separate query for each item's related data (1 query + N extra queries), which is slow. You avoid it by using `populate()` (Mongoose) or `JOIN`s (SQL) to fetch related data in fewer queries.

```js
// BAD: N+1 — one query per post to get its author
const posts = await Post.find();
for (const post of posts) { post.author = await User.findById(post.authorId); }

// GOOD: single query with populate
const posts = await Post.find().populate('author');
```

### Q102. What is caching, and how do you implement it with Redis in Node?
Caching stores the result of an expensive operation (DB query, computation) in fast memory (like Redis) so repeated requests for the same data don't hit the database again — hugely reducing latency and load.

```js
const redis = require('redis').createClient();
async function getUser(id) {
  const cached = await redis.get(`user:${id}`);
  if (cached) return JSON.parse(cached);
  const user = await User.findById(id);
  await redis.set(`user:${id}`, JSON.stringify(user), { EX: 3600 }); // expire in 1hr
  return user;
}
```

### Q103. What is cache invalidation and why is it hard?
It's the process of removing or updating stale cached data when the underlying data changes. It's hard because you need to know exactly WHEN and WHAT to invalidate — too aggressive and you lose caching benefits, too lazy and users see stale data.

```js
async function updateUser(id, data) {
  const user = await User.findByIdAndUpdate(id, data, { new: true });
  await redis.del(`user:${id}`); // invalidate stale cache
  return user;
}
```

### Q104. What is a WebSocket, and how is it different from HTTP?
HTTP is request-response — the client always initiates, connection closes after each response. A WebSocket opens a single persistent, full-duplex connection where BOTH client and server can push messages at any time — used for chat apps, live notifications, real-time dashboards.

```js
const { Server } = require('socket.io');
const io = new Server(httpServer);
io.on('connection', (socket) => {
  socket.on('message', (msg) => io.emit('message', msg)); // broadcast to everyone
});
```

### Q105. What is Socket.IO and how does it differ from raw WebSockets?
Socket.IO is a library built on top of WebSockets that adds automatic reconnection, fallback to HTTP long-polling if WebSockets aren't available, room/namespace support, and an easy event-based API — trading a bit of raw performance for much better developer experience and browser compatibility.

```js
io.on('connection', (socket) => {
  socket.join('room1');
  io.to('room1').emit('event', 'hello room 1 only');
});
```

### Q106. What is the difference between `req.body`, `req.params`, and `req.query` in Express?
`req.body` holds data sent in the request body (POST/PUT payloads, needs body-parsing middleware). `req.params` holds named URL route segments (e.g. `/users/:id`). `req.query` holds the URL query string key-value pairs after `?`.

```js
// POST /users/5?notify=true   body: { "name": "A" }
req.params.id   // "5"
req.query.notify // "true"
req.body.name   // "A"
```

### Q107. What is dependency injection, and is it common in Node?
It's a design pattern where a component receives its dependencies from outside rather than creating them itself, making code more testable and decoupled. It's less baked-in than in frameworks like Angular/NestJS, but plain Node apps use it manually — e.g. passing a DB instance into a service function.

```js
// Instead of hardcoding inside the function:
function createUserService(db) { // db injected
  return { getUser: (id) => db.collection('users').findOne({ id }) };
}
```

### Q108. What is the Repository pattern in a Node app?
It's a design pattern that abstracts data-access logic behind an interface (repository), so the rest of the app doesn't care whether data comes from MongoDB, Postgres, or an in-memory store — making it easier to test and swap databases.

```js
class UserRepository {
  async findById(id) { return User.findById(id); }
  async create(data) { return User.create(data); }
}
// Business logic only talks to userRepository, not Mongoose directly
```

### Q109. What is MVC architecture in the context of a Node/Express app?
Model (schema/database logic), View (what's returned to the client — JSON in APIs, or templates in server-rendered apps), Controller (handles the request, calls the model, sends the response). It separates concerns so routes, business logic, and data access don't all live in one file.

```
/models/User.js       — Mongoose schema
/controllers/userController.js  — request handling logic
/routes/userRoutes.js — maps URLs to controller functions
```

### Q110. What is a service layer, and why separate it from controllers?
The service layer holds the actual business logic (rules, calculations, orchestration), while controllers just handle HTTP-specific concerns (parsing request, sending response). This separation lets you reuse business logic outside of HTTP (e.g. in a CLI script or a cron job) and makes unit testing easier.

```js
// controller — just HTTP glue
async function register(req, res) {
  const user = await userService.registerUser(req.body);
  res.status(201).json(user);
}
// service — actual logic, testable without req/res
async function registerUser(data) {
  const hashed = await bcrypt.hash(data.password, 10);
  return User.create({ ...data, password: hashed });
}
```

### Q111. What is the difference between `Array.forEach()` and using `for...of` with `await` inside?
`forEach()` does NOT wait for async callbacks — it fires them all off immediately without pausing, so `await` inside it is essentially useless for sequencing. `for...of` properly pauses at each `await`, processing items one at a time.

```js
// BROKEN: doesn't actually wait between iterations
items.forEach(async (item) => { await save(item); });

// CORRECT: waits for each save before moving to the next
for (const item of items) { await save(item); }
```

### Q112. How do you process an array of async tasks with a concurrency limit (not all at once)?
Using a library like `p-limit`, or manually chunking the array and using `Promise.all` per chunk — this avoids overwhelming a database or external API with thousands of simultaneous requests.

```js
const pLimit = require('p-limit');
const limit = pLimit(5); // max 5 concurrent
const results = await Promise.all(items.map(item => limit(() => processItem(item))));
```

### Q113. What is graceful shutdown and why does a Node server need it?
It's closing a server cleanly — finishing in-flight requests, closing DB connections, releasing resources — instead of abruptly killing the process, which could corrupt data or drop active users. Triggered by listening for `SIGTERM`/`SIGINT`.

```js
process.on('SIGTERM', async () => {
  console.log('Shutting down gracefully...');
  server.close(() => { db.disconnect(); process.exit(0); });
});
```

### Q114. What is `SIGTERM` vs `SIGKILL`?
`SIGTERM` politely asks a process to terminate — it CAN be caught and handled (e.g. for graceful shutdown). `SIGKILL` forcefully and immediately kills the process — it cannot be caught, ignored, or handled at all.

```js
process.on('SIGTERM', () => { console.log('cleaning up...'); process.exit(0); });
// SIGKILL bypasses this entirely — no cleanup possible
```

### Q115. What is the difference between horizontal and vertical scaling for a Node app?
Vertical scaling means giving one server more resources (more RAM/CPU) — simple but has a hard ceiling. Horizontal scaling means running multiple instances of your app across multiple machines/containers behind a load balancer — more complex (needs stateless design) but scales much further.

```js
// Horizontal scaling requires the app to be stateless —
// e.g. sessions in Redis, not in local memory, so any instance can serve any request
```

### Q116. What is a load balancer, and how does it relate to Node clustering?
A load balancer distributes incoming traffic across multiple server instances (or cluster workers) so no single one is overwhelmed, and can also reroute traffic away from a crashed instance. It can operate at the OS/network level (e.g. Nginx) or within Node itself via the `cluster` module's round-robin default.

```js
// Nginx config snippet
upstream node_app { server 127.0.0.1:3000; server 127.0.0.1:3001; }
```

### Q117. What is a reverse proxy, and why put one (like Nginx) in front of Node?
A reverse proxy sits between clients and your Node servers, handling things Node isn't great at doing directly — SSL termination, static file serving, load balancing, compression, and basic security filtering — so your Node process can focus purely on application logic.

```
Client → Nginx (SSL, static files, load balancing) → Node.js app servers
```

### Q118. What is CORS preflight, and what request triggers it?
Before certain cross-origin requests (like those with custom headers, or methods other than GET/POST with simple content-types), the browser automatically sends an `OPTIONS` request first to check if the actual request is allowed — this is the "preflight" check.

```js
app.options('/api/data', cors()); // explicitly handle preflight
app.get('/api/data', cors(), handler);
```

### Q119. What is the difference between `res.send()`, `res.json()`, and `res.end()` in Express?
`res.json()` sets the `Content-Type` to `application/json` and stringifies the data. `res.send()` is more flexible — it auto-detects the type (string, buffer, object) and sets headers accordingly. `res.end()` is the lowest-level, raw method to end the response with no content-type magic — mostly used internally.

```js
res.json({ ok: true });    // always JSON
res.send('plain text');    // auto content-type
res.end();                 // just close the connection
```

### Q120. What is `multer` used for in Node/Express?
It's a middleware for handling `multipart/form-data`, which is the encoding used for file uploads — it parses uploaded files and makes them available on `req.file`/`req.files`.

```js
const multer = require('multer');
const upload = multer({ dest: 'uploads/' });
app.post('/upload', upload.single('avatar'), (req, res) => {
  console.log(req.file); // uploaded file info
  res.send('Uploaded!');
});
```

### Q121. What is streaming file uploads/downloads, and why prefer it over loading the whole file into memory?
Streaming processes a file in small chunks as it arrives/leaves, keeping memory usage constant regardless of file size. Loading an entire large file into memory (e.g. via `fs.readFileSync`) can crash the server under load or with big files.

```js
app.get('/download', (req, res) => {
  fs.createReadStream('large-video.mp4').pipe(res); // streams, doesn't load fully in RAM
});
```

### Q122. What is GraphQL, and how is it different from REST in a Node backend?
GraphQL lets the client specify EXACTLY what fields/data it needs in a single request (via a query language), avoiding over-fetching or under-fetching that's common with fixed REST endpoints. REST has many endpoints returning fixed shapes; GraphQL typically has one endpoint with a flexible schema.

```js
// GraphQL query — client asks for exactly these fields
const query = `{ user(id: "1") { name email posts { title } } }`;
```

### Q123. What is `apollo-server` (or similar) used for in Node?
It's a library to build a GraphQL server in Node — you define a schema (types + queries/mutations) and resolver functions that fetch the actual data for each field.

```js
const typeDefs = `type Query { hello: String }`;
const resolvers = { Query: { hello: () => 'Hello world!' } };
const server = new ApolloServer({ typeDefs, resolvers });
```

### Q124. What is the difference between a monolith and microservices architecture?
A monolith is one single deployable application containing all features/modules together — simpler to build and deploy initially. Microservices split the app into small, independently deployable services (each with its own responsibility, often its own database) that communicate over the network — more scalable and flexible, but adds operational complexity (networking, data consistency, deployment orchestration).

```
Monolith:      [ Auth + Orders + Payments + Users ] — one app, one deploy
Microservices: [Auth Service] [Order Service] [Payment Service] — separate apps
```

### Q125. How do Node.js microservices typically communicate with each other?
Synchronously via HTTP/REST or gRPC (direct request-response calls), or asynchronously via a message broker/queue (RabbitMQ, Kafka, Redis Pub/Sub) where services publish events and others subscribe — async decouples services so one being down doesn't immediately break another.

```js
// Async example with a queue
await channel.sendToQueue('order_created', Buffer.from(JSON.stringify(order)));
// A separate "notification service" consumes from this queue independently
```

### Q126. What is a message queue and why use one (e.g. RabbitMQ) with Node?
A message queue lets one service send ("publish") a message without waiting for another service to process it immediately — the message sits in a queue until a consumer picks it up. This decouples producers and consumers, smooths out traffic spikes, and adds resilience (if a consumer is down, messages just wait).

```js
const amqp = require('amqplib');
const conn = await amqp.connect('amqp://localhost');
const channel = await conn.createChannel();
await channel.assertQueue('emails');
channel.sendToQueue('emails', Buffer.from('send welcome email'));
```

### Q127. What is Kafka, and how does it differ from a traditional message queue like RabbitMQ?
Kafka is a distributed event streaming platform — messages ("events") are appended to a durable, ordered log (a "topic") and can be replayed/re-read by multiple independent consumers. Traditional queues like RabbitMQ typically remove a message once consumed by one consumer. Kafka is built for high-throughput event streaming and replayability; RabbitMQ is built for flexible task/message routing.

```js
// kafkajs example
const producer = kafka.producer();
await producer.send({ topic: 'orders', messages: [{ value: JSON.stringify(order) }] });
```

### Q128. What is idempotency in the context of message queues/APIs, and why does it matter?
It means processing the same message/request multiple times produces the same end result as processing it once — important because message queues often guarantee "at-least-once" delivery, so your consumer MUST handle duplicate messages safely (e.g. by checking an already-processed ID before applying an update).

```js
async function processOrder(msg) {
  const alreadyDone = await Processed.findOne({ msgId: msg.id });
  if (alreadyDone) return; // skip duplicate
  await applyOrder(msg);
  await Processed.create({ msgId: msg.id });
}
```

### Q129. What is `async_hooks` in Node.js?
A low-level core API that lets you track the lifecycle of asynchronous resources (when a Promise, timer, or callback is created, before/after it runs, and when it's destroyed) — mainly used by tools like APM/tracing libraries to follow a request across async boundaries, not typically used directly in app code.

```js
const async_hooks = require('async_hooks');
const hook = async_hooks.createHook({
  init(asyncId, type) { console.log('New async resource:', type); }
});
hook.enable();
```

### Q130. What is `AsyncLocalStorage` and what problem does it solve?
It lets you store context (like a request ID or logged-in user) that's automatically available anywhere inside an async call chain, without manually passing it through every function argument — very useful for request-scoped logging/tracing in Node servers.

```js
const { AsyncLocalStorage } = require('async_hooks');
const als = new AsyncLocalStorage();
app.use((req, res, next) => als.run({ requestId: crypto.randomUUID() }, next));
function logInfo(msg) {
  console.log(als.getStore().requestId, msg); // accessible anywhere downstream
}
```

### Q131. What is the difference between `Object.assign()` and the spread operator for merging objects?
Both do a shallow merge and behave almost identically for plain objects, but `Object.assign()` mutates the FIRST argument if it's an existing object, while spread `{...a, ...b}` always creates a brand-new object without mutating either source.

```js
const a = { x: 1 };
Object.assign(a, { y: 2 }); // mutates `a` itself
const c = { ...a, y: 3 };   // new object, `a` untouched
```

### Q132. What is a deep clone vs a shallow clone, and how do you deep clone an object in Node?
A shallow clone copies only the top-level properties — nested objects/arrays are still shared by reference. A deep clone recursively copies everything so nothing is shared. Modern Node supports `structuredClone()` natively for deep cloning.

```js
const obj = { a: 1, nested: { b: 2 } };
const shallow = { ...obj };
shallow.nested.b = 99; // also changes obj.nested.b! (shared reference)

const deep = structuredClone(obj); // truly independent copy
```

### Q133. What is the difference between `Map` and a plain Object in JS/Node?
`Map` preserves insertion order reliably, allows any type (not just strings) as keys, has a `.size` property, and performs better for frequent additions/removals. A plain Object is simpler for JSON-like data but keys are coerced to strings and iteration order has quirks with numeric keys.

```js
const map = new Map();
map.set({ id: 1 }, 'value for object key'); // objects as keys — impossible with plain Object
console.log(map.size);
```

### Q134. What is a `WeakMap` and when would you use one?
Like a `Map`, but its keys must be objects, and if there are no other references to a key object, it can be garbage collected — the WeakMap won't keep it alive. Useful for attaching private/extra data to objects without causing memory leaks.

```js
const cache = new WeakMap();
function process(obj) {
  if (cache.has(obj)) return cache.get(obj);
  const result = expensiveComputation(obj);
  cache.set(obj, result);
  return result;
} // if `obj` is garbage collected elsewhere, its cache entry disappears too
```

### Q135. What is the difference between `Symbol` and using string keys in objects?
A `Symbol` creates a guaranteed-unique value, often used as an object property key to avoid naming collisions (e.g. between two libraries adding the same-named property) and to create "hidden" properties that don't show up in normal `for...in` or `JSON.stringify()`.

```js
const id = Symbol('id');
const user = { name: 'A', [id]: 123 };
console.log(JSON.stringify(user)); // {"name":"A"} — symbol key is skipped
```

### Q136. What is the purpose of `express.static()`?
It serves static files (HTML, CSS, images, client-side JS) directly from a folder, without needing a route handler for each file.

```js
app.use(express.static('public'));
// A request to /logo.png automatically serves ./public/logo.png
```

### Q137. What is compression middleware, and why use it?
`compression` middleware gzips/brotli-compresses HTTP responses before sending them, reducing payload size and speeding up transfer over the network, at the cost of a small CPU overhead to compress.

```js
const compression = require('compression');
app.use(compression());
```

### Q138. What is the purpose of `express.Router()`?
It creates a mini, self-contained Express application (a modular route handler) that you can mount onto your main app under a path prefix — keeps large apps organized into separate files by feature.

```js
// userRoutes.js
const router = express.Router();
router.get('/', listUsers);
router.get('/:id', getUser);
module.exports = router;

// app.js
app.use('/users', require('./userRoutes'));
```

### Q139. What is the difference between synchronous and asynchronous error handling in Express route handlers?
Express automatically catches errors THROWN synchronously inside a route handler and forwards them to your error middleware. But it does NOT automatically catch errors from rejected Promises inside `async` functions (in Express 4) — you must manually `catch` and call `next(err)`, or use a wrapper utility, or upgrade to Express 5 which handles this natively.

```js
// Express 4 — must catch manually
app.get('/data', async (req, res, next) => {
  try {
    const data = await riskyOperation();
    res.json(data);
  } catch (err) { next(err); } // forward to error middleware
});
```

### Q140. What is a wrapper function for async route handlers, and why use it?
It's a small utility that wraps an async route handler in a `try/catch` automatically, so you don't repeat boilerplate `try/catch` in every single route.

```js
const asyncHandler = (fn) => (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);
app.get('/data', asyncHandler(async (req, res) => {
  const data = await riskyOperation(); // errors auto-forwarded to error middleware
  res.json(data);
}));
```

### Q141. What is the Circuit Breaker pattern, and why is it useful in Node microservices?
It's a pattern where, after a downstream service fails repeatedly, calls to it are automatically "short-circuited" (fail fast without even trying) for a cooldown period — preventing a struggling downstream service from being overwhelmed further, and preventing your own service from hanging on slow/dead calls.

```js
const CircuitBreaker = require('opossum');
const breaker = new CircuitBreaker(callExternalApi, { timeout: 3000, errorThresholdPercentage: 50 });
breaker.fallback(() => ({ message: 'Service temporarily unavailable' }));
```

### Q142. What is retry logic with exponential backoff, and how do you implement it?
It's retrying a failed operation multiple times, waiting progressively LONGER between each attempt (e.g. 1s, 2s, 4s, 8s), instead of hammering a struggling service immediately and repeatedly.

```js
async function retryWithBackoff(fn, retries = 5, delay = 1000) {
  for (let i = 0; i < retries; i++) {
    try { return await fn(); }
    catch (err) {
      if (i === retries - 1) throw err;
      await new Promise(r => setTimeout(r, delay * 2 ** i));
    }
  }
}
```

### Q143. What is the difference between `axios` and the built-in `fetch` in Node?
`fetch` is now built into modern Node (18+) and browsers natively — no install needed, but requires manual JSON parsing and doesn't throw on HTTP error status codes (like 404/500) by default. `axios` is a third-party library with automatic JSON transform, request/response interceptors, easier error handling (throws on non-2xx), and wider legacy Node support.

```js
// fetch
const res = await fetch(url); const data = await res.json();
// axios
const { data } = await axios.get(url); // JSON parsed automatically, throws on 4xx/5xx
```

### Q144. What is a health-check endpoint, and why does every production Node service need one?
It's a simple route (like `/health` or `/status`) that returns whether the app (and its critical dependencies like the DB) is up and functioning — load balancers and orchestration tools (Kubernetes, PM2) use it to decide whether to route traffic to this instance or restart it.

```js
app.get('/health', async (req, res) => {
  const dbOk = mongoose.connection.readyState === 1;
  res.status(dbOk ? 200 : 503).json({ status: dbOk ? 'ok' : 'degraded' });
});
```

### Q145. What is PM2, and why is it commonly used to run Node apps in production?
PM2 is a process manager for Node.js — it keeps your app alive (auto-restarts on crash), can run it in cluster mode across all CPU cores, manages logs, and supports zero-downtime reloads, all without changing your app's code.

```bash
pm2 start server.js -i max   # cluster mode, one instance per CPU core
pm2 reload server             # zero-downtime restart
```

### Q146. What is the difference between `npm run build` and `npm start` typically doing in a Node project?
`npm run build` usually compiles/transpiles source code (TypeScript → JS, bundling, minifying) into a production-ready output folder. `npm start` runs the actual application (often the already-built output) — these are custom scripts you define in `package.json`, not built-in Node behavior.

```json
{
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js"
  }
}
```

### Q147. What is TypeScript, and why do teams use it with Node.js?
TypeScript is a superset of JavaScript that adds static typing, checked at compile time. It catches type-related bugs before runtime, improves editor autocomplete/refactoring, and makes large codebases easier to maintain — it compiles down to plain JS which Node actually runs.

```ts
interface User { id: number; name: string; }
function getUser(id: number): Promise<User> { /* ... */ }
```

### Q148. What is the difference between `interface` and `type` in TypeScript (as used in Node projects)?
Both describe object shapes. `interface` can be reopened/extended later (declaration merging) and is generally preferred for objects/classes meant to be extended. `type` is more flexible — it can represent unions, primitives, and tuples, but can't be reopened once declared.

```ts
interface User { name: string; }
interface User { age: number; } // merges automatically — now has both fields

type Status = 'active' | 'inactive'; // union type — `interface` can't do this
```

### Q149. What is `ts-node` and `nodemon` used together for?
`ts-node` lets you run TypeScript files directly without a separate compile step, and combined with `nodemon`, it gives you auto-restarting development for a TypeScript Node project.

```bash
npm install -D typescript ts-node nodemon
npx nodemon --exec ts-node src/index.ts
```

### Q150. What is the difference between `import type` and a regular `import` in TypeScript Node projects?
`import type` imports ONLY the type information (interfaces, types) and is completely erased at compile time — it has zero runtime cost/footprint. A regular `import` may bring in actual runtime code. Using `import type` explicitly for types keeps the compiled JS output leaner and avoids accidental circular runtime imports.

```ts
import type { User } from './types'; // erased after compilation, zero runtime cost
import { createUser } from './service'; // actual runtime import
```


---

# PART 3: ADVANCED (Q151–225)

### Q151. Explain the V8 engine's memory structure (heap generations) relevant to Node.js.
V8 divides the heap into the **young generation** (new space — small, for recently created objects, collected frequently and fast via "Scavenge") and the **old generation** (old space — for objects that survived multiple young-gen collections, collected less often but more expensively via "Mark-Sweep-Compact"). Most objects die young, so this generational design keeps garbage collection efficient overall.

```js
// Check current heap stats
const v8 = require('v8');
console.log(v8.getHeapStatistics());
```

### Q152. What is Garbage Collection (GC) and how does V8's Mark-and-Sweep work?
GC automatically frees memory that's no longer reachable from your code. V8's Mark-Sweep-Compact algorithm: **Mark** phase walks from "roots" (global objects, active stack) and marks everything reachable; **Sweep** phase reclaims memory for anything NOT marked; **Compact** phase (periodically) moves surviving objects together to reduce fragmentation.

```js
// Forcing GC for debugging (run node with --expose-gc)
if (global.gc) global.gc();
```

### Q153. What causes a memory leak in a long-running Node server, and how do you find one?
Common causes: uncleared timers/intervals, event listeners never removed, growing caches/arrays with no eviction, closures unintentionally holding references. You find them by taking heap snapshots (via `--inspect` + Chrome DevTools Memory tab, or the `heapdump` package) at different points in time and comparing which objects keep growing.

```js
const heapdump = require('heapdump');
heapdump.writeSnapshot('./' + Date.now() + '.heapsnapshot');
// Load two snapshots in Chrome DevTools and compare object counts
```

### Q154. What is a heap snapshot, and how do you use one to debug memory issues?
It's a full capture of every live object in memory at a point in time, including their sizes and retaining references (what's keeping them alive). By comparing multiple snapshots taken minutes apart under load, you can spot objects whose count/size keeps growing — a strong signal of a leak — and trace the "retainer chain" back to what's holding onto them.

```bash
node --inspect app.js
# In Chrome: chrome://inspect -> Memory tab -> Take Heap Snapshot (take 2, compare)
```

### Q155. What is CPU profiling in Node, and how do you generate a CPU profile?
CPU profiling records which functions are consuming the most CPU time over a period, helping you find performance bottlenecks. Node has a built-in profiler you can trigger with `--prof`, producing a log you process with `--prof-process`, or you can use `--inspect` and Chrome DevTools' Performance/Profiler tab for a visual flame graph.

```bash
node --prof app.js
node --prof-process isolate-0x*.log > processed.txt
```

### Q156. What is event loop lag / starvation, and how do you detect it?
It happens when a synchronous, CPU-heavy operation blocks the event loop for too long, delaying ALL other pending callbacks (timers, I/O, incoming requests) — the server appears "frozen" even though it's technically still running. You detect it by measuring the delay between when a `setTimeout`/`setImmediate` was scheduled and when it actually ran (a growing gap = lag).

```js
let last = Date.now();
setInterval(() => {
  const lag = Date.now() - last - 1000;
  if (lag > 100) console.warn(`Event loop lag: ${lag}ms`);
  last = Date.now();
}, 1000);
```

### Q157. How do you offload CPU-intensive work in Node.js without blocking the event loop?
Move it off the main thread — either to a `worker_threads` Worker (for CPU-bound JS computation), to a separate `child_process` (for isolating crashes or using another language), or to an external service/queue that processes it asynchronously and reports back.

```js
const { Worker } = require('worker_threads');
function runHeavyTask(data) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./heavyTask.js', { workerData: data });
    worker.on('message', resolve);
    worker.on('error', reject);
  });
}
```

### Q158. How do Worker Threads share data efficiently without copying?
Via `SharedArrayBuffer`, a raw binary buffer that multiple threads can access and modify directly in shared memory — avoiding the cost of serializing/copying large data between threads (which regular `postMessage` does by default, using the structured clone algorithm).

```js
const { Worker } = require('worker_threads');
const shared = new SharedArrayBuffer(4);
const view = new Int32Array(shared);
const worker = new Worker('./worker.js', { workerData: shared });
// Both main thread and worker can read/write `view` directly
```

### Q159. What is `Atomics` used for with `SharedArrayBuffer`?
`Atomics` provides thread-safe operations (like `Atomics.add`, `Atomics.wait`, `Atomics.notify`) on shared memory, preventing race conditions when multiple threads read/write the same `SharedArrayBuffer` concurrently — normal read/write operations aren't safe across threads without it.

```js
const shared = new Int32Array(new SharedArrayBuffer(4));
Atomics.add(shared, 0, 1); // thread-safe increment
Atomics.wait(shared, 0, 0); // block until value at index 0 changes from 0
```

### Q160. What is libuv, and what role does it play in Node.js?
libuv is the C library underlying Node.js that implements the event loop itself, and provides a thread pool (default 4 threads) for operations that can't be done asynchronously at the OS level (like `fs` operations on some platforms, DNS lookups, some crypto functions). Network I/O (sockets) typically use the OS's native async mechanisms directly, not the thread pool.

```js
// You can resize libuv's thread pool via an env variable:
process.env.UV_THREADPOOL_SIZE = 8; // must be set BEFORE any threadpool-using call
```

### Q161. What is the libuv thread pool, and which operations use it by default?
It's a pool of worker threads (default size 4) libuv uses for blocking operations that don't have a native async OS API — mainly `fs` module calls, `dns.lookup`, some `crypto` functions (`pbkdf2`, `randomBytes`), and `zlib` compression. If you have more concurrent thread-pool-bound operations than pool size, extra ones queue and wait.

```js
process.env.UV_THREADPOOL_SIZE = 16; // increase if doing lots of concurrent fs/crypto work
```

### Q162. What is the difference between how Node handles file I/O vs network I/O under the hood?
Network I/O (TCP sockets, HTTP) typically uses the operating system's native asynchronous, non-blocking APIs directly (epoll on Linux, kqueue on macOS, IOCP on Windows) — no extra threads needed. File I/O, however, doesn't have reliable native async support across all OSes, so Node offloads it to libuv's thread pool, which then reports back to the event loop when done.

```js
// Both look async in code, but internally:
fs.readFile('file.txt', cb);      // uses libuv thread pool
net.connect(port, host, cb);      // uses OS native async socket APIs
```

### Q163. What is the difference between `Buffer.alloc()` and `Buffer.allocUnsafe()`?
`Buffer.alloc(size)` creates a zero-filled buffer — safe but slightly slower due to the zeroing step. `Buffer.allocUnsafe(size)` is faster because it doesn't clear the memory first — it may contain old, potentially sensitive leftover data, so you must fully overwrite it before using/sending it.

```js
const safe = Buffer.alloc(10);       // all zeros
const fast = Buffer.allocUnsafe(10); // uninitialized — could leak old memory contents if not overwritten
```

### Q164. What is the difference between `Buffer` and `TypedArray` (like `Uint8Array`)?
`Buffer` is a Node-specific subclass of `Uint8Array` with extra convenience methods (`toString()`, `write()`, encoding support). Since Node 4+, `Buffer` and `TypedArray`s share the same underlying `ArrayBuffer` memory model, so they're largely interoperable, but `TypedArray` is the standard JS/browser API while `Buffer` adds Node-specific ergonomics.

```js
const buf = Buffer.from([1, 2, 3]);
const view = new Uint8Array(buf.buffer, buf.byteOffset, buf.length); // shares same memory
```

### Q165. What is a memory-efficient way to concatenate many Buffers?
Avoid repeatedly using `Buffer.concat()` in a loop (it allocates a new buffer every time, O(n²) overall). Instead, collect all chunks in an array first, then call `Buffer.concat(chunksArray)` ONCE at the end.

```js
// BAD: creates a new buffer on every iteration
let result = Buffer.alloc(0);
chunks.forEach(c => { result = Buffer.concat([result, c]); });

// GOOD: one allocation at the end
const result = Buffer.concat(chunks);
```

### Q166. What is the Singleton design pattern, and how would you implement it in Node (e.g. for a DB connection)?
Singleton ensures only ONE instance of something exists across the whole app — commonly used for a database connection or config object, since Node's `require()` cache naturally makes modules behave like singletons already (a module is only executed once, then cached).

```js
// db.js — naturally a singleton because require() caches the module
let connection = null;
module.exports = {
  getConnection: async () => {
    if (!connection) connection = await connectToDb();
    return connection;
  }
};
```

### Q167. What is the Factory design pattern, and where might you use it in a Node app?
A Factory function/class creates and returns objects without exposing the exact creation logic to the caller — useful when object creation depends on some condition (e.g. creating different logger types, or different payment provider clients based on config).

```js
function createLogger(type) {
  if (type === 'file') return new FileLogger();
  if (type === 'console') return new ConsoleLogger();
  throw new Error('Unknown logger type');
}
```

### Q168. What is the Observer pattern, and how does Node's EventEmitter relate to it?
Observer pattern: an object (subject) maintains a list of dependents (observers) and notifies them automatically of any state changes. Node's `EventEmitter` IS a built-in implementation of this pattern — `emit()` notifies all registered `on()` listeners.

```js
class OrderService extends require('events') {
  createOrder(data) {
    const order = { ...data, id: Date.now() };
    this.emit('order:created', order); // notify all subscribers
    return order;
  }
}
```

### Q169. What is the Strategy design pattern, and where is it useful in Node?
It defines a family of interchangeable algorithms/behaviors and lets you swap between them at runtime — useful for things like choosing between different payment gateways, sorting strategies, or authentication methods without changing the calling code.

```js
const strategies = {
  stripe: (amount) => chargeWithStripe(amount),
  paypal: (amount) => chargeWithPaypal(amount)
};
function processPayment(provider, amount) { return strategies[provider](amount); }
```

### Q170. What is the Middleware design pattern (as a general concept, beyond Express)?
It's a chain of processing functions where each one can inspect/modify data and decide to pass control to the next one (or stop the chain). Express's request pipeline is the classic example, but the same pattern applies to Redux middleware, GraphQL resolver chains, or any pipeline processing.

```js
function compose(...fns) {
  return (input) => fns.reduce((acc, fn) => fn(acc), input);
}
const pipeline = compose(sanitize, validate, transform);
```

### Q171. What is the difference between SQL injection and NoSQL injection, and how do you prevent both in Node?
SQL injection happens when unsanitized user input is concatenated directly into a raw SQL string, letting attackers inject their own SQL. NoSQL injection (specific to MongoDB) happens when user input containing operators (like `{"$gt": ""}`) is passed directly into a query object, altering its logic. Prevention: use parameterized queries/prepared statements for SQL, and libraries like `express-mongo-sanitize` (or Mongoose's built-in casting) for MongoDB.

```js
// SQL — safe with parameterized query
await client.query('SELECT * FROM users WHERE email = $1', [email]);

// MongoDB — vulnerable if unsanitized
await User.findOne({ password: req.body.password }); // attacker sends {"$ne": null}
// Fix: sanitize/validate input types before querying
```

### Q172. What is XSS (Cross-Site Scripting), and how do Node/Express apps get protected?
XSS happens when untrusted user input is rendered as HTML/JS on a page without escaping, letting an attacker inject a malicious script that runs in other users' browsers. Protection: escape/sanitize any user content before rendering in HTML (template engines usually auto-escape), set a strict `Content-Security-Policy` header (e.g. via `helmet`), and sanitize rich-text input with a library like `DOMPurify`.

```js
const helmet = require('helmet');
app.use(helmet.contentSecurityPolicy({
  directives: { scriptSrc: ["'self'"] } // blocks inline/external untrusted scripts
}));
```

### Q173. What is a timing attack, and how does it relate to comparing secrets (like passwords/tokens) in Node?
A timing attack exploits the fact that a naive string comparison (`===`) returns FALSE as soon as it finds the first mismatched character — so comparison time subtly reveals how many leading characters were correct, letting an attacker guess a secret character by character. The fix is a constant-time comparison function that always takes the same time regardless of where the mismatch is.

```js
const crypto = require('crypto');
function safeCompare(a, b) {
  return crypto.timingSafeEqual(Buffer.from(a), Buffer.from(b));
}
```

### Q174. What is the difference between symmetric and asymmetric encryption, and where does Node's `crypto` module fit in?
Symmetric encryption uses the SAME key to encrypt and decrypt (fast, but the key must be shared secretly beforehand — e.g. AES). Asymmetric encryption uses a public/private key PAIR — anyone can encrypt with the public key, but only the private key holder can decrypt (slower, no shared-secret problem — used in TLS handshakes, JWT signing with RS256). Node's built-in `crypto` module supports both.

```js
const crypto = require('crypto');
// Symmetric (AES)
const cipher = crypto.createCipheriv('aes-256-cbc', key, iv);
// Asymmetric (RSA) key pair generation
const { publicKey, privateKey } = crypto.generateKeyPairSync('rsa', { modulusLength: 2048 });
```

### Q175. What is HTTPS/TLS, and how do you set it up in a Node server?
TLS (Transport Layer Security, the successor to SSL) encrypts data in transit between client and server, and HTTPS is HTTP running over TLS. In Node, you use the `https` module with a certificate and private key (often terminated at a reverse proxy like Nginx in production instead, for simplicity).

```js
const https = require('https');
const fs = require('fs');
const options = {
  key: fs.readFileSync('key.pem'),
  cert: fs.readFileSync('cert.pem')
};
https.createServer(options, app).listen(443);
```

### Q176. What is HTTP/2, and what advantages does it bring to Node servers?
HTTP/2 is a binary (not text-based) protocol that supports multiplexing (many requests/responses over a SINGLE TCP connection, eliminating head-of-line blocking at the app layer), header compression (HPACK), and server push. Node has a built-in `http2` module to build HTTP/2 servers, offering reduced latency versus HTTP/1.1's one-request-per-connection-at-a-time model.

```js
const http2 = require('http2');
const server = http2.createSecureServer({ key, cert });
server.on('stream', (stream, headers) => {
  stream.respond({ ':status': 200 });
  stream.end('Hello HTTP/2');
});
```

### Q177. What is the difference between JWT access tokens and refresh tokens?
An access token is short-lived (minutes) and sent with every API request to prove identity — if stolen, the damage window is small. A refresh token is long-lived and used ONLY to obtain a new access token when the old one expires, without forcing the user to log in again — it's stored more securely (httpOnly cookie) and can be revoked server-side.

```js
const accessToken = jwt.sign({ id: user.id }, ACCESS_SECRET, { expiresIn: '15m' });
const refreshToken = jwt.sign({ id: user.id }, REFRESH_SECRET, { expiresIn: '7d' });
res.cookie('refreshToken', refreshToken, { httpOnly: true, secure: true });
```

### Q178. What are common JWT vulnerabilities, and how do you avoid them?
1) Accepting `alg: none` (an attacker crafts an unsigned token) — always explicitly whitelist allowed algorithms when verifying. 2) Storing JWTs in `localStorage` (vulnerable to XSS theft) — prefer httpOnly cookies. 3) Not setting/checking expiration. 4) Using a weak/guessable secret for HS256 signing.

```js
// Explicitly restrict allowed algorithms
jwt.verify(token, SECRET, { algorithms: ['HS256'] }); // rejects "none" or unexpected algs
```

### Q179. What is the difference between stateless and stateful authentication, and which scales better horizontally?
Stateless auth (JWT) requires no server-side session storage — any server instance can verify the token independently, making it naturally horizontally scalable. Stateful auth (session-based) requires the server to look up session data, which needs a shared store (like Redis) across instances, or "sticky sessions" pinning a user to one server — extra infrastructure to scale.

```js
// Stateless — works on any instance, no shared storage needed
const user = jwt.verify(token, SECRET);
```

### Q180. What is OWASP, and what are a few of its Top 10 risks relevant to Node.js APIs?
OWASP (Open Web Application Security Project) publishes the "Top 10" list of the most critical web security risks. Relevant ones for Node APIs: Injection (SQL/NoSQL), Broken Authentication, Sensitive Data Exposure (not encrypting data at rest/in transit), Broken Access Control (missing authorization checks), Security Misconfiguration (default configs, verbose error messages leaking stack traces), and Using Components with Known Vulnerabilities (outdated npm packages).

```bash
npm audit          # scan for known vulnerable dependencies
npm audit fix
```

### Q181. What is `npm audit`, and how do you handle vulnerabilities it finds in a production Node app?
It scans your dependency tree against a database of known security vulnerabilities and reports severity levels. You handle it by running `npm audit fix` (auto-updates to patched versions where possible), manually upgrading packages with breaking changes, or evaluating the actual risk/exploitability if a fix isn't yet available.

```bash
npm audit --production   # check only production dependencies
npm audit fix --force    # applies even breaking-change updates (use cautiously)
```

### Q182. What is dependency confusion, and how can it affect a Node/npm project?
It's an attack where a malicious package is published to the public npm registry using the SAME name as an internal/private package your company uses — if your build config isn't set up correctly, npm might install the malicious public version instead of your intended private one. Mitigation: use scoped package names (`@yourcompany/pkg`) and explicitly configure your registry/scope mapping.

```json
// .npmrc — pin your scope to your private registry
"@yourcompany:registry=https://npm.yourcompany.com"
```

### Q183. What is Content Security Policy (CSP), and how does it protect a Node-served web app?
CSP is an HTTP response header that tells the browser which sources (scripts, styles, images, etc.) are allowed to load/execute on the page — it's a strong defense against XSS, since even if an attacker injects a `<script>` tag, the browser will refuse to run it if it violates the policy.

```js
app.use((req, res, next) => {
  res.setHeader('Content-Security-Policy', "default-src 'self'; script-src 'self'");
  next();
});
```

### Q184. What is the difference between `Access-Control-Allow-Origin: *` and specifying an explicit origin in CORS?
`*` allows ANY website to make cross-origin requests to your API — convenient for fully public APIs but dangerous if the API returns sensitive user-specific data, since it can't be combined with credentials (cookies). Specifying an explicit origin (or a validated list) restricts access to only trusted domains and allows credentialed requests.

```js
app.use(cors({
  origin: ['https://myapp.com', 'https://admin.myapp.com'],
  credentials: true // only works with a specific origin, not '*'
}));
```

### Q185. What is Server-Sent Events (SSE), and how does it differ from WebSockets?
SSE is a one-way (server → client only) persistent connection over plain HTTP, using the `text/event-stream` content type — simpler than WebSockets (no special protocol upgrade, works over regular HTTP/2 multiplexing, auto-reconnects), but can't send data client → server over the same connection. Good fit for live feeds/notifications where the client doesn't need to push data back.

```js
app.get('/events', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  setInterval(() => res.write(`data: ${Date.now()}\n\n`), 1000);
});
```

### Q186. What is database sharding, and why might a Node backend need to support it?
Sharding splits a large dataset across multiple database servers (shards), each holding a subset of the data (e.g. by user ID range or region), so no single database has to handle the entire dataset/load alone. The application layer (Node) often needs shard-aware routing logic to know which shard to query for a given piece of data.

```js
function getShardForUser(userId) {
  const shardIndex = userId % NUM_SHARDS;
  return shardConnections[shardIndex];
}
```

### Q187. What is database replication, and what's the difference between a primary and a replica?
Replication copies data from a primary (master) database to one or more replica (secondary) databases in near real-time. Writes go to the primary; reads can be distributed across replicas to reduce load on the primary — but replicas can lag slightly behind (eventual consistency), so reading right after a write might return stale data unless you explicitly read from the primary.

```js
// Mongoose read preference — route reads to replicas
mongoose.connect(uri, { readPreference: 'secondaryPreferred' });
```

### Q188. What is eventual consistency, and where does it show up in Node microservice systems?
It means that after a write, all parts of a distributed system will EVENTUALLY reflect that change, but not necessarily immediately — some reads shortly after a write might return stale data. It commonly shows up when using database replicas, caches (Redis), or event-driven microservices where one service updates its own data and asynchronously notifies others via events.

```js
// Service A updates immediately, Service B's copy updates a moment later via an event
await orderService.updateStatus(orderId, 'shipped');
eventBus.emit('order.shipped', { orderId }); // notification service catches up asynchronously
```

### Q189. What is the Saga pattern, and why is it used instead of distributed transactions in microservices?
A Saga breaks a business transaction that spans multiple services into a sequence of local transactions, each publishing an event that triggers the next step. If a step fails, previously completed steps run "compensating" actions to undo their effect — since true distributed ACID transactions across independent microservices/databases are impractical at scale.

```js
// Orchestration-style saga (simplified)
async function createOrderSaga(order) {
  try {
    await paymentService.charge(order);
    await inventoryService.reserve(order);
  } catch (err) {
    await paymentService.refund(order); // compensating action
    throw err;
  }
}
```

### Q190. What is CQRS (Command Query Responsibility Segregation)?
It's a pattern that separates the model used for WRITING data (commands — optimized for validation/business rules) from the model used for READING data (queries — often a denormalized, read-optimized view, sometimes even a different database) — useful when read and write workloads/patterns are very different at scale.

```js
// Write side — normalized, transactional
await orderCommandService.createOrder(data);
// Read side — denormalized, fast, maybe served from a cache/materialized view
const summary = await orderQueryService.getOrderSummary(orderId);
```

### Q191. What is Event Sourcing, and how does it differ from traditional state storage?
Instead of storing just the current state of an entity, Event Sourcing stores every STATE-CHANGING EVENT that ever happened to it (e.g. "OrderCreated", "ItemAdded", "OrderShipped") — the current state is derived by replaying all events. This gives a full audit trail and lets you reconstruct state at any point in time, at the cost of more complex querying.

```js
const events = [
  { type: 'OrderCreated', data: {...} },
  { type: 'ItemAdded', data: {...} }
];
const currentState = events.reduce(applyEvent, initialState);
```

### Q192. What is gRPC, and when would you choose it over REST for Node microservices?
gRPC is a high-performance RPC framework using HTTP/2 and Protocol Buffers (a compact binary format) instead of JSON over plain HTTP/1.1 — much faster serialization and smaller payloads, plus strongly-typed contracts (`.proto` files) and streaming support. Preferred for internal service-to-service communication where performance matters; REST/JSON remains more common for public-facing APIs due to simplicity/tooling/browser support.

```proto
// user.proto
service UserService {
  rpc GetUser (UserRequest) returns (UserResponse);
}
```

### Q193. What is an API Gateway, and what role does it play in a microservices architecture?
It's a single entry point that sits in front of all your microservices, handling cross-cutting concerns centrally: routing requests to the right service, authentication, rate limiting, request/response transformation, and aggregating responses from multiple services — so individual services don't each need to reimplement these concerns.

```
Client → API Gateway (auth, rate limit, routing) → [Order Service] [User Service] [Payment Service]
```

### Q194. What is service discovery, and why do microservices need it?
In dynamic environments (containers/cloud), service instances' IP addresses/ports change constantly as they scale up/down or get rescheduled. Service discovery (via tools like Consul, etcd, or Kubernetes' built-in DNS) lets services find each other by NAME instead of hardcoded addresses, automatically updating as instances come and go.

```js
// Instead of a hardcoded IP, resolve by service name (e.g. via Kubernetes DNS)
const res = await fetch('http://order-service.default.svc.cluster.local/orders');
```

### Q195. What is a distributed lock, and why might a Node app need one (e.g. with Redis)?
When multiple instances of a Node app run concurrently, a distributed lock ensures only ONE instance can execute a critical section of code at a time (e.g. processing a scheduled job, or preventing duplicate order processing) — implemented using an atomic operation in a shared store like Redis (`SET key value NX EX ttl`).

```js
const lock = await redis.set('job:lock', 'locked', { NX: true, EX: 30 });
if (lock) {
  try { await runScheduledJob(); }
  finally { await redis.del('job:lock'); }
}
```

### Q196. What is a cron job, and how do you schedule recurring tasks in Node?
A cron job runs a task on a recurring schedule (e.g. "every day at midnight"). In Node, you use a library like `node-cron` or `node-schedule` since there's no built-in scheduler, defining the schedule with cron syntax (minute, hour, day, month, weekday).

```js
const cron = require('node-cron');
cron.schedule('0 0 * * *', () => { // every day at midnight
  console.log('Running daily cleanup job');
});
```

### Q197. What is the difference between running scheduled jobs inside your Node app vs using an external scheduler (e.g. Kubernetes CronJob)?
Running jobs inside the app is simple but risky in a clustered/multi-instance deployment — EVERY instance would trigger the same job simultaneously (duplicate execution) unless you add locking. An external scheduler triggers a SEPARATE, dedicated process/pod on schedule, naturally avoiding duplication and decoupling scheduling from your main app's uptime.

```yaml
# Kubernetes CronJob runs a one-off job container on schedule, not duplicated per app replica
schedule: "0 0 * * *"
```

### Q198. What is observability, and what are its three pillars in a Node production system?
Observability is the ability to understand a system's internal state from its external outputs. The three pillars: **Logs** (discrete event records), **Metrics** (numeric measurements over time, like request rate/error rate/latency), **Traces** (following a single request's journey across multiple services). Together they let you diagnose issues you didn't anticipate in advance.

```js
// Example: a request trace ID tying logs together across services
logger.info('Processing order', { traceId: req.traceId, orderId });
```

### Q199. What is distributed tracing, and how does OpenTelemetry relate to Node.js?
Distributed tracing follows a single request as it flows through multiple services, showing exactly where time was spent at each hop (a "span"). OpenTelemetry is a vendor-neutral standard/SDK for Node (and other languages) to automatically instrument your app and export these traces to backends like Jaeger, Zipkin, or Datadog.

```js
const { NodeSDK } = require('@opentelemetry/sdk-node');
const sdk = new NodeSDK({ /* exporters, instrumentations */ });
sdk.start();
```

### Q200. What is an APM (Application Performance Monitoring) tool, and what does it track in Node apps?
APM tools (New Relic, Datadog, Dynatrace) automatically instrument your app to track response times, throughput, error rates, slow database queries, and external call latency, giving you dashboards and alerts without manually adding logging everywhere.

```js
// Typically just requiring the agent at the very top of your entry file
require('newrelic');
const app = require('./app');
```

### Q201. What is a memory-mapped file, and does Node support it?
A memory-mapped file maps a file directly into a process's address space, so reading/writing the file looks like reading/writing memory, avoiding explicit read/write syscalls for each access. Node doesn't have this built into core, but native addons (or packages that wrap OS `mmap`) can provide it — mostly relevant for very high-performance, low-level use cases.

```js
// Not core Node — typically via a native addon package like 'mmap-io'
```

### Q202. What is a native addon in Node.js, and when would you write one?
A native addon is a compiled module (usually written in C/C++, or via N-API/Rust) that Node can `require()` like a normal JS module, used when you need raw performance beyond what JS provides, or need to interface with existing native libraries (image/video processing, hardware access, cryptography primitives).

```js
// binding.gyp defines the native build; then in JS:
const addon = require('bindings')('my_addon');
console.log(addon.hello());
```

### Q203. What is N-API, and why was it introduced?
N-API (Node-API) is a stable, ABI-stable C API for building native addons, independent of the underlying V8 engine version — before N-API, addons had to be recompiled for every Node/V8 version. N-API addons work across Node versions without recompilation, making them much easier to distribute/maintain.

```c
// N-API addons compile once and generally keep working across Node major versions,
// unlike older NAN/V8-direct addons which broke on V8 API changes.
```

### Q204. What is the difference between `process.binding()` (deprecated) and public core modules?
`process.binding()` exposed Node's internal, undocumented C++ bindings directly to JS — fast but unstable and unsupported, since internals could change/break without warning between versions. It's deprecated/restricted in modern Node; you should always use the public, documented core modules (`fs`, `crypto`, etc.) instead, which provide a stable contract.

```js
// Deprecated/dangerous — relies on unstable internals
// process.binding('fs')

// Correct — stable public API
const fs = require('fs');
```

### Q205. What is a zero-downtime deployment, and how do you achieve it for a Node app?
It means deploying a new version of your app without any moment where requests fail or the service is unavailable — achieved via rolling deployments (start new instances, wait until healthy, THEN kill old ones), blue-green deployments (switch traffic between two full environments), or PM2's `reload` (graceful worker-by-worker restart in cluster mode).

```bash
pm2 reload app --update-env  # restarts workers one at a time, never all down together
```

### Q206. What is blue-green deployment?
You maintain two identical production environments ("blue" = currently live, "green" = new version). You deploy and fully test the new version on "green" while "blue" still serves live traffic, then switch the router/load balancer to point to "green" instantly — giving instant rollback (just switch back) if something's wrong.

```
Router → [Blue: v1.0 (live)]
       ↘ [Green: v1.1 (staged, tested)] — switch router here when ready
```

### Q207. What is canary deployment, and how does it differ from blue-green?
Canary deployment gradually routes a small percentage of real traffic (e.g. 5%) to the new version while most traffic still goes to the old version, monitoring for errors before slowly increasing the percentage — reducing blast radius if something's wrong. Blue-green switches ALL traffic at once; canary is a gradual, incremental rollout.

```
95% traffic → v1.0 (stable)
5% traffic  → v1.1 (canary) — watch error rates, then increase gradually
```

### Q208. What is a feature flag, and how is it typically implemented in a Node backend?
A feature flag lets you toggle a feature on/off (or for specific users/percentages) at runtime WITHOUT deploying new code — useful for gradual rollouts, A/B testing, or quickly disabling a broken feature. Implemented via a config service, database flag, or a dedicated tool (LaunchDarkly, Unleash) checked at the relevant code path.

```js
if (await featureFlags.isEnabled('new-checkout-flow', req.user.id)) {
  return newCheckoutHandler(req, res);
}
return oldCheckoutHandler(req, res);
```

### Q209. What is the difference between vertical and horizontal Pod/instance autoscaling for a Node app in Kubernetes?
Vertical autoscaling increases the CPU/memory allocated to EXISTING pod(s). Horizontal autoscaling (HPA) increases the NUMBER of pod replicas based on metrics like CPU usage or request rate — horizontal scaling is generally preferred for stateless Node apps since it also improves availability (more independent instances).

```yaml
# Kubernetes HorizontalPodAutoscaler
minReplicas: 2
maxReplicas: 10
targetCPUUtilizationPercentage: 70
```

### Q210. What does it mean for a Node.js app to be "stateless," and why does it matter for scaling?
A stateless app doesn't store any request-specific or user session data in its own local memory/disk between requests — any instance can handle any request. This matters because it lets you freely add/remove/restart instances behind a load balancer without losing data or breaking users mid-session; state instead lives in shared external stores (Redis, a database).

```js
// Stateful (bad for scaling): session stored in local memory
const sessions = {}; // lost if this instance restarts, invisible to other instances

// Stateless (good): session stored in Redis, any instance can read it
app.use(session({ store: new RedisStore({ client: redisClient }) }));
```

### Q211. What is the difference between `Docker` image layers and how does it affect Node build speed?
A Docker image is built in layers, each corresponding to a command in the Dockerfile, and Docker caches unchanged layers. Ordering your Dockerfile so `package.json`/`package-lock.json` are copied and `npm install` runs BEFORE copying the rest of your source code means dependency installation is cached and skipped on rebuilds unless dependencies actually changed — much faster iterative builds.

```dockerfile
COPY package*.json ./
RUN npm ci --production   # cached unless package*.json changes
COPY . .                  # source code changes don't invalidate the npm install layer
CMD ["node", "server.js"]
```

### Q212. What is a multi-stage Docker build, and why use one for a Node app?
It uses multiple `FROM` stages in one Dockerfile — an earlier stage installs devDependencies and builds/compiles the app (e.g. TypeScript → JS), and the FINAL stage copies only the compiled output and production dependencies into a clean, smaller image — keeping devDependencies and build tools out of your production image.

```dockerfile
FROM node:20 AS build
COPY . .
RUN npm ci && npm run build

FROM node:20-slim
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]
```

### Q213. What is a `Dockerfile` `HEALTHCHECK`, and why add one for a Node app's container?
It defines a command Docker (or Kubernetes) runs periodically to check if the container is actually healthy, not just "running" — a Node process can technically stay alive while its event loop is stuck or its DB connection is dead. A failing health check lets the orchestrator restart or stop routing to that container automatically.

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s CMD curl -f http://localhost:3000/health || exit 1
```

### Q214. What is a serverless function (like AWS Lambda), and how does writing Node.js for it differ from a normal Express server?
A serverless function runs your code on-demand per request/event, with the cloud provider managing the underlying servers, auto-scaling to zero when idle. In Node, instead of a long-running `app.listen()` process, you export a single handler function invoked per event, and you must account for "cold starts" (extra latency the first time a function spins up) and avoid relying on in-memory state persisting between invocations.

```js
// AWS Lambda handler
exports.handler = async (event) => {
  return { statusCode: 200, body: JSON.stringify({ message: 'Hello from Lambda' }) };
};
```

### Q215. What is a cold start in serverless Node functions, and how do you minimize it?
A cold start is the extra latency incurred when a serverless platform has to spin up a fresh execution environment (load runtime, initialize your code) before it can process a request, versus reusing an already-"warm" instance. Minimize it by keeping deployment package size small, minimizing heavy top-level imports/initialization, choosing a lighter runtime configuration, or using "provisioned concurrency" to keep instances pre-warmed.

```js
// Move expensive setup (DB connection) OUTSIDE the handler so it's reused across warm invocations
const dbConnection = createConnection(); // runs once per warm container, not per request
exports.handler = async (event) => { return dbConnection.query(...); };
```

### Q216. What is the difference between a Docker container and a virtual machine (VM), relevant to deploying Node apps?
A VM virtualizes an entire OS (its own kernel, full OS overhead) on top of a hypervisor — heavier, slower to start. A Docker container shares the HOST machine's kernel and only packages the application plus its dependencies — much lighter, starts in seconds instead of minutes, and is the standard way to package/deploy Node apps consistently across environments.

```dockerfile
FROM node:20-alpine  # lightweight base image sharing the host's kernel
```

### Q217. What is a race condition in Node.js, and can you give an async example?
A race condition happens when the outcome of code depends on the unpredictable timing/order of async operations. Even though Node is single-threaded, race conditions still occur with interleaved async operations — e.g. two concurrent requests both reading a value, then both writing an updated value, causing one update to silently overwrite the other ("lost update").

```js
// Race condition: two concurrent requests both read balance=100, both add 10, final = 110 instead of 120
async function deposit(userId, amount) {
  const user = await User.findById(userId);      // both requests read 100 here
  user.balance += amount;
  await user.save();                              // second save overwrites the first
}
// Fix: use an atomic update instead
await User.updateOne({ _id: userId }, { $inc: { balance: amount } });
```

### Q218. What is an atomic operation, and why prefer them over "read-then-write" logic in Node/DB code?
An atomic operation completes as a single, indivisible step from the database's perspective — no other operation can interleave in the middle of it. Using atomic update operators (like MongoDB's `$inc`, `$push`, or SQL's `UPDATE ... SET x = x + 1`) instead of "read value, modify in app, write back" avoids race conditions entirely, since the database guarantees consistency for that single operation.

```js
// Not atomic — vulnerable to race conditions
const doc = await Counter.findOne(); doc.count++; await doc.save();
// Atomic — safe under concurrent requests
await Counter.updateOne({}, { $inc: { count: 1 } });
```

### Q219. What is optimistic locking vs pessimistic locking, and how would you implement optimistic locking in Node?
Pessimistic locking blocks other operations from touching a record while one is being processed (e.g. a DB row lock) — safe but can hurt throughput. Optimistic locking assumes conflicts are rare: it allows concurrent reads, but on write, checks whether the record changed since it was read (usually via a version number) — if it changed, the write is rejected and retried.

```js
// Mongoose has built-in optimistic concurrency via versionKey (__v)
const doc = await Model.findById(id);
doc.value = newValue;
await doc.save(); // fails with VersionError if another process updated it first
```

### Q220. What is the difference between `structuredClone()`, `JSON.parse(JSON.stringify())`, and `lodash.cloneDeep()` for deep cloning?
`JSON.parse(JSON.stringify(obj))` is a common hack but loses `undefined`, functions, `Date` objects (become strings), `Map`/`Set`, and breaks on circular references. `structuredClone()` (native, modern Node) correctly handles most built-in types (Dates, Maps, Sets, circular refs) but still can't clone functions. `lodash.cloneDeep()` handles nearly everything including some custom objects, at the cost of an extra dependency.

```js
const obj = { date: new Date(), map: new Map([['a', 1]]) };
JSON.parse(JSON.stringify(obj)); // date becomes a string, map becomes {}
structuredClone(obj);            // date and map preserved correctly
```

### Q221. What is a "hot path" in performance optimization, and how do you identify one in a Node app?
A hot path is code that executes extremely frequently (e.g. a function called on every single request, or inside a tight loop) — even tiny inefficiencies here multiply into significant overall impact. You identify hot paths using a CPU profiler (flame graphs show which functions consume the most cumulative time) rather than guessing.

```bash
node --prof server.js
# generate load, then:
node --prof-process isolate-*.log | less   # shows top time-consuming functions
```

### Q222. What is V8's hidden classes / inline caching optimization, and how can your JS code accidentally defeat it?
V8 creates internal "hidden classes" for objects with the same shape (same properties, added in the same order) so it can access properties fast via memory offsets instead of a slow dictionary lookup ("inline caching"). Code that creates objects with inconsistent shapes (adding properties conditionally, in different orders, or deleting properties) forces V8 to fall back to slower lookups.

```js
// BAD: inconsistent shapes defeat inline caching
function Point(x, y) { this.x = x; if (y) this.y = y; } // sometimes has y, sometimes not

// GOOD: consistent shape every time
function Point(x, y) { this.x = x; this.y = y || 0; }
```

### Q223. What is monomorphic vs polymorphic function calls in V8, and why does it matter for performance?
A monomorphic call site always receives the same object shape/type — V8 can optimize it heavily (inline caching hits every time). A polymorphic (or worse, megamorphic) call site receives many DIFFERENT shapes/types over time, forcing V8 to fall back to slower generic lookup paths instead of the optimized fast path.

```js
// Monomorphic (fast): always called with the same shape of object
function getX(point) { return point.x; }
getX({x: 1, y: 2}); getX({x: 3, y: 4}); // same shape every time

// Megamorphic (slow): called with wildly different shapes
getX({x: 1}); getX({x: 1, y: 2, z: 3}); getX({x: 1, label: 'a'});
```

### Q224. What is the difference between `Object.keys()`, `Object.values()`, `Object.entries()`, and `for...in`, performance-wise?
`Object.keys/values/entries()` only iterate over the object's OWN enumerable properties (not inherited ones) and return a fresh array. `for...in` iterates over ALL enumerable properties INCLUDING inherited ones from the prototype chain, which is usually not what you want and is generally slower — most modern code prefers `Object.keys()`/`entries()` with `.forEach()`/`for...of` for clarity and correctness.

```js
class Base { constructor() { this.a = 1; } }
Base.prototype.inherited = 'x';
const obj = new Base();
Object.keys(obj);         // ['a'] — own properties only
for (const k in obj) {}   // 'a', 'inherited' — includes prototype chain
```

### Q225. What is a flame graph, and how do you read one to find a Node.js performance bottleneck?
A flame graph is a visualization of a CPU profile — each horizontal bar is a function call, stacked vertically to show the call hierarchy (parent functions on top, functions they call below), and the WIDTH of each bar represents how much total time was spent in that function (including its children). Wide bars near the top of the stack (or wide leaf/bottom bars) point to where optimization effort should focus.

```bash
# Generate a flame graph with clinic.js
npx clinic flame -- node server.js
# then send load to the server, stop it, and it opens an interactive flame graph
```


---

# PART 4: SUPER ADVANCED (Q226–300)

### Q226. Walk through exactly what happens internally when you call `fs.readFile()` in Node.js.
1) Your JS call goes into Node's `fs` binding layer. 2) Since file I/O lacks reliable native async OS support, libuv queues the actual read operation onto its thread pool (default 4 threads). 3) A worker thread picks it up and performs the blocking read at the OS level. 4) Once done, the worker thread signals completion back to the event loop via libuv's internal mechanism. 5) On the next poll phase, your callback is invoked on the MAIN thread with the result — the actual disk read happened off-thread, but your callback still runs safely on the single JS thread.

```js
fs.readFile('file.txt', (err, data) => {
  // This callback runs back on the main thread,
  // even though the disk read itself happened on a libuv worker thread
});
```

### Q227. What is the exact difference between how `setImmediate()` and `setTimeout(fn, 0)` are ordered when called from the TOP-LEVEL of your script vs from WITHIN an I/O callback?
From the top level (main module), the order is NOT guaranteed — it depends on process performance/timing since the timers phase runs before the check phase, but the exact 1ms+ threshold for a 0ms timeout can go either way. From WITHIN an I/O callback (inside the poll phase), `setImmediate()` is ALWAYS guaranteed to fire before `setTimeout(fn, 0)`, because the check phase (setImmediate) directly follows the poll phase, while timers requires looping back around to the timers phase.

```js
// Top level — order can vary
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));

// Inside I/O callback — immediate ALWAYS wins
fs.readFile(__filename, () => {
  setTimeout(() => console.log('timeout'), 0);
  setImmediate(() => console.log('immediate')); // guaranteed first
});
```

### Q228. How does V8 JIT (Just-In-Time) compilation work, and what are the roles of Ignition and TurboFan?
V8 doesn't purely interpret JS — **Ignition** is V8's bytecode interpreter, which first quickly compiles JS to bytecode and starts running it right away (fast startup). While running, V8 profiles "hot" functions (called frequently with consistent argument types). Those get handed to **TurboFan**, the optimizing JIT compiler, which compiles them down to highly optimized machine code. If TurboFan's assumptions about types turn out wrong later (a "deopt"), V8 falls back to the slower Ignition bytecode again.

```js
// A function called millions of times with consistent number types
// gets "optimized" by TurboFan into fast machine code over time
function add(a, b) { return a + b; }
for (let i = 0; i < 10_000_000; i++) add(i, i+1);
```

### Q229. What is a "deoptimization" in V8, and what commonly triggers it?
A deoptimization happens when TurboFan's optimized machine code makes an assumption (e.g. "this argument is always a number") that turns out to be violated at runtime, forcing V8 to throw away the optimized code and fall back to slower bytecode execution, then possibly re-optimize later with new assumptions. Common triggers: calling a function with inconsistent argument types, using `arguments` object unusually, or triggering an exception in an unexpected spot inside a hot function.

```js
function add(a, b) { return a + b; }
for (let i = 0; i < 100000; i++) add(1, 2);     // optimized for numbers
add("1", "2"); // suddenly called with strings — can trigger a deopt
```

### Q230. What is the Temporal Dead Zone (TDZ), and why does it exist for `let`/`const`?
The TDZ is the period between entering a block scope and the actual `let`/`const` declaration line, during which the variable exists but CANNOT be accessed — accessing it throws a `ReferenceError` rather than silently returning `undefined` (as `var` would). It exists to catch bugs early — using a variable before its intended initialization is almost always a mistake.

```js
console.log(x); // ReferenceError: Cannot access 'x' before initialization
let x = 5;
```

### Q231. What is the difference between how `Promise` microtasks and `queueMicrotask()` interact with `process.nextTick()`'s queue in Node?
Node maintains TWO separate microtask-like queues internally: the `process.nextTick()` queue and the Promise/`queueMicrotask()` queue. After every operation, Node fully drains the `nextTick` queue FIRST, then fully drains the Promise/microtask queue — and this happens BEFORE moving to the next event loop phase. So `process.nextTick()` always has strict priority over Promise callbacks.

```js
Promise.resolve().then(() => console.log('promise 1'));
process.nextTick(() => console.log('nextTick 1'));
Promise.resolve().then(() => console.log('promise 2'));
process.nextTick(() => console.log('nextTick 2'));
// Output: nextTick 1, nextTick 2, promise 1, promise 2
```

### Q232. What is the risk of using `process.nextTick()` recursively, and how can it starve the event loop?
Since `process.nextTick()`'s queue is FULLY drained before the event loop can proceed to I/O/timers, if a `nextTick` callback keeps scheduling MORE `nextTick` calls recursively, the event loop can never move forward — I/O, timers, and everything else gets starved indefinitely ("I/O starvation").

```js
// DANGEROUS: this will hang the process, starving all I/O forever
function recurse() { process.nextTick(recurse); }
recurse();
```

### Q233. What is the difference between how Node.js implements `EventEmitter` and how it handles listener limits/memory warnings?
`EventEmitter` warns (via a console warning) if more than 10 listeners are attached to the SAME event on the SAME emitter by default — this is a heuristic to catch likely memory leaks (e.g. attaching a new listener on every request instead of once). You can adjust it with `setMaxListeners()` if genuinely needed, but it's usually a sign of an actual bug.

```js
const emitter = new (require('events'))();
emitter.setMaxListeners(20); // raise the warning threshold if truly needed
// MaxListenersExceededWarning fires past the default of 10 otherwise
```

### Q234. How would you implement a custom Transform stream that uppercases text chunks passing through it?
Extend `stream.Transform` and implement `_transform(chunk, encoding, callback)`, calling `callback(null, transformedChunk)` to push the processed data downstream. This lets you build reusable, pipeable data-processing stages.

```js
const { Transform } = require('stream');
class UppercaseTransform extends Transform {
  _transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  }
}
process.stdin.pipe(new UppercaseTransform()).pipe(process.stdout);
```

### Q235. What is `stream.pipeline()`, and why is it preferred over manually chaining `.pipe()` calls?
`stream.pipeline()` chains multiple streams together (like `.pipe()`) but ALSO properly handles errors and cleanup across the WHOLE chain — if any stream in the chain errors or closes, all the others are automatically destroyed/cleaned up. Manual `.pipe()` chains don't propagate errors automatically (you'd need `.on('error', ...)` on every single stream), making it easy to leak resources on failure.

```js
const { pipeline } = require('stream');
pipeline(
  fs.createReadStream('input.txt'),
  zlib.createGzip(),
  fs.createWriteStream('output.txt.gz'),
  (err) => { if (err) console.error('Pipeline failed:', err); else console.log('Done'); }
);
```

### Q236. What is `stream.finished()` used for?
It's a utility that lets you get notified (via callback or Promise) when a stream is no longer readable/writable — whether it finished successfully, errored, or was prematurely closed — giving one unified place to handle cleanup instead of listening to multiple separate events (`end`, `finish`, `close`, `error`).

```js
const { finished } = require('stream/promises');
await finished(readStream); // resolves when stream ends, rejects if it errors
```

### Q237. What is object mode in Node streams, and when do you need it?
By default, streams work with `Buffer`/string chunks. "Object mode" (`{ objectMode: true }`) allows a stream to push/consume ANY JavaScript value (objects, numbers, arrays) instead of just binary/string data — useful for building data-processing pipelines of structured records (e.g. parsed CSV rows) rather than raw bytes.

```js
const { Transform } = require('stream');
const parseRows = new Transform({
  objectMode: true,
  transform(row, enc, cb) { cb(null, { ...row, parsed: true }); }
});
```

### Q238. How do you implement a rate limiter algorithm (token bucket) manually in Node without a library?
A token bucket holds a maximum number of tokens, refilled at a fixed rate over time; each request consumes one token, and if none are available, the request is rejected/delayed. This allows brief bursts up to the bucket size while enforcing an average rate over time.

```js
class TokenBucket {
  constructor(capacity, refillRatePerSec) {
    this.capacity = capacity; this.tokens = capacity;
    this.refillRate = refillRatePerSec; this.lastRefill = Date.now();
  }
  tryConsume() {
    const now = Date.now();
    const elapsed = (now - this.lastRefill) / 1000;
    this.tokens = Math.min(this.capacity, this.tokens + elapsed * this.refillRate);
    this.lastRefill = now;
    if (this.tokens >= 1) { this.tokens -= 1; return true; }
    return false;
  }
}
```

### Q239. What is the sliding window algorithm for rate limiting, and how does it improve on a fixed window?
A fixed window (e.g. "max 100 requests per minute, resetting every minute") can allow a burst of 200 requests right around the boundary (100 at the end of one window, 100 at the start of the next). A sliding window tracks requests within a continuously moving time frame (often approximated using a weighted count of the current and previous window), smoothing out this boundary-burst problem.

```js
// Sliding window log approach with Redis sorted sets (simplified)
const now = Date.now();
await redis.zRemRangeByScore(key, 0, now - windowMs); // drop old entries
const count = await redis.zCard(key);
if (count < maxRequests) await redis.zAdd(key, { score: now, value: `${now}` });
```

### Q240. What is the difference between how Node.js implements `process.hrtime()`/`process.hrtime.bigint()` vs `Date.now()`, and why does it matter for benchmarking?
`Date.now()` returns wall-clock time in milliseconds, which can be affected by system clock adjustments (NTP sync, manual changes) — unreliable for measuring short precise durations. `process.hrtime.bigint()` returns a monotonic, high-resolution time in nanoseconds that's NOT affected by system clock changes, making it the correct choice for accurately measuring elapsed time/performance benchmarks.

```js
const start = process.hrtime.bigint();
doExpensiveWork();
const end = process.hrtime.bigint();
console.log(`Took ${(end - start) / 1_000_000n} ms`);
```

### Q241. What is the `perf_hooks` module used for?
It provides access to high-resolution performance metrics within Node — you can create marks and measures (similar to the browser's Performance API), and it also reports Node/V8-specific timing data (like event loop delay, GC pause duration) for detailed performance analysis.

```js
const { performance, PerformanceObserver } = require('perf_hooks');
performance.mark('start');
doWork();
performance.mark('end');
performance.measure('doWork', 'start', 'end');
```

### Q242. How would you detect and monitor event loop delay/utilization in production Node apps?
Using the `perf_hooks` module's `monitorEventLoopDelay()` (gives statistical percentiles of loop delay over time) or `performance.eventLoopUtilization()` (fraction of time the loop spent doing actual work vs idle) — both let you set up alerting if the event loop starts consistently lagging under load, without manually implementing a timer-diff check.

```js
const { monitorEventLoopDelay } = require('perf_hooks');
const histogram = monitorEventLoopDelay({ resolution: 20 });
histogram.enable();
setInterval(() => console.log('p99 delay (ms):', histogram.percentile(99) / 1e6), 5000);
```

### Q243. What is the difference between `Domain` (legacy, deprecated) and modern error handling approaches in Node?
`domain` was an early API meant to group and catch errors across multiple async operations (including uncaught exceptions inside callbacks), but it had serious edge cases and memory implications, and is now deprecated. Modern Node favors explicit `try/catch` with `async/await`, proper Promise rejection handling, and `AsyncLocalStorage` for context — none of which have `domain`'s pitfalls.

```js
// Deprecated — avoid in new code
// const domain = require('domain');

// Modern approach
async function handler() {
  try { await riskyOp(); } catch (err) { logger.error(err); }
}
```

### Q244. What is a "hanging promise" and how do unhandled rejections lead to process crashes in modern Node versions?
A hanging/unhandled promise is one that rejects but has no `.catch()` (or try/catch around its `await`) anywhere in its chain. In modern Node (since v15), an unhandled promise rejection, BY DEFAULT, crashes the process (throws and terminates) rather than just printing a warning like older versions did — treating it with the same severity as an uncaught synchronous exception.

```js
process.on('unhandledRejection', (reason) => {
  console.error('Unhandled Rejection:', reason);
  // log/alert, then usually still exit gracefully — the app may be in a bad state
  process.exit(1);
});
```

### Q245. What is the difference between `Error.captureStackTrace()` and a regular `new Error()` stack trace, and why would you use it in a custom error class?
`Error.captureStackTrace(targetObject, constructorOpt)` builds a `.stack` property on the target object while EXCLUDING the frames from the constructor function passed as the second argument — used in custom error classes to hide the noisy internal error-construction frames, so the stack trace starts from where the error was actually thrown/created by the caller.

```js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    Error.captureStackTrace(this, this.constructor); // omit AppError's own constructor frame
  }
}
```

### Q246. What is the difference between exit codes `0`, `1`, and other conventions when calling `process.exit()`?
`0` conventionally means successful/clean exit. Any non-zero code (commonly `1`) signals an error occurred — process managers, CI pipelines, and shell scripts check this code to decide success/failure and whether to retry/restart. You can also use specific custom codes to distinguish different failure types.

```js
process.exit(0); // success
process.exit(1); // generic failure — e.g. process managers will treat this as a crash to restart
```

### Q247. Why is `process.exit()` generally discouraged inside async code, and what's the safer alternative?
`process.exit()` terminates immediately, potentially cutting off in-flight I/O (unflushed logs, unsent responses, unfinished DB writes) since it doesn't wait for the event loop to drain pending callbacks. The safer approach is to let the process exit naturally by closing all open handles/connections (server, DB pool) — Node exits on its own once there's nothing left keeping the event loop alive.

```js
// Instead of process.exit() immediately:
server.close(() => {
  db.disconnect(() => {
    // Now nothing is keeping the event loop alive — Node exits naturally, cleanly
  });
});
```

### Q248. What is the difference between `beforeExit` and `exit` events on `process`?
`beforeExit` fires when Node's event loop has no more work scheduled and is ABOUT to exit naturally — you CAN still schedule new async work here to keep the process alive (e.g. queue one more operation). `exit` fires synchronously right as the process is actually terminating — ONLY synchronous code can run here; any async operation scheduled inside it will never execute.

```js
process.on('beforeExit', (code) => console.log('About to exit with code:', code));
process.on('exit', (code) => console.log('Exiting now, code:', code)); // sync only!
```

### Q249. What is the difference between a "keep-alive handle" and how Node decides when to exit the process naturally?
Node tracks internal handles/references that keep the event loop "alive" — an open server socket, a pending timer, an active file descriptor. As long as ANY such handle exists, the event loop keeps running. Once ALL of them are closed/cleared (no pending timers, no open connections, no active listeners), Node has nothing left to do and exits on its own — this is why calling `server.close()` (and closing DB connections) lets a script terminate naturally instead of hanging forever.

```js
const timer = setInterval(() => {}, 1000); // keeps process alive indefinitely
clearInterval(timer); // now nothing keeps it alive — process can exit naturally
```

### Q250. What is `unref()` on a timer, and why would you use it?
Calling `.unref()` on a `Timeout` object tells Node "don't let this timer alone keep the process alive" — if it's the ONLY thing still pending, the process can exit even though the timer hasn't fired yet. Useful for background/heartbeat timers that shouldn't prevent a script from naturally ending.

```js
const timer = setInterval(() => console.log('heartbeat'), 1000);
timer.unref(); // this timer alone won't keep the process running
```

### Q251. What is the difference between `worker_threads` and `cluster` in terms of memory footprint and startup cost?
`cluster` forks entire new OS processes, each with its own V8 instance, memory heap, and event loop — higher memory overhead (each process might use tens of MB baseline) and slower startup, but complete isolation (a crash in one doesn't directly affect others). `worker_threads` run inside the SAME process, sharing the V8 instance's isolate infrastructure more efficiently, with lower per-worker overhead and the option of truly shared memory via `SharedArrayBuffer` — but a severe crash can be more likely to affect the whole process depending on the failure type.

```js
// cluster: heavier, full process isolation, best for scaling network I/O across cores
// worker_threads: lighter, shared-memory capable, best for CPU-bound parallel computation
```

### Q252. How would you design a Node.js worker pool to handle many CPU-intensive tasks without creating a new Worker Thread per task?
Creating a new `Worker` per task has real startup overhead (each spins up a new V8 isolate/thread). A worker pool pre-spawns a FIXED number of long-lived Worker threads, then distributes incoming tasks to whichever worker is currently free (via a task queue), reusing workers instead of constantly creating/destroying them — the `piscina` or `workerpool` libraries implement this pattern, or you can build it manually.

```js
const Piscina = require('piscina');
const pool = new Piscina({ filename: './task.js' });
const result = await pool.run({ data: 42 }); // reuses pooled workers automatically
```

### Q253. What is the difference between how `structuredClone`-based message passing works between Worker Threads vs `SharedArrayBuffer`-based sharing?
Regular `postMessage()` between a main thread and a Worker COPIES the data using the structured clone algorithm — safe (no shared mutable state, no race conditions) but costs time/memory proportional to data size, especially for large objects. `SharedArrayBuffer` shares the SAME underlying memory directly between threads with ZERO copying — much faster for large data, but requires manual synchronization (via `Atomics`) since both threads can read/write the same memory concurrently.

```js
// Copied (safe but slower for large data)
worker.postMessage({ largeArray });
// Shared (fast, zero-copy, but needs Atomics for safety)
worker.postMessage(sharedArrayBuffer);
```

### Q254. What is the "double free" or use-after-transfer problem when using Transferable objects (like `ArrayBuffer`) with `postMessage()`?
When you TRANSFER (not copy) an `ArrayBuffer` via `postMessage(data, [transferList])`, ownership of that memory moves to the receiving thread, and the ORIGINAL thread's reference becomes unusable ("neutered") — attempting to read/write it afterward throws an error. This is intentional (avoids the cost of copying) but is a common source of bugs if code doesn't realize the buffer was transferred, not copied.

```js
const buffer = new ArrayBuffer(1024);
worker.postMessage(buffer, [buffer]); // transferred, NOT copied
console.log(buffer.byteLength); // 0 — the original is now neutered/unusable
```

### Q255. How would you implement a custom async iterator in Node.js (e.g. to iterate over paginated API results lazily)?
Implement the `Symbol.asyncIterator` method on an object, returning an object with a `next()` method that returns a Promise resolving to `{ value, done }` — this lets you use `for await...of` to lazily consume items (like paginated results) one at a time without loading everything into memory upfront.

```js
class PaginatedResults {
  constructor(fetchPage) { this.fetchPage = fetchPage; this.page = 1; }
  async *[Symbol.asyncIterator]() {
    while (true) {
      const items = await this.fetchPage(this.page++);
      if (items.length === 0) return;
      for (const item of items) yield item;
    }
  }
}
for await (const item of new PaginatedResults(fetchFromApi)) { console.log(item); }
```

### Q256. What is the difference between a generator function and an async generator function in Node.js?
A regular generator (`function*`) produces a SYNCHRONOUS sequence of values one at a time via `yield`, paused/resumed with `.next()`. An async generator (`async function*`) does the same but each yielded value can itself be a Promise, and it's consumed with `for await...of` — combining lazy iteration with async operations (e.g. yielding results as they arrive from a stream or paginated API).

```js
async function* fetchPages() {
  let page = 1;
  while (true) {
    const data = await fetchPage(page++);
    if (!data.length) return;
    yield data;
  }
}
```

### Q257. How does Node.js's HTTP `Agent` and connection keep-alive work, and why does it matter for outgoing requests at scale?
An `http.Agent` manages connection reuse/pooling for OUTGOING requests your Node app makes to other servers — without keep-alive, every outgoing HTTP request opens a brand-new TCP connection (slow, expensive handshake), while a keep-alive agent reuses existing open connections for subsequent requests to the same host, dramatically reducing latency and file descriptor usage under high request volume.

```js
const https = require('https');
const agent = new https.Agent({ keepAlive: true, maxSockets: 50 });
https.get(url, { agent }, (res) => { /* reuses pooled connections */ });
```

### Q258. What is a file descriptor leak, and how can a Node.js app accidentally cause one?
A file descriptor (FD) is an OS-level handle to an open file, socket, or pipe — the OS has a limited number available per process. A leak happens when your code opens files/sockets/streams but never closes them (e.g. `fs.createReadStream()` without ever letting it finish/close, or forgetting to close DB/HTTP connections), eventually exhausting the OS limit and causing the app to fail to open ANY new connections ("EMFILE: too many open files").

```js
// LEAK: stream opened repeatedly, error path never closes it
function processFile(path) {
  const stream = fs.createReadStream(path);
  stream.on('error', () => {}); // forgot stream.destroy() here
}
```

### Q259. What is `ulimit`, and how does it relate to a Node.js server hitting "too many open files" errors in production?
`ulimit` is an OS-level setting controlling the maximum number of file descriptors (and other resources) a process can open. A production Node server handling many concurrent connections can hit the default limit (often 1024 on Linux) under load, causing `EMFILE` errors — the fix is raising the OS `ulimit` for the process (via systemd config, Docker settings, or the shell) alongside actually fixing any FD leaks in the app.

```bash
ulimit -n 65536   # raise the per-process open-file-descriptor limit
```

### Q260. What is the difference between how TCP and UDP sockets are handled in Node's `net` and `dgram` modules, and when would you use each?
`net` module handles TCP — reliable, ordered, connection-based (guarantees delivery and order, has more overhead) — used for things like custom protocols requiring reliability. `dgram` module handles UDP — connectionless, no delivery/order guarantees, much lower overhead — used for things like real-time streaming, DNS, or gaming where occasional lost packets are acceptable in exchange for speed.

```js
// TCP
const net = require('net');
const server = net.createServer((socket) => { socket.write('hello'); });

// UDP
const dgram = require('dgram');
const udpServer = dgram.createSocket('udp4');
udpServer.on('message', (msg, rinfo) => console.log(`Received: ${msg} from ${rinfo.address}`));
```

### Q261. How would you build a simple custom TCP-based protocol server in Node using the `net` module?
Use `net.createServer()`, which gives you a raw `socket` per connection — you read/write raw data directly, and typically need to implement your OWN message framing (since TCP is a byte stream with no built-in message boundaries) — e.g. prefixing each message with its length, or using a delimiter.

```js
const net = require('net');
const server = net.createServer((socket) => {
  socket.on('data', (data) => {
    const message = data.toString();
    socket.write(`Echo: ${message}`);
  });
});
server.listen(9000);
```

### Q262. What is TCP message framing, and why is it necessary when building custom protocols in Node?
TCP delivers a continuous stream of bytes with no inherent concept of "messages" — a single `write()` on one end might arrive split across multiple `data` events on the other end, or multiple writes might arrive coalesced into one `data` event. Message framing (e.g. prefixing each message with its length, or using a delimiter like `\n`) lets the receiver correctly reconstruct where one logical message ends and the next begins.

```js
// Length-prefixed framing (simplified)
function sendMessage(socket, obj) {
  const payload = Buffer.from(JSON.stringify(obj));
  const length = Buffer.alloc(4);
  length.writeUInt32BE(payload.length);
  socket.write(Buffer.concat([length, payload]));
}
```

### Q263. What is the difference between `process.env.NODE_ENV` values and how does Express (and other libraries) behave differently based on it?
`NODE_ENV` is a convention (not a Node built-in requirement) typically set to `development`, `production`, or `test`. Many libraries check it to change behavior — Express, for example, disables verbose error pages and enables view caching when `NODE_ENV=production`, and many logging/ORM libraries reduce verbosity or enable connection pooling optimizations in production mode.

```bash
NODE_ENV=production node server.js
```

```js
if (process.env.NODE_ENV === 'production') {
  app.use(compression()); // e.g. only compress in production
}
```

### Q264. What is the risk of trusting `NODE_ENV` alone for security-sensitive configuration, and what's a more robust approach?
Relying solely on `NODE_ENV` for things like enabling debug endpoints or verbose errors is risky because it's easy to accidentally leave it unset or misconfigured in a deployment, silently falling back to `development` defaults (which might expose stack traces or debug routes). A more robust approach uses EXPLICIT configuration flags (e.g. `DEBUG_MODE=false` set intentionally per environment) validated at startup, rather than inferring security-critical behavior from a loosely-conventioned variable.

```js
// Fail loudly instead of silently defaulting to an insecure mode
if (!process.env.NODE_ENV) throw new Error('NODE_ENV must be explicitly set');
```

### Q265. What is the difference between how Kubernetes liveness probes and readiness probes should be implemented differently for a Node app?
A liveness probe checks "is this process alive/functioning at all?" — if it fails repeatedly, Kubernetes RESTARTS the container. A readiness probe checks "is this instance ready to receive traffic right now?" (e.g. has it finished connecting to the DB?) — if it fails, Kubernetes stops ROUTING traffic to it WITHOUT restarting it. Conflating them (e.g. making liveness fail whenever a downstream DB is briefly slow) causes unnecessary restarts instead of just temporarily pausing traffic.

```js
app.get('/livez', (req, res) => res.status(200).send('alive')); // simple, always healthy unless truly stuck
app.get('/readyz', async (req, res) => {
  const dbOk = mongoose.connection.readyState === 1;
  res.status(dbOk ? 200 : 503).send(dbOk ? 'ready' : 'not ready');
});
```

### Q266. How would you implement graceful connection draining for a Node HTTP server during a rolling deployment?
On receiving `SIGTERM`: 1) Immediately mark the readiness probe as failing (so the load balancer stops sending NEW traffic). 2) Call `server.close()`, which stops accepting new connections but lets EXISTING in-flight requests finish. 3) Wait for those to complete (with a reasonable timeout, force-closing any that hang too long). 4) Then close DB connections and exit.

```js
let isShuttingDown = false;
app.get('/readyz', (req, res) => res.status(isShuttingDown ? 503 : 200).end());

process.on('SIGTERM', () => {
  isShuttingDown = true; // fail readiness immediately
  server.close(() => process.exit(0)); // finish in-flight requests, then exit
  setTimeout(() => process.exit(1), 10000); // force-kill if it takes too long
});
```

### Q267. What is idempotency key usage in payment/order APIs, and how do you implement it in Node?
An idempotency key is a unique client-generated identifier sent with a request (e.g. in a header), so that if the SAME request is accidentally sent twice (e.g. due to a network retry), the server recognizes the duplicate and returns the ORIGINAL result instead of processing the action (like a payment) a second time.

```js
app.post('/payments', async (req, res) => {
  const key = req.headers['idempotency-key'];
  const existing = await IdempotencyRecord.findOne({ key });
  if (existing) return res.json(existing.response); // return cached result, don't reprocess
  const result = await processPayment(req.body);
  await IdempotencyRecord.create({ key, response: result });
  res.json(result);
});
```

### Q268. What is the "thundering herd" problem in the context of Node caching, and how do you prevent it?
It happens when a popular cached value EXPIRES, and many concurrent requests simultaneously see a cache miss and all hit the underlying database/expensive resource at once, potentially overwhelming it. Prevention strategies: locking so only ONE request regenerates the cache while others wait for it (or serve stale data briefly), or staggering expiration times ("jitter") so not all cache entries expire at exactly the same moment.

```js
async function getCachedData(key, fetchFn) {
  const lockKey = `lock:${key}`;
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);
  const gotLock = await redis.set(lockKey, '1', { NX: true, EX: 10 });
  if (!gotLock) { await sleep(100); return getCachedData(key, fetchFn); } // wait & retry
  const fresh = await fetchFn();
  await redis.set(key, JSON.stringify(fresh), { EX: 300 });
  return fresh;
}
```

### Q269. How would you design a rate limiter that works correctly across MULTIPLE Node.js instances (not just one process)?
An in-memory rate limiter (a plain JS object counting requests) only works within a SINGLE process — with multiple instances behind a load balancer, each instance would track its own separate count, effectively multiplying the real allowed rate. You need a SHARED store (like Redis) that ALL instances read/write to atomically (using `INCR` + `EXPIRE`, or a Lua script for full atomicity), so the limit is enforced GLOBALLY across all instances.

```js
async function isAllowed(userId, limit, windowSec) {
  const key = `rate:${userId}`;
  const count = await redis.incr(key);
  if (count === 1) await redis.expire(key, windowSec);
  return count <= limit;
}
```

### Q270. What is the difference between designing a chat application's message delivery with "at-most-once," "at-least-once," and "exactly-once" semantics, and which is realistically achievable in Node systems?
At-most-once: a message might be lost but never duplicated (simplest, fire-and-forget). At-least-once: a message is guaranteed to arrive but might be delivered MORE than once (requires retries on the sender side) — the client/consumer must then handle deduplication. Exactly-once is the ideal but is EXTREMELY hard to guarantee in a truly distributed system (it usually means "effectively-once," achieved by combining at-least-once delivery with idempotent processing on the receiving end) — pure exactly-once delivery is generally considered practically unattainable across network boundaries.

```js
// Realistic approach: at-least-once delivery + idempotent consumer = "effectively exactly-once"
async function handleMessage(msg) {
  if (await alreadyProcessed(msg.id)) return; // dedupe on the consumer side
  await applyMessage(msg);
  await markProcessed(msg.id);
}
```

### Q271. How would you scale WebSocket connections (e.g. Socket.IO) across multiple Node.js instances behind a load balancer?
A single WebSocket connection is "sticky" to whichever server instance accepted it — but if User A (connected to instance 1) sends a message meant for User B (connected to instance 2), instance 1 has no direct way to reach User B's socket. The fix is a shared "adapter" (like the Socket.IO Redis adapter) that uses Redis Pub/Sub so ANY instance can broadcast an event, and it gets relayed to the correct instance holding that specific user's connection.

```js
const { createAdapter } = require('@socket.io/redis-adapter');
const io = new Server(httpServer);
io.adapter(createAdapter(pubClient, subClient)); // now broadcasts work across all instances
```

### Q272. What are "sticky sessions," and why are they needed for WebSocket load balancing but generally NOT needed for stateless REST APIs?
Sticky sessions ensure a load balancer routes ALL requests/connections from a given client to the SAME backend instance, based on something like a cookie or IP hash. WebSockets need this because the persistent connection itself lives on one specific instance's memory — a stateless REST API doesn't need it because ANY instance can independently handle ANY request without prior connection state.

```
# Nginx sticky session config (ip_hash)
upstream node_app {
  ip_hash;
  server 127.0.0.1:3000;
  server 127.0.0.1:3001;
}
```

### Q273. What is the difference between horizontal scaling a stateful WebSocket server vs a stateless HTTP API, in terms of architectural complexity?
Stateless HTTP APIs can scale by simply adding more identical instances behind a round-robin load balancer — no coordination needed between instances. Stateful WebSocket servers need EXTRA infrastructure (sticky sessions + a shared pub/sub layer like Redis, or a dedicated presence/routing service) to let instances communicate about which user is connected where, making horizontal scaling meaningfully more complex.

```js
// Stateless HTTP: any instance, any request — trivially scalable
// Stateful WebSocket: needs sticky routing AND cross-instance messaging (e.g. Redis adapter)
```

### Q274. How would you design a URL shortener service's core Node.js logic, focusing on ID generation at scale?
Two common approaches: 1) Base62-encode an auto-incrementing counter (from a centralized ID generator like a DB sequence, or a distributed ID scheme like Twitter's Snowflake) to produce a short, unique, non-colliding code. 2) Hash the original URL and take a short prefix, checking for collisions and appending a salt/retry if one occurs. Approach 1 avoids collisions entirely and is simpler to reason about at scale; approach 2 needs collision-handling logic.

```js
function toBase62(num) {
  const chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
  let result = '';
  while (num > 0) { result = chars[num % 62] + result; num = Math.floor(num / 62); }
  return result || '0';
}
// id = 125 -> short, unique code via toBase62(id)
```

### Q275. In a URL shortener, how would you handle the redirect endpoint to minimize database load at scale?
Cache the short-code → original-URL mapping aggressively in Redis (or an in-memory LRU cache with a short TTL) since redirects are read-heavy and rarely change once created — check the cache FIRST, only falling back to the database on a cache miss, and populate the cache on that miss for next time.

```js
app.get('/:code', async (req, res) => {
  let url = await redis.get(`url:${req.params.code}`);
  if (!url) {
    const doc = await UrlModel.findOne({ code: req.params.code });
    if (!doc) return res.status(404).send('Not found');
    url = doc.originalUrl;
    await redis.set(`url:${req.params.code}`, url, { EX: 86400 });
  }
  res.redirect(301, url);
});
```

### Q276. How would you design the Node.js backend architecture for a real-time chat application supporting millions of users?
Key pieces: WebSocket gateway servers (stateless-ish, but connection-sticky) handling client connections; a message broker (Kafka/Redis Pub/Sub) to fan out messages between gateway instances and to persistence workers; a database optimized for chat history (often a wide-column/NoSQL store partitioned by conversation ID); presence tracking (who's online) in a fast key-value store like Redis; and horizontal scaling of the gateway layer behind a load balancer with sticky routing.

```
Client <-WS-> [Gateway instances] <-PubSub-> [Message Broker] -> [Persistence Workers] -> [DB]
                                                             \-> [Presence Store (Redis)]
```

### Q277. What is fan-out-on-write vs fan-out-on-read, and how does this decision affect a Node.js-based notification/feed system's design?
Fan-out-on-write pushes a new post/event to ALL of a user's followers' feeds immediately when it's created (fast reads later, but expensive/slow writes for users with huge follower counts). Fan-out-on-read computes a user's feed on-demand when they request it, by pulling from everyone they follow (cheap writes, but potentially slow/expensive reads, especially for users following many people). Large systems often use a HYBRID: fan-out-on-write for most users, fan-out-on-read for "celebrity" accounts with huge follower counts.

```js
// Fan-out-on-write (simplified)
async function createPost(post) {
  const followers = await getFollowers(post.authorId);
  await Promise.all(followers.map(f => feedStore.push(f.id, post))); // push to every follower's feed now
}
```

### Q278. How would you design a distributed job scheduling system in Node.js that ensures a job runs exactly once even with multiple instances running?
Use a shared job queue (Redis-backed via BullMQ, or a dedicated scheduler service) where workers COMPETE to atomically claim a job (e.g. via `BRPOPLPUSH` or a similar atomic dequeue operation) — only ONE worker succeeds in claiming any given job, others move on. Combine this with a distributed lock or "leader election" for the SCHEDULING trigger itself (deciding WHEN to enqueue a recurring job), so multiple instances don't all enqueue duplicate job triggers.

```js
const { Queue, Worker } = require('bullmq');
const queue = new Queue('jobs', { connection: redisConnection });
new Worker('jobs', async (job) => { await processJob(job.data); }, { connection: redisConnection });
// BullMQ guarantees only one worker instance actually processes each specific job
```

### Q279. What is leader election, and how might you implement it among multiple Node.js instances (e.g. to decide which one runs a singleton scheduled task)?
Leader election is the process by which a group of distributed instances agree on ONE of them to act as the "leader" responsible for a particular task, with automatic failover if that leader goes down. A simple approach in Node uses Redis: each instance repeatedly tries to acquire a lock key with a short TTL (`SET leader:lock <id> NX EX 10`) — whoever holds it is the leader, and must keep renewing it; if it crashes and stops renewing, another instance's next attempt succeeds and takes over.

```js
async function tryBecomeLeader(instanceId) {
  const acquired = await redis.set('leader:lock', instanceId, { NX: true, EX: 10 });
  return !!acquired;
}
setInterval(async () => {
  if (await tryBecomeLeader(myId)) runScheduledSingletonTask();
}, 5000);
```

### Q280. How would you design rate-limited, resilient outbound webhook delivery from a Node.js service to third-party endpoints?
Queue webhook deliveries (rather than sending synchronously inline with the triggering event) so a slow/down third-party endpoint doesn't block your app. Use retry with exponential backoff for failed deliveries, a dead-letter queue for messages that exhaust all retries, per-destination rate limiting to avoid overwhelming any single receiver, and signature verification (HMAC) so receivers can trust the payload's authenticity.

```js
const crypto = require('crypto');
function signPayload(payload, secret) {
  return crypto.createHmac('sha256', secret).update(JSON.stringify(payload)).digest('hex');
}
// Send with header: X-Signature: signPayload(...)
// Receiver verifies the signature matches before trusting the webhook
```

### Q281. What is the difference between vertical partitioning and horizontal partitioning of a database, from a Node backend's perspective?
Horizontal partitioning (sharding) splits ROWS/documents of the SAME table/collection across multiple databases (e.g. users 1-1M on shard A, 1M-2M on shard B) — the schema is identical everywhere, just the data is split. Vertical partitioning splits COLUMNS/fields of a table into SEPARATE tables/services (e.g. user profile data in one service/DB, user billing data in another) — often aligning with microservice boundaries.

```js
// Horizontal: same schema, different data subsets per shard
// Vertical: different schemas/services entirely, split by domain (users vs billing)
```

### Q282. How would you implement a bulkhead pattern in a Node.js service to prevent one slow downstream dependency from exhausting all resources?
The bulkhead pattern isolates resources (like connection pools, thread/worker limits, or concurrency caps) PER downstream dependency, so a slow/failing dependency can only exhaust ITS OWN allotted resources, not starve calls to OTHER, healthy dependencies. In Node, this often means separate `http.Agent` connection pools or separate concurrency limiters (like `p-limit`) per external service.

```js
const pLimit = require('p-limit');
const paymentLimiter = pLimit(10); // max 10 concurrent calls to payment service
const emailLimiter = pLimit(20);   // separate pool for email service — one won't starve the other
```

### Q283. What is the difference between synchronous replication and asynchronous replication for a database used by a Node.js app, and what trade-off does each make?
Synchronous replication waits for the write to be confirmed on the replica(s) BEFORE acknowledging success to the application — guarantees no data loss on primary failure, but adds latency to every write (and can block writes entirely if a replica is slow/down). Asynchronous replication acknowledges the write as soon as the PRIMARY has it, replicating to secondaries in the background — faster writes, but a primary crash right after a write (before replication completes) can lose that data.

```js
// Trade-off is chosen at the database config level, not in app code —
// but app code should be aware: async replication means reads from replicas can be momentarily stale
```

### Q284. How would you design idempotent database migrations for a Node.js app deployed with zero downtime, where old and new code run simultaneously briefly?
Migrations must be BACKWARD-compatible during the rollout window: add new columns/fields as OPTIONAL (nullable) first, deploy new code that can handle BOTH old and new shapes, THEN later (in a separate migration) make fields required/remove old ones once ALL instances are on the new code — this "expand-contract" pattern avoids a moment where old code breaks against a new required schema, or new code breaks against old data.

```js
// Step 1: Add optional new field (old code ignores it, new code can use it)
// Step 2: Deploy new code that writes to BOTH old and new fields
// Step 3: Backfill existing data
// Step 4: Deploy code that only uses new field
// Step 5: Drop old field in a LATER migration, once fully rolled out
```

### Q285. What is the "expand-contract" (aka "parallel change") pattern for schema migrations, and why does it matter for Node.js apps with rolling deployments?
It's a multi-step migration strategy: EXPAND the schema (add new structure alongside the old, without removing anything), migrate/dual-write data, THEN CONTRACT (remove the old structure) only once ALL running instances are confirmed to be using the new structure. This matters for rolling deployments because for a period, BOTH old and new code versions run simultaneously against the SAME database — a migration that isn't backward-compatible would break whichever version is still running the old code.

```
Expand:   add new_field (nullable) alongside old_field
Migrate:  dual-write both fields; backfill new_field for existing rows
Contract: once ALL instances use new_field, drop old_field in a later deploy
```

### Q286. How would you design a system to detect and prevent duplicate/replayed requests in a Node.js API that's also protected against replay attacks?
Combine a short-lived, signed request timestamp/nonce with server-side tracking: reject requests with a timestamp too far in the past/future (clock skew tolerance), and store recently-seen NONCES (unique per-request random values) in a fast store (Redis, with a TTL matching the timestamp tolerance window) — reject any request reusing a nonce already seen within that window.

```js
app.use(async (req, res, next) => {
  const { nonce, timestamp } = req.headers;
  if (Math.abs(Date.now() - Number(timestamp)) > 5 * 60 * 1000) return res.status(400).send('Expired');
  const seen = await redis.set(`nonce:${nonce}`, '1', { NX: true, EX: 300 });
  if (!seen) return res.status(409).send('Replayed request');
  next();
});
```

### Q287. What is the difference between designing a Node.js API for strong consistency versus eventual consistency, and how does that choice ripple through your architecture?
Strong consistency means every read reflects the most recent write immediately — usually requires reading from a single source of truth (the primary DB), limiting how much you can cache or distribute reads, and can add latency/reduce availability during network partitions (per the CAP theorem). Eventual consistency allows reads to sometimes be slightly stale in exchange for much better scalability/availability (via caches, read replicas, async event propagation) — the right choice depends on whether the specific feature (e.g. a bank balance vs a "like count") can tolerate staleness.

```js
// Strong consistency: always read from primary
const balance = await primaryDb.query('SELECT balance FROM accounts WHERE id = $1', [id]);
// Eventual consistency: read from cache/replica, might be a few seconds stale
const likeCount = await redis.get(`post:${id}:likes`);
```

### Q288. What is the CAP theorem, and how does it inform real architectural decisions in a distributed Node.js system?
CAP theorem states that a distributed system can only fully guarantee TWO of three properties during a network partition: Consistency (every node sees the same data), Availability (every request gets a response), Partition tolerance (the system keeps working despite network failures between nodes). Since network partitions ARE going to happen in real distributed systems, partition tolerance isn't really optional — the real-world choice becomes CP (consistent but may reject requests during a partition) vs AP (available but may serve stale data during a partition), and this choice guides which databases/patterns you pick for which parts of your Node system.

```js
// CP choice: reject the write rather than risk inconsistency during a partition
// AP choice: accept the write locally, reconcile/merge later (eventual consistency)
```

### Q289. How would you implement a distributed rate limiter using Redis with a Lua script for full atomicity, and why is a Lua script needed here instead of separate commands?
Running separate Redis commands (`GET`, check, then `INCR`) from your Node app isn't atomic — between the `GET` and the `INCR`, ANOTHER concurrent request could slip in, causing a race condition where the limit is exceeded. A Lua script runs ENTIRELY atomically on the Redis server itself (Redis processes commands single-threaded, and a Lua script is treated as one indivisible operation), eliminating that race window completely.

```js
const script = `
  local current = redis.call('INCR', KEYS[1])
  if current == 1 then redis.call('EXPIRE', KEYS[1], ARGV[1]) end
  return current
`;
const count = await redis.eval(script, { keys: [`rate:${userId}`], arguments: ['60'] });
```

### Q290. What is the difference between designing your Node.js system for "high availability" versus "disaster recovery," and what does each practically require?
High availability (HA) is about minimizing DOWNTIME during normal operational failures (a single server crashing, a brief network blip) — achieved via redundancy (multiple instances, automatic failover) WITHIN a region/data center. Disaster recovery (DR) is about recovering from a CATASTROPHIC event (an entire region/data center going down) — requires cross-region replication/backups and a documented, tested recovery plan (RTO — recovery time objective, RPO — recovery point objective) to restore service elsewhere.

```
HA: multiple app instances + auto-restart + load balancer within one region
DR: cross-region database replication + regular backups + a tested failover runbook
```

### Q291. How would you design a Node.js background job system to handle "poison pill" messages (jobs that always fail and would otherwise retry forever)?
Track the retry count per job (stored alongside the job itself), and once it exceeds a maximum threshold, stop retrying and move the job to a "dead-letter queue" instead — a separate place for failed jobs to be inspected/alerted on/manually reprocessed, rather than endlessly consuming worker capacity retrying something that will never succeed.

```js
async function processJob(job) {
  try { await doWork(job); }
  catch (err) {
    job.attempts = (job.attempts || 0) + 1;
    if (job.attempts >= MAX_ATTEMPTS) await deadLetterQueue.add(job);
    else await mainQueue.add(job, { delay: backoffDelay(job.attempts) });
  }
}
```

### Q292. What is a "poison pill" specifically in the context of a Kafka/queue consumer written in Node.js, and how does it differ from a transient failure?
A poison pill is a specific MESSAGE whose content itself causes processing to fail EVERY time (e.g. malformed JSON, a value that always triggers a bug), as opposed to a transient failure (like a temporary network blip) that would likely succeed on a simple retry. Naively retrying a poison pill forever blocks the consumer from making progress on ALL subsequent messages in that partition/queue — the fix is detecting repeated failures on the SAME message and routing it to a dead-letter queue instead of retrying indefinitely.

```js
// Kafka consumer: skip and dead-letter a message after N consecutive failures,
// rather than letting it block the entire partition forever
```

### Q293. How would you design schema validation for incoming Kafka/queue messages in a Node.js consumer to prevent malformed data from crashing downstream processing?
Validate every incoming message against a strict schema (using a library like `zod`, `joi`, or JSON Schema) IMMEDIATELY upon receipt, BEFORE any business logic runs — messages failing validation get routed straight to a dead-letter queue/logged for investigation, rather than letting an unexpected shape propagate deep into your processing logic and cause a harder-to-diagnose crash.

```js
const { z } = require('zod');
const OrderSchema = z.object({ id: z.string(), amount: z.number().positive() });
async function handleMessage(raw) {
  const parsed = OrderSchema.safeParse(JSON.parse(raw));
  if (!parsed.success) return deadLetterQueue.add({ raw, error: parsed.error });
  await processOrder(parsed.data);
}
```

### Q294. What is the difference between "push" and "pull" based consumer models for message queues, and what does Node.js typically use for each major broker?
Push: the broker actively SENDS messages to the consumer as they arrive (lower latency, but the consumer has less control over pacing — risk of being overwhelmed). Pull: the consumer actively POLLS/requests messages when it's ready for more (better backpressure control, since the consumer decides its own pace). Kafka consumers in Node (`kafkajs`) are pull-based by design; RabbitMQ consumers can operate in either mode, though push-based consumption (`channel.consume()`) is common.

```js
// Kafka (pull-based): consumer explicitly fetches the next batch when ready
await consumer.run({ eachMessage: async ({ message }) => { await process(message); } });
```

### Q295. How would you architect a Node.js system to safely process financial transactions with strict ordering guarantees per account, while still processing DIFFERENT accounts in parallel?
Use PARTITIONED processing: route all messages/events for the SAME account to the SAME queue partition (e.g. by hashing the account ID to a Kafka partition), ensuring strict in-order processing WITHIN that partition (guaranteeing correct sequencing for that account's transactions), while DIFFERENT accounts' partitions are processed fully in parallel by different consumers — giving both correctness (per-account ordering) and throughput (cross-account parallelism).

```js
// Kafka producer: partition by account ID so all of one account's events land in the same partition (ordered)
await producer.send({
  topic: 'transactions',
  messages: [{ key: accountId, value: JSON.stringify(txn) }] // same key -> same partition
});
```

### Q296. What is the difference between optimizing a Node.js API for throughput versus optimizing for latency, and can you give an example where these goals conflict?
Throughput optimization focuses on maximizing the TOTAL number of requests processed per unit time (often via batching, larger buffers, more concurrency). Latency optimization focuses on minimizing the RESPONSE TIME of each INDIVIDUAL request. They can conflict: e.g. batching multiple database writes together improves throughput (fewer round trips overall) but INCREASES latency for the first request in a batch, which must wait for the batch to fill or timeout before it's actually processed.

```js
// Throughput-favoring: batch writes, higher overall efficiency, but each write waits a bit
function batchWrite(items) { /* accumulate for e.g. 50ms, then write all at once */ }
// Latency-favoring: write immediately, one round trip per item, less efficient overall
async function immediateWrite(item) { await db.insert(item); }
```

### Q297. How would you design load testing for a Node.js API to accurately simulate production traffic patterns, and what metrics matter most?
Use a load testing tool (k6, Artillery, autocannon) to simulate realistic concurrent user patterns (not just raw max throughput, but realistic think-time between requests, mixed endpoint usage matching real traffic proportions). Key metrics to track: requests per second at various load levels, p50/p95/p99 latency (not just averages, which hide tail latency problems), error rate under load, and the point where latency/errors start degrading sharply ("the knee" of the curve) — that's your realistic capacity ceiling.

```bash
# k6 example
k6 run --vus 100 --duration 30s script.js
# Watch p95/p99 latency and error rate, not just average response time
```

### Q298. What is "tail latency," and why is it more important than average latency for a Node.js API serving real users at scale?
Tail latency refers to the SLOWEST percentage of requests (p95, p99, p99.9) — even if the AVERAGE latency looks great, a meaningful fraction of real users could be experiencing painfully slow responses, especially in systems where a single user action triggers MULTIPLE backend calls (since the odds of hitting at least ONE slow call compound with more calls per user action). Optimizing only for the average can mask a genuinely bad experience for a significant chunk of users.

```js
// A single page load hitting 10 backend services, each with p99 = 500ms,
// has a much higher chance that AT LEAST ONE of those 10 calls is slow for that user
```

### Q299. How would you design a chaos engineering test for a Node.js microservices system, and what failure modes would you specifically want to simulate?
Chaos engineering deliberately injects controlled failures into a system to verify it degrades gracefully rather than catastrophically. For a Node microservices system, you'd want to simulate: a downstream service becoming slow (not just fully down — partial degradation is often worse and harder to handle), a downstream service returning errors intermittently, network partitions between services, a database failover event, and resource exhaustion (CPU/memory pressure) — then verify circuit breakers, timeouts, retries, and fallbacks actually behave as designed under each scenario.

```js
// Example: inject artificial latency into a dependency call to test your timeout/circuit-breaker logic
if (process.env.CHAOS_MODE) await new Promise(r => setTimeout(r, 5000));
const result = await callDownstreamService();
```

### Q300. If you were asked to design a production-grade, scalable Node.js backend from scratch in a system-design interview, what are the key architectural decisions you'd walk through, in order?
1) **API layer**: REST/GraphQL/gRPC choice, versioning strategy. 2) **Statelessness**: keep app servers stateless, push session/state to Redis. 3) **Database**: choice of SQL/NoSQL based on data shape, indexing strategy, read replicas/sharding plan. 4) **Caching**: what to cache, invalidation strategy, thundering-herd protection. 5) **Async processing**: what should be a background job/queue vs synchronous. 6) **Scaling**: horizontal scaling behind a load balancer, autoscaling triggers. 7) **Resilience**: timeouts, retries with backoff, circuit breakers, graceful degradation. 8) **Security**: auth strategy, input validation, rate limiting, secrets management. 9) **Observability**: logging, metrics, tracing, alerting. 10) **Deployment**: CI/CD, zero-downtime rollout strategy, rollback plan. Walking through these in order shows the interviewer you think about a system holistically, not just "write some Express routes."

```
Client -> Load Balancer -> [Stateless Node instances] -> Cache (Redis) -> DB (+ replicas/shards)
                                    |-> Queue (async jobs) -> Workers
                                    |-> Observability (logs/metrics/traces)
```

---

## How to use this pack
- **Basic (1–75):** core syntax, modules, npm, fs/http basics — make sure these are instant recall.
- **Mid (76–150):** Express patterns, streams, auth, testing, DB basics — this is where most SDE-1/SDE-2 rounds live.
- **Advanced (151–225):** V8/GC internals, security, microservices, design patterns — SDE-2/SDE-3 and system-design-adjacent rounds.
- **Super Advanced (226–300):** internals, distributed systems, large-scale architecture — senior/staff-level and system design interviews.

Tip: don't just memorize the answers — pick 10–15 questions per section and actually type out the code snippets yourself. That's what makes it stick under interview pressure.

*Total: 301 questions (Basic → Super Advanced), each with an explanation and a code snippet.*
