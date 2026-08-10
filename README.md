# 300 Node.js Interview Questions — With Answers & Code

**Tiers:** 🟢 Basic · 🔵 Mid · 🟠 Advanced · 🔴 Super-Advanced

**Contents**

| Section | Questions | Tier focus |
|---|---|---|
| 1. Core Fundamentals & Runtime | 1–35 | 🟢 |
| 2. Modules, CommonJS & ESM | 36–65 | 🟢🔵 |
| 3. Event Loop, Timers & Async Internals | 66–110 | 🔵🟠🔴 |
| 4. Promises, async/await & Concurrency Patterns | 111–140 | 🔵🟠 |
| 5. Events, EventEmitter & Observability | 141–160 | 🔵 |
| 6. Buffers, Streams & Backpressure | 161–195 | 🟠🔴 |
| 7. File System, Path & OS | 196–212 | 🟢🔵 |
| 8. HTTP, Networking & Express | 213–245 | 🔵🟠 |
| 9. Error Handling, Debugging & Testing | 246–262 | 🔵🟠 |
| 10. Performance, Cluster, Worker Threads & Memory | 263–283 | 🟠🔴 |
| 11. Security | 284–294 | 🟠 |
| 12. Architecture, Deployment & System Design | 295–300 | 🔴 |

---

## 1. Core Fundamentals & Runtime (1–35)

### 1. What is Node.js? 🟢
A runtime that executes JavaScript outside the browser, built on V8 and libuv. It gives JS access to the file system, network sockets, processes and OS primitives, using a single-threaded event loop with non-blocking I/O.

```js
// hello.js
const http = require('node:http');
http.createServer((req, res) => res.end('hi')).listen(3000);
```

### 2. Is Node.js single-threaded? 🟢
Your JavaScript runs on one thread. Node itself is multi-threaded: libuv maintains a thread pool (default 4) for file I/O, DNS, crypto and zlib, plus V8 has its own helper threads for GC and JIT.

```js
process.env.UV_THREADPOOL_SIZE = 8; // must be set before first async fs/crypto call
```

### 3. What is V8? 🟢
Google's JavaScript engine. It parses JS, produces bytecode via Ignition, optimizes hot functions with TurboFan, and manages the heap and garbage collection.

### 4. What is libuv? 🟢
The C library that provides Node's event loop, thread pool, and cross-platform async I/O (epoll on Linux, kqueue on macOS, IOCP on Windows).

### 5. Node.js vs the browser JavaScript environment? 🟢
No DOM/`window`; instead `global`, `process`, `Buffer`, `require`. Node has file/network access, browsers have sandboxing and CORS. Module systems historically differ (CJS vs `<script>`/ESM).

### 6. What is the `global` object? 🟢
Node's top-level object, analogous to `window`. `globalThis` is the portable name.

```js
globalThis.appStartedAt = Date.now();
console.log(global === globalThis); // true
```

### 7. What is `process`? 🟢
A global EventEmitter describing the current Node process: arguments, env vars, stdio, exit control, and lifecycle events.

```js
console.log(process.pid, process.version, process.platform);
process.on('exit', code => console.log('exiting with', code));
```

### 8. How do you read command-line arguments? 🟢
`process.argv` is an array: `[execPath, scriptPath, ...userArgs]`.

```js
// node app.js --port 4000
const args = process.argv.slice(2);
console.log(args); // ['--port', '4000']

// Node 18.3+ built-in parser
const { values } = require('node:util').parseArgs({
  options: { port: { type: 'string', default: '3000' } }
});
console.log(values.port);
```

### 9. How do you read environment variables? 🟢
`process.env`. Values are always strings (or `undefined`).

```js
const PORT = Number(process.env.PORT ?? 3000);
if (!process.env.DB_URL) throw new Error('DB_URL is required');
```

### 10. What is `process.env.NODE_ENV` used for? 🟢
A conventional flag (`development`/`production`/`test`) that libraries like Express use to toggle caching, verbose errors and logging.

### 11. Difference between `process.exit()` and letting the process end naturally? 🔵
`process.exit()` terminates immediately, potentially truncating pending async writes to stdout/files. Prefer setting `process.exitCode` and letting the loop drain.

```js
process.exitCode = 1; // graceful
// process.exit(1);   // abrupt
```

### 12. What keeps a Node process alive? 🔵
Active handles (servers, sockets, timers) and active requests. When the reference count drops to zero, the loop exits.

```js
const t = setInterval(() => {}, 1000);
t.unref(); // no longer keeps the process alive
```

### 13. What is `require` vs `import`? 🟢
`require` is CommonJS: synchronous, runtime-resolved, returns `module.exports`. `import` is ESM: statically analyzed, hoisted, asynchronous, supports tree-shaking and top-level `await`.

### 14. What is the `node:` prefix? 🔵
An explicit protocol for built-ins that prevents shadowing by an npm package of the same name and works identically in CJS and ESM.

```js
const fs = require('node:fs/promises');
```

### 15. Is Node.js good for CPU-bound work? 🔵
Not by default — a long synchronous computation blocks the loop and starves all requests. Offload to `worker_threads`, a child process, or a native addon.

```js
// blocks everything for ~seconds
function fib(n) { return n < 2 ? n : fib(n - 1) + fib(n - 2); }
```

### 16. What is non-blocking I/O? 🟢
The OS is asked to perform I/O and notify Node on completion; the thread continues executing other JS in the meantime instead of waiting.

```js
const fs = require('node:fs');
fs.readFile('a.txt', () => console.log('2nd'));
console.log('1st');
```

### 17. `readFile` vs `readFileSync`? 🟢
The sync version blocks the event loop until the OS returns — acceptable at startup, harmful inside a request handler.

### 18. What is the REPL? 🟢
Read-Eval-Print Loop: type `node` with no args. `_` holds the last result; `.editor`, `.load`, `.save`, `.exit` are dot commands.

### 19. How do you check the Node version programmatically? 🟢
```js
console.log(process.version);          // 'v20.11.0'
console.log(process.versions.v8, process.versions.uv);
```

### 20. What is `nvm`? 🟢
Node Version Manager — installs and switches between Node versions per shell or per project (`.nvmrc`).

### 21. What are LTS and Current releases? 🔵
Even-numbered majors become LTS (30 months of support) and are the safe choice for production. Odd-numbered lines are short-lived and for experimentation.

### 22. What is `__dirname` and `__filename`? 🟢
CJS-only variables for the directory and full path of the current module. In ESM use `import.meta`.

```js
// ESM equivalent
import { fileURLToPath } from 'node:url';
import path from 'node:path';
const __filename = fileURLToPath(import.meta.url);
const __dirname  = path.dirname(__filename);
// Node 20.11+: import.meta.dirname / import.meta.filename
```

### 23. What is `module.exports` vs `exports`? 🟢
`exports` is just a reference to `module.exports`. Reassigning `exports` breaks the link; reassigning `module.exports` is what actually gets returned.

```js
exports.a = 1;              // works
exports = { b: 2 };         // ❌ lost
module.exports = { b: 2 };  // ✅
```

### 24. What is the module wrapper function? 🔵
Node wraps every CJS file before execution, which is why those five identifiers exist and why top-level `var` is module-scoped.

```js
(function (exports, require, module, __filename, __dirname) {
  // your code
});
```

### 25. What is `require.main === module` for? 🔵
Detects whether the file was run directly vs imported — the Node equivalent of Python's `__main__`.

```js
function main() { /* ... */ }
if (require.main === module) main();
module.exports = main;
```

### 26. What is `Buffer`? 🟢
A fixed-length view of raw binary memory outside the V8 heap, used for file and socket data.

```js
const b = Buffer.from('héllo', 'utf8');
console.log(b.length, b.toString('base64'));
```

### 27. What is `process.nextTick()`? 🔵
Schedules a callback to run after the current operation completes, before the event loop continues — ahead of promises and all timers.

```js
console.log('A');
process.nextTick(() => console.log('B'));
Promise.resolve().then(() => console.log('C'));
setTimeout(() => console.log('D'));
console.log('E');
// A E B C D
```

### 28. What is `setImmediate()`? 🔵
Runs the callback in the *check* phase of the next loop iteration — after I/O callbacks of the current iteration.

### 29. `setImmediate` vs `setTimeout(fn, 0)` at top level? 🟠
Non-deterministic — it depends on how long process startup took. Inside an I/O callback, `setImmediate` always fires first.

```js
require('node:fs').readFile(__filename, () => {
  setTimeout(() => console.log('timeout'), 0);
  setImmediate(() => console.log('immediate')); // always first here
});
```

### 30. What is a callback? 🟢
A function passed as an argument, invoked when work completes. Node's convention is error-first: `(err, result)`.

```js
fs.readFile('f.txt', (err, data) => {
  if (err) return console.error(err);
  console.log(data.toString());
});
```

### 31. What is callback hell and how do you fix it? 🟢
Deep nesting of callbacks. Fix with named functions, promises, or `async/await`.

```js
const data = await fs.promises.readFile('f.txt', 'utf8');
```

### 32. What is `util.promisify`? 🔵
Wraps an error-first callback function into one returning a promise.

```js
const { promisify } = require('node:util');
const sleep = promisify(setTimeout);
await sleep(100);
```

### 33. What is `util.callbackify`? 🔵
The inverse — exposes an async function to callback-style consumers.

```js
const cbFn = require('node:util').callbackify(async () => 'ok');
cbFn((err, v) => console.log(v));
```

### 34. What are the standard streams in Node? 🟢
`process.stdin` (Readable), `process.stdout` and `process.stderr` (Writable). stdout is synchronous to files and TTYs on POSIX, asynchronous to pipes.

```js
process.stdin.pipe(process.stdout); // echo
```

### 35. What is `console.log` really doing? 🔵
Formatting with `util.format` and writing to `process.stdout`. It is not free — in hot loops it can dominate CPU and blocks on TTY writes.

---
## 2. Modules, CommonJS & ESM (36–65)

### 36. How does `require()` resolve a module? 🔵
Core module → relative/absolute path (with `.js`, `.json`, `.node` extension probing, then `/index.*`) → walk up `node_modules` directories from the caller to the filesystem root.

```js
console.log(require.resolve('lodash'));
console.log(module.paths); // the node_modules search chain
```

### 37. Is `require()` cached? 🔵
Yes — keyed by resolved absolute filename. A module's body runs exactly once per path per process.

```js
delete require.cache[require.resolve('./config')]; // force reload (test-only)
```

### 38. What happens with circular `require`? 🟠
Node returns the **partially filled** `module.exports` of the in-progress module rather than looping forever. Exports read at load time may be `undefined`.

```js
// a.js
exports.done = false;
const b = require('./b');
exports.done = true;

// b.js
const a = require('./a');
console.log(a.done); // false — a isn't finished yet
```
Fix: require lazily inside the function, or restructure to remove the cycle.

### 39. How do circular imports behave in ESM? 🟠
ESM uses live bindings and hoisted declarations, so function declarations are usable, but accessing a `const` in the temporal dead zone throws a `ReferenceError`.

### 40. How do you enable ESM in Node? 🔵
`"type": "module"` in package.json, or the `.mjs` extension. Use `.cjs` to force CommonJS inside an ESM package.

### 41. Can you `require()` an ESM module? 🟠
Historically no — you had to use dynamic `import()`. Node 22+ can `require()` a synchronous ES module (no top-level await) behind `require(esm)` support.

```js
const mod = await import('./esm-only.mjs'); // universally safe
```

### 42. Can ESM import CommonJS? 🔵
Yes. The CJS `module.exports` becomes the default export; named exports are statically detected best-effort.

```js
import pkg from './legacy.cjs';
const { helper } = pkg;
```

### 43. What is the `exports` field in package.json? 🟠
Defines the public entry points and **blocks deep imports** into internals. Supports conditional resolution.

```json
{
  "name": "mylib",
  "exports": {
    ".": { "import": "./dist/index.mjs", "require": "./dist/index.cjs", "types": "./dist/index.d.ts" },
    "./utils": "./dist/utils.mjs"
  }
}
```

### 44. What is the `imports` field / subpath imports? 🟠
Private aliases beginning with `#`, resolved without a bundler.

```json
{ "imports": { "#db": "./src/db/index.js" } }
```

### 45. What is dual-package hazard? 🔴
A library shipped as both CJS and ESM can be loaded twice, producing two copies of state (two class identities, two singletons). Mitigate by putting state in one CJS core that the ESM wrapper re-exports.

### 46. What is top-level await? 🔵
Allowed only in ESM. It delays the module's evaluation until the promise settles.

```js
// config.mjs
export const config = await fetch(url).then(r => r.json());
```

### 47. What is `import.meta`? 🔵
Per-module metadata. `import.meta.url` is the module's URL; Node 20.11+ adds `dirname`, `filename`, and `resolve()`.

### 48. What is dynamic `import()`? 🔵
A function-like expression returning a promise — enables lazy loading and conditional loading in both CJS and ESM.

```js
if (needHeavy) { const { heavy } = await import('./heavy.js'); heavy(); }
```

### 49. What is npm vs npx? 🟢
`npm` installs/manages packages; `npx` executes a package binary, downloading it temporarily if not installed.

### 50. `dependencies` vs `devDependencies` vs `peerDependencies` vs `optionalDependencies`? 🔵
Runtime needs / build-and-test-only / "host must provide this version" (plugins) / install failure is tolerated.

### 51. What does `package-lock.json` do? 🔵
Pins the exact resolved version + integrity hash of every node in the tree so installs are reproducible. Always commit it for applications.

### 52. `npm install` vs `npm ci`? 🔵
`install` may mutate the lockfile and reuses `node_modules`. `ci` deletes `node_modules`, installs strictly from the lockfile, and fails if the lockfile is out of sync — correct for CI/CD.

### 53. Explain semver ranges. 🔵
`1.2.3` exact · `^1.2.3` → `<2.0.0` · `~1.2.3` → `<1.3.0` · `>=`, `||`, `*`. For `0.x`, caret only allows patch bumps.

### 54. What are npm scripts and lifecycle hooks? 🔵
```json
{ "scripts": {
  "prebuild": "rimraf dist",
  "build": "tsc -p .",
  "start": "node dist/index.js",
  "test": "node --test"
}}
```
`pre`/`post` prefixes run automatically around a script.

### 55. What are npm workspaces? 🟠
Monorepo support: sibling packages symlinked into the root `node_modules` and installed together.

```json
{ "workspaces": ["packages/*"] }
```

### 56. How do you publish a package? 🔵
`npm login` → set `name`, `version`, `main`/`exports`, `files` → `npm publish` (add `--access public` for a scoped package).

### 57. What is `npm audit` and `npm outdated`? 🔵
Report known CVEs in the dependency tree, and list packages behind their latest version. `npm audit fix --force` can introduce breaking changes — review it.

### 58. What is `.npmrc`? 🔵
Config file for registry URLs, auth tokens, `save-exact`, scoped registries. Never commit tokens — use `${NPM_TOKEN}` env substitution.

### 59. What are `bin` and shebangs for CLI tools? 🔵
```json
{ "bin": { "mytool": "./cli.js" } }
```
```js
#!/usr/bin/env node
console.log('hello from CLI');
```

### 60. How do you create a native/C++ addon? 🔴
Use Node-API (N-API) via `node-addon-api` — ABI-stable across Node versions, unlike raw V8 APIs — and build with `node-gyp`.

### 61. What is `module.createRequire`? 🟠
Builds a `require` inside an ESM file, e.g. to load JSON or a CJS-only dependency.

```js
import { createRequire } from 'node:module';
const require = createRequire(import.meta.url);
const pkg = require('./package.json');
```

### 62. How do you import JSON in ESM? 🔵
```js
import pkg from './package.json' with { type: 'json' }; // import attributes
```

### 63. What are loaders / `module.register()`? 🔴
Hooks that intercept module resolution and loading — used by TS runners and mocking libraries.

```js
// node --import ./register.mjs app.js
import { register } from 'node:module';
register('./my-loader.mjs', import.meta.url);
```

### 64. How is TypeScript run in Node? 🔵
Compile with `tsc`, or run directly with `tsx`/`ts-node`, or use Node 22.6+ `--experimental-strip-types` (type stripping only, no transpilation of enums/decorators).

### 65. How do you avoid `../../../` import paths? 🔵
Subpath imports (`#`), workspaces, or `tsconfig` `paths` + a runtime resolver. Prefer the standard `imports` field for pure Node.

---
## 3. Event Loop, Timers & Async Internals (66–110)

### 66. Describe the event loop phases in order. 🟠
1. **timers** — expired `setTimeout`/`setInterval`
2. **pending callbacks** — deferred system errors (e.g. TCP `ECONNREFUSED`)
3. **idle/prepare** — internal
4. **poll** — retrieve new I/O events, execute I/O callbacks; may block here
5. **check** — `setImmediate`
6. **close callbacks** — `'close'` events (e.g. `socket.on('close')`)

Between every callback (and between phases), the microtask queues drain: `process.nextTick` queue first, then the promise microtask queue.

### 67. What is the microtask queue vs macrotask queue? 🔵
Microtasks (promises, `queueMicrotask`, `nextTick`) run to exhaustion after the current callback. Macrotasks (timers, I/O, immediates) are one-per-phase-turn.

```js
setTimeout(() => console.log('macro'));
queueMicrotask(() => console.log('micro'));
// micro, then macro
```

### 68. Why is `process.nextTick` dangerous? 🟠
Its queue is drained completely before promises and before the loop advances — a recursive `nextTick` starves I/O forever.

```js
function loop() { process.nextTick(loop); } // event loop never progresses
```

### 69. Predict the output. 🟠
```js
console.log('1');
setTimeout(() => console.log('2'), 0);
setImmediate(() => console.log('3'));
process.nextTick(() => console.log('4'));
Promise.resolve().then(() => console.log('5'));
(async () => { console.log('6'); await null; console.log('7'); })();
console.log('8');
```
**Output:** `1, 6, 8, 4, 5, 7, 2, 3` — sync first; nextTick queue; then microtasks in FIFO (`5` was queued before `7`); then timers; then check.

### 70. Why is `setTimeout(fn, 0)` not really 0ms? 🔵
Node clamps to 1ms and the callback only runs when the loop reaches the timers phase, so the real delay is "≥1ms, whenever we get there".

### 71. How are timers stored internally? 🔴
libuv keeps a min-heap keyed by expiry; Node groups timers with identical durations into linked lists so inserting is O(1) for the common case.

### 72. What is timer drift with `setInterval`? 🟠
The interval measures from callback *start*, so slow callbacks accumulate lag and can queue up. For accuracy, self-schedule with a corrected delay.

```js
function tick(period, fn, next = Date.now() + period) {
  setTimeout(() => { fn(); tick(period, fn, next + period); }, next - Date.now());
}
```

### 73. What do `ref()`/`unref()` do on a timer? 🟠
`unref()` removes the handle from the loop's "keep alive" count — the timer still fires if the process is alive for other reasons.

### 74. What is the poll phase's blocking behaviour? 🔴
If there are no pending timers and no `setImmediate`s scheduled, the poll phase blocks in `epoll_wait` until an I/O event arrives — that's why an idle server uses ~0% CPU.

### 75. What is a "tick" of the event loop? 🔵
One full pass through all phases.

### 76. How do you measure event loop lag? 🟠
```js
const { monitorEventLoopDelay } = require('node:perf_hooks');
const h = monitorEventLoopDelay({ resolution: 10 });
h.enable();
setInterval(() => console.log('p99 lag ms:', h.percentile(99) / 1e6), 5000).unref();
```

### 77. What is `setTimeout` returning in Node vs the browser? 🔵
Node returns a `Timeout` object (with `ref`/`unref`/`refresh`), not a number. Use `timersPromises` for a promise API.

```js
const { setTimeout: sleep } = require('node:timers/promises');
await sleep(500);
```

### 78. How do you make a cancellable sleep? 🟠
```js
const { setTimeout: sleep } = require('node:timers/promises');
const ac = new AbortController();
sleep(5000, null, { signal: ac.signal }).catch(e => console.log(e.name)); // AbortError
ac.abort();
```

### 79. What is `AbortController` used for in Node? 🟠
Standard cancellation for `fetch`, `fs` operations, streams, timers, and your own async functions.

```js
const ac = new AbortController();
setTimeout(() => ac.abort(), 1000);
await fetch(url, { signal: ac.signal });
```

### 80. What is `AbortSignal.timeout()`? 🔵
```js
await fetch(url, { signal: AbortSignal.timeout(2000) });
```

### 81. What runs on the libuv thread pool? 🟠
`fs` (most operations), `dns.lookup`, `crypto.pbkdf2`/`scrypt`/`randomBytes`, `zlib`. **Not** network sockets — those use the OS event notification mechanism directly.

### 82. Demonstrate thread pool saturation. 🔴
```js
process.env.UV_THREADPOOL_SIZE = 4;
const crypto = require('node:crypto');
const start = Date.now();
for (let i = 0; i < 8; i++)
  crypto.pbkdf2('pw', 'salt', 200000, 64, 'sha512', () => console.log(i, Date.now() - start));
// first 4 finish together, next 4 finish at ~2x — the pool is the bottleneck
```

### 83. `dns.lookup` vs `dns.resolve`? 🟠
`lookup` calls `getaddrinfo` (blocking → thread pool, honours `/etc/hosts`). `resolve` speaks DNS over the network asynchronously without occupying a pool thread. Under heavy load, `resolve` avoids starving `fs`.

### 84. How do you detect that you're blocking the event loop? 🟠
Event loop delay histogram, `--prof`/`--cpu-prof`, `blocked-at` in dev, or a watchdog timer whose actual firing time you compare against expected.

### 85. What is `queueMicrotask` vs `Promise.resolve().then()`? 🔵
Same queue; `queueMicrotask` avoids allocating a promise and rethrows errors as uncaught exceptions rather than silently rejecting.

### 86. Explain starvation with a long microtask chain. 🔴
Because microtasks drain fully, an unbounded `then` chain blocks all I/O. Yield to the macrotask queue periodically.

```js
async function processAll(items) {
  for (let i = 0; i < items.length; i++) {
    work(items[i]);
    if (i % 1000 === 0) await new Promise(r => setImmediate(r)); // yield
  }
}
```

### 87. What is `--max-old-space-size`? 🟠
Caps the V8 old-generation heap (MB). Raise it for memory-heavy jobs; it doesn't affect Buffers, which live outside the heap.

### 88. How do you break a CPU-heavy task into chunks? 🟠
Slice the work and yield with `setImmediate` so I/O callbacks interleave — or move it to a worker thread if throughput matters.

### 89. What are `perf_hooks` used for? 🟠
```js
const { performance, PerformanceObserver } = require('node:perf_hooks');
new PerformanceObserver(list =>
  list.getEntries().forEach(e => console.log(e.name, e.duration))
).observe({ entryTypes: ['measure'] });
performance.mark('a'); doWork(); performance.mark('b');
performance.measure('work', 'a', 'b');
```

### 90. What is `AsyncLocalStorage`? 🔴
Continuation-local storage: a context object that automatically propagates across async boundaries — ideal for request IDs and tracing without threading a parameter through every function.

```js
const { AsyncLocalStorage } = require('node:async_hooks');
const als = new AsyncLocalStorage();

app.use((req, res, next) => als.run({ reqId: crypto.randomUUID() }, next));
function log(msg) { console.log(als.getStore()?.reqId, msg); }
```

### 91. What is `async_hooks`? 🔴
Low-level lifecycle hooks (`init`/`before`/`after`/`destroy`) for every async resource. Powerful for APM tools but costly — it disables some V8 optimizations.

### 92. What is the async resource / `AsyncResource` class? 🔴
Wraps a callback so it keeps the correct async context (used when you pool or cache callbacks, e.g. in a custom thread pool).

```js
const { AsyncResource } = require('node:async_hooks');
const bound = AsyncResource.bind(() => log('still has context'));
```

### 93. How does `await` compile? 🟠
Roughly to `promise.then(resume)` — the function suspends and the remainder becomes a microtask. `await` of a non-promise still costs one microtask tick.

### 94. What is an unhandled rejection and what does Node do? 🔵
Since Node 15 it **crashes the process** by default (`--unhandled-rejections=throw`).

```js
process.on('unhandledRejection', (reason, p) => { logger.fatal(reason); process.exit(1); });
```

### 95. What is `uncaughtException` and should you use it? 🟠
A last-resort hook. The process is in an undefined state — log, flush, and exit. Never resume serving traffic.

```js
process.on('uncaughtException', err => { logger.fatal(err); process.exit(1); });
```

### 96. What is `uncaughtExceptionMonitor`? 🟠
Observe the exception for logging without altering the default crash behaviour.

### 97. What are `beforeExit` vs `exit` events? 🟠
`beforeExit` fires when the loop empties and *can* schedule more async work (not fired on explicit `process.exit()`). `exit` is synchronous-only.

### 98. How do you implement graceful shutdown? 🟠
```js
const server = app.listen(3000);
let shuttingDown = false;
for (const sig of ['SIGTERM', 'SIGINT']) process.on(sig, async () => {
  if (shuttingDown) return; shuttingDown = true;
  server.close(() => console.log('http closed'));       // stop accepting
  setTimeout(() => process.exit(1), 10_000).unref();     // hard deadline
  await db.close();
  process.exitCode = 0;
});
```

### 99. What are POSIX signals in Node? 🔵
`SIGTERM` (polite stop, sent by Docker/K8s), `SIGINT` (Ctrl-C), `SIGHUP`, `SIGKILL`/`SIGSTOP` (cannot be trapped).

### 100. What is `process.on('warning')`? 🔵
Catches deprecation and max-listener warnings so you can route them to your logger.

### 101. What is the `--trace-warnings` flag? 🔵
Prints a full stack trace for warnings such as `MaxListenersExceededWarning`.

### 102. How do you profile CPU in production? 🟠
`node --cpu-prof app.js` writes a `.cpuprofile` loadable in Chrome DevTools; or attach with `--inspect` and take a profile; or use `0x`/`clinic flame`.

### 103. What is the inspector protocol? 🟠
```bash
node --inspect=0.0.0.0:9229 app.js   # attach DevTools / VS Code
node --inspect-brk app.js            # pause on first line
```
Never expose 9229 publicly — it grants arbitrary code execution.

### 104. What is `node:diagnostics_channel`? 🔴
A zero-cost-when-unsubscribed pub/sub bus for instrumenting libraries.

```js
const dc = require('node:diagnostics_channel');
const ch = dc.channel('app:query');
ch.subscribe(({ sql, ms }) => metrics.observe(sql, ms));
if (ch.hasSubscribers) ch.publish({ sql, ms });
```

### 105. What is a diagnostic report? 🟠
`node --report-on-fatalerror --report-uncaught-exception app.js` writes a JSON dump of the stack, heap, handles and environment at failure — invaluable post-mortem.

### 106. How would you detect an event loop stall in a health check? 🟠
Expose the p99 loop delay and fail readiness above a threshold, so the orchestrator stops routing traffic to a wedged pod.

### 107. Why can a `Promise` chain silently swallow errors? 🔵
A missing `.catch()` on a detached chain. Always terminate chains or use `await` inside a `try/catch`.

### 108. What is `Promise.prototype.finally` semantics? 🔵
Runs on both paths and **passes the original settlement through** unless it throws or returns a rejected promise.

### 109. What is `--trace-sync-io`? 🟠
Warns when synchronous I/O happens after the first event loop turn — catches accidental `readFileSync` in request paths.

### 110. What is `--frozen-intrinsics` / `--experimental-permission`? 🔴
Hardening flags: freeze built-in prototypes against prototype pollution, and restrict a process's filesystem/child-process/worker access (`--allow-fs-read=/app`).

---
## 4. Promises, async/await & Concurrency Patterns (111–140)

### 111. What are the promise states? 🟢
`pending` → `fulfilled` or `rejected`. Settlement is one-way and permanent.

### 112. `Promise.all` vs `allSettled` vs `race` vs `any`? 🔵
| Combinator | Resolves when | Rejects when |
|---|---|---|
| `all` | all fulfil | first rejection (fail-fast) |
| `allSettled` | all settle | never |
| `race` | first settles (either way) | first settles as rejection |
| `any` | first fulfilment | all reject → `AggregateError` |

```js
const [a, b] = await Promise.all([getUser(), getOrders()]); // parallel
```

### 113. Sequential vs parallel awaits — what's the bug? 🔵
```js
// ❌ 2 round trips serialized
const u = await getUser(); const o = await getOrders();
// ✅ overlapped
const [u2, o2] = await Promise.all([getUser(), getOrders()]);
```

### 114. Why can `Promise.all` leak unhandled rejections? 🟠
It rejects on the first failure but the other promises keep running; if one later rejects it's already "handled" by `all`, but a promise created *outside* the array and never awaited will crash the process. Prefer `allSettled` when you must observe every outcome.

### 115. Implement a promise concurrency limiter. 🟠
```js
async function mapLimit(items, limit, fn) {
  const results = new Array(items.length);
  let i = 0;
  const workers = Array.from({ length: Math.min(limit, items.length) }, async () => {
    while (i < items.length) {
      const idx = i++;
      results[idx] = await fn(items[idx], idx);
    }
  });
  await Promise.all(workers);
  return results;
}
// await mapLimit(urls, 5, u => fetch(u).then(r => r.json()));
```

### 116. Implement retry with exponential backoff and jitter. 🟠
```js
async function retry(fn, { attempts = 5, base = 100, cap = 5000 } = {}) {
  let lastErr;
  for (let n = 0; n < attempts; n++) {
    try { return await fn(); }
    catch (err) {
      lastErr = err;
      if (err.status && err.status < 500) throw err;      // don't retry 4xx
      const delay = Math.min(cap, base * 2 ** n) * (0.5 + Math.random() / 2);
      await new Promise(r => setTimeout(r, delay));
    }
  }
  throw lastErr;
}
```

### 117. Implement a timeout wrapper. 🔵
```js
function withTimeout(promise, ms) {
  let t;
  const timer = new Promise((_, rej) =>
    t = setTimeout(() => rej(new Error('ETIMEDOUT')), ms));
  return Promise.race([promise, timer]).finally(() => clearTimeout(t));
}
```

### 118. Implement a circuit breaker. 🔴
```js
class CircuitBreaker {
  constructor(fn, { threshold = 5, cooldown = 10_000 } = {}) {
    Object.assign(this, { fn, threshold, cooldown, fails: 0, state: 'CLOSED', openedAt: 0 });
  }
  async exec(...args) {
    if (this.state === 'OPEN') {
      if (Date.now() - this.openedAt < this.cooldown) throw new Error('CIRCUIT_OPEN');
      this.state = 'HALF_OPEN';
    }
    try {
      const out = await this.fn(...args);
      this.fails = 0; this.state = 'CLOSED';
      return out;
    } catch (e) {
      if (++this.fails >= this.threshold) { this.state = 'OPEN'; this.openedAt = Date.now(); }
      throw e;
    }
  }
}
```

### 119. Implement request deduplication (single-flight). 🟠
```js
const inflight = new Map();
function once(key, fn) {
  if (inflight.has(key)) return inflight.get(key);
  const p = fn().finally(() => inflight.delete(key));
  inflight.set(key, p);
  return p;
}
```

### 120. Implement a simple promise-based mutex. 🟠
```js
class Mutex {
  #tail = Promise.resolve();
  lock() {
    let release;
    const next = new Promise(r => release = r);
    const wait = this.#tail.then(() => release);
    this.#tail = this.#tail.then(() => next);
    return wait;
  }
}
// const unlock = await mutex.lock(); try { ... } finally { unlock(); }
```

### 121. Write a minimal Promise implementation. 🔴
```js
class MyPromise {
  constructor(executor) {
    this.state = 'pending'; this.value = undefined; this.cbs = [];
    const settle = (state, value) => {
      if (this.state !== 'pending') return;
      if (state === 'fulfilled' && value && typeof value.then === 'function')
        return value.then(v => settle('fulfilled', v), e => settle('rejected', e));
      this.state = state; this.value = value;
      queueMicrotask(() => { this.cbs.forEach(cb => cb()); this.cbs = []; });
    };
    try { executor(v => settle('fulfilled', v), e => settle('rejected', e)); }
    catch (e) { settle('rejected', e); }
  }
  then(onOk, onErr) {
    return new MyPromise((res, rej) => {
      const run = () => queueMicrotask(() => {
        try {
          const h = this.state === 'fulfilled' ? onOk : onErr;
          if (typeof h !== 'function')
            return this.state === 'fulfilled' ? res(this.value) : rej(this.value);
          res(h(this.value));
        } catch (e) { rej(e); }
      });
      this.state === 'pending' ? this.cbs.push(run) : run();
    });
  }
  catch(fn) { return this.then(null, fn); }
}
```

### 122. What is promisification of an event-based API? 🟠
```js
const { once } = require('node:events');
const [socket] = await once(server, 'connection');
```

### 123. What is `events.once` vs `emitter.once`? 🔵
`events.once(emitter, name)` returns a promise for the next emission and rejects on `'error'` — safer for one-shot awaits.

### 124. What is an async iterator? 🟠
```js
async function* pages(url) {
  let next = url;
  while (next) {
    const res = await fetch(next).then(r => r.json());
    yield* res.items;
    next = res.nextUrl;
  }
}
for await (const item of pages('/api/items')) console.log(item);
```

### 125. What is `Symbol.asyncIterator`? 🟠
The protocol `for await...of` looks up. Streams implement it, which is why you can iterate a Readable.

```js
for await (const chunk of fs.createReadStream('big.log')) process(chunk);
```

### 126. How do you convert an event emitter to an async iterable? 🟠
```js
const { on } = require('node:events');
for await (const [data] of on(emitter, 'data')) { /* ... */ }
```

### 127. What is the difference between returning and awaiting a promise in a `try` block? 🟠
`return promise` escapes the `try` before rejection is observed; `return await promise` keeps the `catch` in scope.

```js
try { return await risky(); } catch (e) { handle(e); } // ✅
```

### 128. Why avoid `async` executors in `new Promise`? 🟠
Throws inside an async executor become an unhandled rejection instead of rejecting the outer promise.

### 129. What is the "async in forEach" trap? 🔵
`Array.prototype.forEach` ignores returned promises, so nothing is awaited.

```js
// ❌ finishes instantly
items.forEach(async i => await save(i));
// ✅
for (const i of items) await save(i);
await Promise.all(items.map(save));
```

### 130. How do you handle partial failures in a batch? 🔵
```js
const results = await Promise.allSettled(jobs.map(run));
const failed = results.filter(r => r.status === 'rejected');
```

### 131. What is a deferred/resolvers pattern? 🔵
```js
const { promise, resolve, reject } = Promise.withResolvers(); // Node 22+
```

### 132. How do you implement a work queue with backpressure? 🟠
Bound the queue length and reject/park producers when full — otherwise memory grows without limit under load.

### 133. What is `structuredClone` in Node? 🔵
Global deep-clone supporting Maps, Sets, Dates, TypedArrays, and cycles (not functions or class prototypes).

### 134. What is a race condition example in Node despite single-threading? 🟠
Interleaving at `await` points: two requests read-modify-write the same row and one overwrites the other. Fix with DB transactions, optimistic locking (`version` column), or an in-process mutex keyed by resource.

### 135. How do you implement idempotency for an API? 🟠
Client sends an `Idempotency-Key`; server stores key → response atomically (`INSERT ... ON CONFLICT DO NOTHING`) and replays the stored response on retry.

### 136. What is `p-queue`-style priority scheduling? 🟠
Maintain a heap of tasks by priority; the runner pulls the highest priority while respecting a concurrency cap.

### 137. What is `Promise.try` / why wrap sync code? 🔵
Turns a synchronous throw into a rejection so callers only need one error path.

```js
Promise.resolve().then(() => mightThrowSync());
```

### 138. How do you cancel an in-flight async pipeline? 🟠
Thread an `AbortSignal` through every layer and check `signal.throwIfAborted()` between steps.

### 139. What is a semaphore and where would you use it in Node? 🟠
Limit concurrent DB connections, outbound HTTP calls, or thread-pool-bound crypto so you don't saturate a shared resource.

### 140. Explain the microtask ordering of nested awaits. 🔴
```js
async function a() { console.log('a1'); await b(); console.log('a2'); }
async function b() { console.log('b1'); }
a(); Promise.resolve().then(() => console.log('p'));
// a1, b1, p, a2  — the await defers a2 by one microtask, letting p run first
```

---

## 5. Events, EventEmitter & Observability (141–160)

### 141. What is `EventEmitter`? 🟢
Node's pub/sub base class — the backbone of streams, servers and sockets.

```js
const { EventEmitter } = require('node:events');
class Job extends EventEmitter {}
const job = new Job();
job.on('done', r => console.log(r));
job.emit('done', 42);
```

### 142. `on` vs `once` vs `off`/`removeListener`? 🟢
Persistent · single-shot (auto-removed) · detach. Always keep a reference to the exact function to remove it.

### 143. Is `emit` synchronous? 🔵
Yes — listeners run synchronously in registration order, before `emit()` returns. This surprises people expecting queuing.

```js
e.on('x', () => console.log('L')); e.emit('x'); console.log('after'); // L, after
```

### 144. What happens on an `'error'` event with no listener? 🔵
Node throws it as an uncaught exception and the process crashes. Always attach an error listener to emitters and streams.

### 145. What is `captureRejections`? 🟠
Routes rejected promises returned by async listeners to the emitter's `'error'` event.

```js
const e = new EventEmitter({ captureRejections: true });
e.on('error', console.error);
e.on('job', async () => { throw new Error('boom'); });
```

### 146. What is `MaxListenersExceededWarning`? 🔵
A leak heuristic at 11 listeners for one event. Fix the leak, or raise deliberately with `emitter.setMaxListeners(n)` / `events.defaultMaxListeners`.

### 147. How do you inspect current listeners? 🔵
```js
console.log(e.eventNames(), e.listenerCount('data'), e.rawListeners('data'));
```

### 148. `prependListener` — why? 🟠
Run before existing handlers, e.g. to install instrumentation ahead of user code.

### 149. Implement your own EventEmitter. 🟠
```js
class Emitter {
  #m = new Map();
  on(e, fn) { (this.#m.get(e) ?? this.#m.set(e, []).get(e)).push(fn); return this; }
  once(e, fn) { const w = (...a) => { this.off(e, w); fn(...a); }; w.fn = fn; return this.on(e, w); }
  off(e, fn) { const l = this.#m.get(e) || []; const i = l.findIndex(x => x === fn || x.fn === fn);
               if (i > -1) l.splice(i, 1); return this; }
  emit(e, ...a) { const l = this.#m.get(e); if (!l?.length) return false;
                  [...l].forEach(f => f(...a)); return true; }
}
```

### 150. EventEmitter vs EventTarget in Node? 🟠
`EventTarget`/`CustomEvent` are the web-standard versions available globally; used by `AbortSignal`. `EventEmitter` is Node-idiomatic and faster for high-frequency internal events.

### 151. How do EventEmitters cause memory leaks? 🟠
Listeners hold references to closures (and everything they capture). Long-lived emitters + per-request listeners = unbounded growth. Remove listeners on cleanup or use `once`/`AbortSignal`.

```js
emitter.on('tick', handler, { signal: ac.signal }); // auto-removed on abort (EventTarget)
```

### 152. What is the observer pattern's downside here? 🟠
Implicit control flow, no type safety, and errors thrown in a listener propagate to the emitter's call site.

### 153. How would you build a typed event emitter in TypeScript? 🟠
```ts
type Events = { done: [id: string]; error: [err: Error] };
class Typed<E extends Record<string, unknown[]>> extends EventEmitter {
  on<K extends keyof E & string>(e: K, fn: (...a: E[K]) => void) { return super.on(e, fn as any); }
  emit<K extends keyof E & string>(e: K, ...a: E[K]) { return super.emit(e, ...a); }
}
```

### 154. How do you make an emitter async-safe? 🟠
Wrap listeners with `AsyncResource.bind` so `AsyncLocalStorage` context survives.

### 155. What is structured logging and why does it matter? 🔵
Emit JSON with stable fields (`level`, `msg`, `reqId`, `latencyMs`) so log aggregators can index and alert. `pino` is the fast standard.

```js
const logger = require('pino')({ level: process.env.LOG_LEVEL || 'info' });
logger.info({ reqId, route: '/users' }, 'handled');
```

### 156. Why is `console.log` a bad production logger? 🔵
No levels, no structure, synchronous on TTY, and it stringifies objects expensively.

### 157. What are the three pillars of observability? 🟠
Logs (events), metrics (aggregates — RED/USE), traces (causal spans). OpenTelemetry covers all three in Node.

```js
// node --require ./tracing.js app.js
const { NodeSDK } = require('@opentelemetry/sdk-node');
new NodeSDK({ instrumentations: [getNodeAutoInstrumentations()] }).start();
```

### 158. How do you propagate a correlation ID? 🟠
Read/create `x-request-id` in middleware, store in `AsyncLocalStorage`, include in every log line and outbound header.

### 159. What metrics would you expose from a Node service? 🟠
RPS, latency histogram (p50/p95/p99), error rate, event loop delay, heap used/total, active handles, GC pause, external (Buffer) memory.

### 160. How do you implement a healthcheck vs readiness check? 🔵
Liveness = "process is responsive" (cheap, no deps). Readiness = "dependencies are reachable and loop lag is acceptable" — used to gate traffic.

---
## 6. Buffers, Streams & Backpressure (161–195)

### 161. Why does `Buffer` exist? 🟢
JS strings are UTF-16 and immutable; binary protocols need raw bytes. `Buffer` is a `Uint8Array` subclass backed by memory outside the V8 heap.

### 162. `Buffer.alloc` vs `allocUnsafe` vs `from`? 🔵
`alloc` zero-fills (safe). `allocUnsafe` skips zeroing — faster but may expose old memory contents; overwrite fully before use. `from` copies existing data.

```js
Buffer.alloc(8);                 // <Buffer 00 ... 00>
Buffer.allocUnsafe(8);           // arbitrary bytes
Buffer.from([0xde, 0xad]);
```

### 163. What encodings does Buffer support? 🔵
`utf8`, `utf16le`, `latin1`, `base64`, `base64url`, `hex`, `ascii`.

```js
Buffer.from('a+b/c=', 'base64url').toString('hex');
```

### 164. How do you concatenate buffers efficiently? 🔵
```js
const out = Buffer.concat(chunks, totalLength); // pass length to skip a pass
```

### 165. What is `Buffer.poolSize`? 🟠
Small allocations (<4KB by default via `allocUnsafe`) are carved out of a shared 8KB pool — so slicing may share memory with unrelated data. Use `Buffer.allocUnsafeSlow` for isolated allocations you'll retain.

### 166. Does `buf.slice()` copy? 🟠
No — `subarray`/`slice` return a **view** over the same memory. Mutating one mutates the other. Use `Buffer.from(buf)` to copy.

```js
const a = Buffer.from('abcdef'); const b = a.subarray(0, 3);
b[0] = 0x7a; console.log(a.toString()); // 'zbcdef'
```

### 167. Buffer vs TypedArray vs ArrayBuffer? 🟠
`ArrayBuffer` is the raw memory; `TypedArray`/`DataView` are typed views; `Buffer` is a `Uint8Array` with Node conveniences.

```js
const u8 = new Uint8Array(4);
const buf = Buffer.from(u8.buffer, u8.byteOffset, u8.byteLength); // no copy
```

### 168. How do you read binary numbers from a buffer? 🟠
```js
const b = Buffer.alloc(8);
b.writeUInt32BE(0xdeadbeef, 0);
b.writeBigInt64LE(123n, 0);
console.log(b.readUInt32BE(0));
```

### 169. What is a string decoder and why is it needed? 🟠
A multi-byte UTF-8 character can be split across chunk boundaries. `StringDecoder` buffers the incomplete tail.

```js
const { StringDecoder } = require('node:string_decoder');
const dec = new StringDecoder('utf8');
stream.on('data', c => process.stdout.write(dec.write(c)));
stream.on('end', () => process.stdout.write(dec.end()));
```

### 170. What are the four stream types? 🟢
Readable (source), Writable (sink), Duplex (both, e.g. TCP socket), Transform (Duplex where output derives from input, e.g. gzip).

### 171. Why use streams instead of reading the whole file? 🟢
Constant memory, lower time-to-first-byte, and composability. A 4GB file will OOM `readFile` but stream fine.

```js
fs.createReadStream('big.csv').pipe(parse()).pipe(res);
```

### 172. What is flowing vs paused mode? 🟠
Attaching `'data'` or calling `pipe()` switches to flowing (data pushed at you). Paused mode uses `'readable'` + `read()` for pull-based control. `pause()`/`resume()` toggle.

```js
rs.on('readable', () => { let c; while ((c = rs.read()) !== null) handle(c); });
```

### 173. What is backpressure? 🟠
When a slow consumer can't keep up with a fast producer. `write()` returns `false` when the internal buffer exceeds `highWaterMark`; the producer should stop until `'drain'`.

```js
function writeAll(ws, chunks, done) {
  (function next() {
    while (chunks.length) if (!ws.write(chunks.shift())) return ws.once('drain', next);
    done();
  })();
}
```

### 174. What does `pipe()` do about backpressure? 🟠
It wires up `data`/`drain`/`pause`/`resume` automatically — that's its main value over manual `on('data')`.

### 175. Why prefer `pipeline()` over `pipe()`? 🟠
`pipe()` does **not** forward errors or destroy the remaining streams on failure — a classic file-descriptor leak. `pipeline` handles errors and cleanup.

```js
const { pipeline } = require('node:stream/promises');
await pipeline(
  fs.createReadStream('in.txt'),
  zlib.createGzip(),
  fs.createWriteStream('out.gz')
);
```

### 176. What is `highWaterMark`? 🟠
The buffer threshold: 16KB for byte streams, 16 objects for object mode. Raising it trades memory for fewer syscalls/context switches.

### 177. What is object mode? 🔵
`{ objectMode: true }` lets streams carry arbitrary JS values instead of bytes — used by CSV parsers and DB cursors.

### 178. Write a custom Readable. 🟠
```js
const { Readable } = require('node:stream');
class Counter extends Readable {
  constructor(max) { super({ objectMode: true }); this.i = 0; this.max = max; }
  _read() { this.push(this.i < this.max ? { n: this.i++ } : null); } // null = EOF
}
```

### 179. Write a custom Writable. 🟠
```js
const { Writable } = require('node:stream');
class DbSink extends Writable {
  constructor(db) { super({ objectMode: true, highWaterMark: 100 }); this.db = db; }
  _write(row, _enc, cb) { this.db.insert(row).then(() => cb(), cb); }
  _final(cb) { this.db.flush().then(() => cb(), cb); }
}
```

### 180. Write a custom Transform. 🟠
```js
const { Transform } = require('node:stream');
const upper = new Transform({
  transform(chunk, _enc, cb) { cb(null, chunk.toString().toUpperCase()); },
  flush(cb) { cb(null, '\n-- EOF --\n'); }
});
```

### 181. What is `_writev`? 🔴
Batch write hook — receives an array of buffered chunks so you can do one syscall/round trip instead of N.

### 182. What is `Readable.from()`? 🔵
Builds a stream from any iterable or async iterable.

```js
const rs = Readable.from(async function* () { yield 'a'; yield 'b'; }());
```

### 183. How do you use streams as async iterators safely? 🟠
`for await` handles backpressure automatically, but breaking early destroys the stream — which is usually what you want.

```js
for await (const line of rl) { if (done) break; }
```

### 184. How do you read a file line by line? 🔵
```js
const rl = require('node:readline').createInterface({
  input: fs.createReadStream('big.log'), crlfDelay: Infinity
});
for await (const line of rl) console.log(line);
```

### 185. What are Web Streams in Node? 🟠
WHATWG `ReadableStream`/`WritableStream`/`TransformStream` are global. Convert with `Readable.fromWeb`/`Readable.toWeb`.

```js
const webStream = Readable.toWeb(fs.createReadStream('a.txt'));
```

### 186. How do you gzip/gunzip a stream? 🔵
```js
await pipeline(fs.createReadStream('a.log'), zlib.createGzip({ level: 6 }), fs.createWriteStream('a.log.gz'));
```

### 187. What is `stream.finished`? 🟠
Reliable completion detection across all stream types, including errors and premature close.

```js
const { finished } = require('node:stream/promises');
await finished(readable);
```

### 188. How do you handle a stream error correctly? 🟠
Attach `'error'` on every stream (or use `pipeline`), and `destroy(err)` to propagate.

```js
rs.on('error', e => { ws.destroy(e); });
```

### 189. What is `stream.destroy()` vs `end()`? 🟠
`end()` finishes gracefully after flushing; `destroy()` tears down immediately, optionally with an error, emitting `'close'`.

### 190. How do you tee/fork a stream to two destinations? 🟠
```js
const pass = new PassThrough();
rs.pipe(pass); pass.pipe(fileWs); pass.pipe(hash);
```
Beware: the slowest consumer sets the pace, and one destination erroring won't stop the other without `pipeline`.

### 191. How do you compute a checksum while streaming? 🟠
```js
const hash = crypto.createHash('sha256');
await pipeline(fs.createReadStream(f), hash);       // hash is a Transform
console.log(hash.digest('hex'));
```

### 192. How do you stream an upload straight to S3 without buffering? 🔴
Pipe the request stream into the SDK's multipart upload; never buffer the whole body in memory. Enforce a max size by counting bytes and destroying on overflow.

```js
let bytes = 0;
req.on('data', c => { if ((bytes += c.length) > MAX) req.destroy(new Error('TOO_LARGE')); });
```

### 193. What is `stream.compose`? 🔴
Combines multiple streams into one Duplex so a pipeline segment can be reused as a unit.

```js
const { compose } = require('node:stream');
const gzipAndHash = compose(zlib.createGzip(), hashTransform);
```

### 194. How do you implement a rate-limited stream? 🔴
```js
class Throttle extends Transform {
  constructor(bytesPerSec) { super(); this.rate = bytesPerSec; }
  _transform(chunk, _e, cb) {
    setTimeout(() => cb(null, chunk), (chunk.length / this.rate) * 1000);
  }
}
```

### 195. What's a common streams memory leak? 🟠
Not consuming a Readable (it buffers to `highWaterMark` and holds the fd), or `pipe()` without error handling leaving destroyed-source/alive-destination pairs. Use `pipeline` + `AbortSignal`.

---

## 7. File System, Path & OS (196–212)

### 196. What are the three `fs` APIs? 🟢
Callback (`fs`), promise (`fs/promises`), and synchronous (`fs.*Sync`). Prefer promises in application code.

```js
const fsp = require('node:fs/promises');
const data = await fsp.readFile('a.json', 'utf8');
```

### 197. How do you check if a file exists? 🔵
Don't pre-check — it's a TOCTOU race. Just attempt the operation and handle `ENOENT`.

```js
try { await fsp.access(p, fs.constants.R_OK); } catch { /* missing */ }
```

### 198. What are common fs error codes? 🔵
`ENOENT` (no such file), `EACCES`/`EPERM` (permission), `EEXIST`, `EISDIR`/`ENOTDIR`, `EMFILE` (too many open files), `ENOSPC`.

### 199. How do you handle `EMFILE`? 🟠
Limit concurrent opens (semaphore or `graceful-fs`), close descriptors in `finally`, and raise `ulimit -n`.

### 200. How do you create nested directories? 🟢
```js
await fsp.mkdir('a/b/c', { recursive: true });
```

### 201. How do you remove a directory tree? 🔵
```js
await fsp.rm('build', { recursive: true, force: true });
```

### 202. How do you walk a directory tree? 🔵
```js
for (const e of await fsp.readdir('.', { withFileTypes: true, recursive: true }))
  if (e.isFile()) console.log(path.join(e.parentPath, e.name));
```

### 203. What do the file open flags mean? 🔵
`r` read · `w` truncate/create · `a` append · `x` fail if exists · `+` adds the opposite mode.

```js
await fsp.writeFile('lock', '', { flag: 'wx' }); // atomic-ish lock creation
```

### 204. How do you write a file atomically? 🟠
Write to a temp file on the same filesystem, `fsync`, then `rename` (atomic on POSIX).

```js
const tmp = target + '.tmp';
const fh = await fsp.open(tmp, 'w');
await fh.writeFile(data); await fh.sync(); await fh.close();
await fsp.rename(tmp, target);
```

### 205. `fs.watch` vs `fs.watchFile`? 🟠
`watch` uses OS notifications (fast, but duplicate/missing events and no recursion on Linux <20). `watchFile` polls `stat` (portable, higher latency and CPU). For robust watching use `chokidar`.

### 206. `path.join` vs `path.resolve`? 🟢
`join` concatenates and normalizes; `resolve` produces an absolute path, processing right-to-left until it hits an absolute segment.

```js
path.join('/a', 'b', '../c'); // '/a/c'
path.resolve('a', '/b', 'c'); // '/b/c'
```

### 207. How do you prevent path traversal? 🟠
```js
const ROOT = path.resolve('./uploads');
const full = path.resolve(ROOT, userInput);
if (!full.startsWith(ROOT + path.sep)) throw new Error('Invalid path');
```

### 208. What is `path.posix` / `path.win32`? 🔵
Force a specific platform's semantics regardless of the host — useful when handling URLs or archive entries on Windows.

### 209. What does `fs.stat` give you? 🔵
```js
const s = await fsp.stat(f);
console.log(s.size, s.mtimeMs, s.isDirectory(), s.mode.toString(8));
```
`lstat` doesn't follow symlinks.

### 210. How do you copy files/directories? 🔵
```js
await fsp.copyFile('a', 'b', fs.constants.COPYFILE_EXCL);
await fsp.cp('src', 'dst', { recursive: true });
```

### 211. What useful info does `os` provide? 🔵
```js
const os = require('node:os');
os.cpus().length; os.totalmem(); os.freemem(); os.tmpdir(); os.homedir(); os.loadavg();
```
In containers, `os.cpus()` reports host CPUs, not the cgroup quota — read `/sys/fs/cgroup/cpu.max` or use `os.availableParallelism()`.

### 212. How do you create a temp directory? 🔵
```js
const dir = await fsp.mkdtemp(path.join(os.tmpdir(), 'job-'));
```

---
## 8. HTTP, Networking & Express (213–245)

### 213. How do you create a bare HTTP server? 🟢
```js
const http = require('node:http');
http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ ok: true }));
}).listen(3000);
```

### 214. `req` and `res` are what, really? 🔵
`req` is a Readable stream (`IncomingMessage`), `res` is a Writable stream (`ServerResponse`). That's why you can pipe them.

### 215. How do you parse a JSON body without a framework? 🔵
```js
async function readJson(req, limit = 1e6) {
  let size = 0; const chunks = [];
  for await (const c of req) {
    if ((size += c.length) > limit) { req.destroy(); throw new Error('PAYLOAD_TOO_LARGE'); }
    chunks.push(c);
  }
  return JSON.parse(Buffer.concat(chunks, size).toString('utf8'));
}
```

### 216. How do you parse the query string? 🟢
```js
const { searchParams, pathname } = new URL(req.url, `http://${req.headers.host}`);
console.log(pathname, searchParams.get('page'));
```

### 217. What is Keep-Alive and why does it matter? 🟠
Reusing a TCP connection avoids the handshake (and TLS handshake) per request. Node's server enables it by default; for outbound calls you must configure the agent.

```js
const agent = new http.Agent({ keepAlive: true, maxSockets: 100, maxFreeSockets: 10 });
```

### 218. What is `undici` / global `fetch`? 🔵
`fetch` is built in since Node 18, powered by `undici`. For pooling, retries and interceptors use undici's `Agent`/`Pool` directly.

```js
const { request } = require('undici');
const { statusCode, body } = await request('https://api.example.com/x');
console.log(await body.json());
```

### 219. Common HTTP timeouts to configure? 🟠
```js
server.headersTimeout = 65_000;      // time to receive headers
server.requestTimeout = 300_000;     // whole request
server.keepAliveTimeout = 61_000;    // must exceed the LB's idle timeout
```
A `keepAliveTimeout` shorter than the load balancer's causes intermittent 502s.

### 220. How do you stream a file download with range support? 🟠
```js
const { size } = await fsp.stat(file);
const m = /bytes=(\d*)-(\d*)/.exec(req.headers.range || '');
if (m) {
  const start = Number(m[1] || 0), end = Number(m[2] || size - 1);
  res.writeHead(206, { 'Content-Range': `bytes ${start}-${end}/${size}`,
    'Accept-Ranges': 'bytes', 'Content-Length': end - start + 1 });
  fs.createReadStream(file, { start, end }).pipe(res);
} else { res.writeHead(200, { 'Content-Length': size }); fs.createReadStream(file).pipe(res); }
```

### 221. How do you handle file uploads? 🔵
`multipart/form-data` via `multer`/`busboy` streaming to disk or S3. Enforce size, count and MIME limits, and never trust the client-supplied filename.

### 222. What is `net` used for? 🔵
Raw TCP.

```js
require('node:net').createServer(sock => sock.pipe(sock)).listen(9000); // echo
```

### 223. What is `dgram`? 🔵
UDP sockets — fire-and-forget, used for StatsD metrics, DNS, discovery.

### 224. How do you implement WebSockets? 🔵
```js
const { WebSocketServer } = require('ws');
const wss = new WebSocketServer({ server });
wss.on('connection', ws => {
  ws.on('message', m => ws.send(`echo: ${m}`));
});
```
Add a ping/pong heartbeat to reap dead connections.

### 225. WebSocket vs SSE vs long polling? 🟠
WS = bidirectional, custom protocol, needs sticky sessions/scale-out pub-sub. SSE = server→client only, plain HTTP, auto-reconnect. Long polling = fallback, highest overhead.

```js
res.writeHead(200, { 'Content-Type': 'text/event-stream', 'Cache-Control': 'no-cache', Connection: 'keep-alive' });
setInterval(() => res.write(`data: ${JSON.stringify({ t: Date.now() })}\n\n`), 1000);
```

### 226. How do you scale Socket.IO across instances? 🟠
A Redis (or NATS) adapter broadcasts events between processes; sticky sessions or WebSocket-only transport avoid handshake mismatches.

### 227. What is HTTP/2 in Node? 🟠
```js
const http2 = require('node:http2');
http2.createSecureServer({ key, cert }, (req, res) => res.end('ok')).listen(443);
```
Multiplexing removes head-of-line blocking at the HTTP layer; server push is deprecated.

### 228. How do you enable HTTPS? 🔵
```js
require('node:https').createServer({ key: fs.readFileSync('k.pem'), cert: fs.readFileSync('c.pem') }, app).listen(443);
```
In production, TLS is usually terminated at the LB/ingress.

### 229. What is Express middleware? 🟢
`(req, res, next)` functions executed in order; they can end the response or delegate. Error middleware has four params.

```js
app.use((req, res, next) => { req.startedAt = Date.now(); next(); });
app.use((err, req, res, next) => res.status(err.status || 500).json({ error: err.message }));
```

### 230. How do you handle async errors in Express 4? 🟠
Rejections aren't caught automatically — wrap handlers.

```js
const ah = fn => (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);
app.get('/u/:id', ah(async (req, res) => res.json(await getUser(req.params.id))));
```
Express 5 forwards rejected promises to the error handler natively.

### 231. What is the order of Express route matching? 🔵
First registered match wins; `app.use` mounts by path prefix. Register the 404 handler last.

### 232. `app.use` vs `router.use` vs `app.all`? 🔵
App-level middleware · modular sub-router mounted at a prefix · all HTTP verbs for one path.

### 233. What is `express.json()` doing? 🔵
Body-parsing middleware with a `limit` (default 100kb) that populates `req.body`; it only runs when `Content-Type` matches.

### 234. How do you implement rate limiting? 🟠
```js
const rateLimit = require('express-rate-limit');
app.use(rateLimit({ windowMs: 60_000, limit: 100, standardHeaders: 'draft-7' }));
```
For multiple instances, back it with Redis so the counter is shared. Algorithms: fixed window (bursty at boundaries), sliding window, token bucket (best for bursts).

### 235. Express vs Fastify vs NestJS vs Koa? 🔵
Express: ubiquitous, minimal, slower. Fastify: schema-based validation/serialization, ~2-3x faster JSON. Koa: async middleware with a context object. Nest: opinionated DI/decorator architecture on top of Express or Fastify.

### 236. How do you validate request input? 🔵
```js
const { z } = require('zod');
const Body = z.object({ email: z.string().email(), age: z.number().int().min(0) });
app.post('/u', (req, res) => {
  const parsed = Body.safeParse(req.body);
  if (!parsed.success) return res.status(400).json(parsed.error.flatten());
  // parsed.data is typed & sanitized
});
```

### 237. How do you implement JWT auth? 🔵
```js
const jwt = require('jsonwebtoken');
const token = jwt.sign({ sub: user.id, role: user.role }, SECRET, { expiresIn: '15m' });
function auth(req, res, next) {
  const t = req.headers.authorization?.split(' ')[1];
  try { req.user = jwt.verify(t, SECRET); next(); }
  catch { res.sendStatus(401); }
}
```

### 238. JWT vs sessions? 🟠
JWTs are stateless and scale horizontally but can't be revoked before expiry — use short access tokens + a rotating refresh token stored server-side. Sessions are revocable but need a shared store (Redis).

### 239. How do you set secure cookies? 🟠
```js
res.cookie('sid', id, { httpOnly: true, secure: true, sameSite: 'lax', maxAge: 864e5, signed: true });
```

### 240. How do you configure CORS correctly? 🟠
```js
app.use(require('cors')({
  origin: (o, cb) => cb(null, ALLOWED.includes(o)), credentials: true,
  methods: ['GET','POST'], maxAge: 600
}));
```
`origin: '*'` is incompatible with `credentials: true`.

### 241. What does `app.set('trust proxy', 1)` do? 🟠
Makes Express honour `X-Forwarded-For`/`Proto` so `req.ip` and `req.secure` are correct behind a load balancer — critical for rate limiting and redirects.

### 242. How do you version an API? 🔵
URL prefix (`/v1/`), header-based (`Accept: application/vnd.api+json;version=1`), or additive-only evolution. URL versioning is the most operationally simple.

### 243. What are idempotent and safe HTTP methods? 🔵
Safe: GET, HEAD, OPTIONS. Idempotent: safe + PUT, DELETE. POST is neither — hence idempotency keys.

### 244. How do you implement caching headers? 🟠
```js
res.set('Cache-Control', 'public, max-age=300, stale-while-revalidate=60');
res.set('ETag', etag);
if (req.headers['if-none-match'] === etag) return res.status(304).end();
```

### 245. How do you serve static files efficiently? 🔵
`express.static` with `maxAge` and `immutable` for hashed assets — but in production put a CDN or nginx in front; Node shouldn't be your static file server.

---

## 9. Error Handling, Debugging & Testing (246–262)

### 246. Operational vs programmer errors? 🟠
Operational = expected runtime conditions (network down, invalid input) → handle and recover. Programmer = bugs (`undefined is not a function`) → crash and restart. Conflating them leads to processes limping along in a corrupt state.

### 247. How do you build a custom error class? 🔵
```js
class AppError extends Error {
  constructor(message, { status = 500, code = 'INTERNAL', cause } = {}) {
    super(message, { cause });
    this.name = this.constructor.name;
    Object.assign(this, { status, code, isOperational: true });
    Error.captureStackTrace(this, this.constructor);
  }
}
class NotFound extends AppError {
  constructor(what) { super(`${what} not found`, { status: 404, code: 'NOT_FOUND' }); }
}
```

### 248. What is `error.cause`? 🔵
Standard error chaining — preserve the low-level error while adding context.

```js
throw new Error('Failed to load user', { cause: dbErr });
```

### 249. What is `AggregateError`? 🔵
Produced by `Promise.any` when everything rejects; `err.errors` is the array.

### 250. How do you centralize error handling? 🟠
One error middleware/handler that maps error types → status codes, logs with request context, hides internals from clients, and increments metrics.

```js
app.use((err, req, res, _next) => {
  const status = err.isOperational ? err.status : 500;
  logger.error({ err, reqId: req.id }, 'request failed');
  res.status(status).json({ code: err.code ?? 'INTERNAL', message: status === 500 ? 'Internal error' : err.message });
});
```

### 251. Why not swallow errors? 🔵
`catch {}` hides failures until they surface as corrupt data. Log, translate, or rethrow — but always do something.

### 252. How do you debug Node? 🔵
`node --inspect-brk`, VS Code attach, `debugger` statements, `NODE_DEBUG=http,net node app.js` for core internals, `util.inspect` with `{ depth: null }`.

### 253. What is `--enable-source-maps`? 🔵
Maps stack traces of compiled TS/bundled code back to original sources.

### 254. What is the built-in test runner? 🔵
```js
const { test, describe, before, mock } = require('node:test');
const assert = require('node:assert/strict');

describe('sum', () => {
  test('adds', () => assert.equal(sum(1, 2), 3));
  test('async', async () => assert.deepEqual(await load(), { ok: true }));
});
// node --test --experimental-test-coverage
```

### 255. Unit vs integration vs e2e tests? 🔵
Isolated logic with mocks · real collaborators (DB, HTTP) usually via Testcontainers · full system through the public interface. Aim for many unit, fewer integration, few e2e.

### 256. How do you test an Express app? 🔵
```js
const request = require('supertest');
await request(app).post('/users').send({ email: 'a@b.c' }).expect(201);
```

### 257. How do you mock modules/timers? 🟠
```js
const { mock } = require('node:test');
const t = mock.timers; t.enable({ apis: ['setTimeout'] });
scheduleThing(); t.tick(5000);
mock.method(db, 'query', async () => [{ id: 1 }]);
```

### 258. How do you test code that uses a real database? 🟠
Spin up a throwaway container per suite, run migrations, wrap each test in a transaction that's rolled back — deterministic and parallel-safe.

### 259. What is a flaky test and how do you fix it? 🟠
Non-determinism from real timers, shared state, ordering, or network. Fix with fake timers, isolated fixtures, and explicit waits on conditions instead of `sleep`.

### 260. What is code coverage and its limits? 🔵
Percentage of lines/branches executed. High coverage with no assertions proves nothing; use it to find untested areas, not as a quality score.

### 261. What linting/formatting setup would you use? 🔵
ESLint (flat config) + Prettier + `typescript-eslint`, enforced in CI and via a pre-commit hook (husky + lint-staged).

### 262. How do you write a load test? 🟠
`autocannon -c 100 -d 30 http://localhost:3000/api` or k6 for scenarios. Measure p99 latency and error rate under fixed RPS, not just throughput; test with production-like data volume.

---
## 10. Performance, Cluster, Worker Threads & Memory (263–283)

### 263. What is the `cluster` module? 🟠
Forks N worker processes sharing one listening socket, so a multi-core machine isn't limited to one CPU.

```js
const cluster = require('node:cluster');
const os = require('node:os');
if (cluster.isPrimary) {
  for (let i = 0; i < os.availableParallelism(); i++) cluster.fork();
  cluster.on('exit', (w, code) => { console.log('worker died', w.process.pid, code); cluster.fork(); });
} else {
  require('./server');
}
```

### 264. How does cluster distribute connections? 🟠
Default on Linux is round-robin in the primary (`SCHED_RR`); on Windows the OS distributes (which can be badly unbalanced). Set with `cluster.schedulingPolicy`.

### 265. Cluster vs PM2 vs Kubernetes? 🟠
Cluster = in-process supervision. PM2 = process manager with clustering, restarts, log rotation. K8s = one Node process per container, scaled by replicas — usually preferred, since the orchestrator already handles restarts and rollouts.

### 266. What is `worker_threads` and when do you use it? 🟠
True threads sharing memory within one process — for CPU-bound work (image processing, parsing, crypto) without paying process-fork overhead.

```js
// main.js
const { Worker } = require('node:worker_threads');
const w = new Worker('./heavy.js', { workerData: { n: 42 } });
w.on('message', r => console.log('result', r));
w.on('error', console.error);
w.on('exit', c => c && console.error('exit', c));

// heavy.js
const { parentPort, workerData } = require('node:worker_threads');
parentPort.postMessage(fib(workerData.n));
```

### 267. Cluster vs worker_threads? 🟠
Cluster: separate processes, separate heaps, isolation, good for scaling I/O-bound HTTP. Workers: shared process, cheaper messaging, `SharedArrayBuffer`, good for CPU work. A crash in a worker doesn't kill the process; a crash in a cluster worker doesn't kill siblings.

### 268. How does data pass between threads? 🟠
Structured clone by default (a copy). Use `transferList` to move an `ArrayBuffer` with zero copy, or `SharedArrayBuffer` + `Atomics` for shared state.

```js
const buf = new ArrayBuffer(1e7);
worker.postMessage({ buf }, [buf]); // buf is now detached in the sender
```

### 269. What is a worker pool and why? 🔴
Spawning a thread per task costs ~10-40ms and memory. Pool them.

```js
const { Piscina } = require('piscina');
const pool = new Piscina({ filename: './task.js', maxThreads: 4 });
const result = await pool.run({ n: 42 });
```

### 270. What is `Atomics.wait` and can you use it on the main thread? 🔴
It blocks the thread until notified. **Not** allowed on the main thread in Node's default configuration for good reason — it would freeze the event loop. Use it inside workers for coordination.

### 271. `child_process`: `spawn` vs `exec` vs `execFile` vs `fork`? 🟠
`spawn` streams output (no buffer limit) · `exec` runs through a shell and buffers into memory (`maxBuffer`, injection risk) · `execFile` no shell, safer · `fork` spawns a Node script with an IPC channel.

```js
const { spawn } = require('node:child_process');
const p = spawn('ffmpeg', ['-i', input, output]);
p.stderr.on('data', d => log(d.toString()));
p.on('close', code => console.log('exit', code));
```

### 272. How do you avoid command injection with child_process? 🟠
Never interpolate user input into `exec`. Use `execFile`/`spawn` with an argument array and `shell: false`.

```js
execFile('git', ['log', '--author', userInput]); // safe
exec(`git log --author ${userInput}`);           // ❌ injectable
```

### 273. How do you communicate with a forked child? 🔵
```js
const child = require('node:child_process').fork('./job.js');
child.send({ task: 'build' });
child.on('message', m => console.log(m));
```

### 274. How does V8 garbage collection work? 🔴
Generational: a small **new space** collected by a fast copying scavenger (most objects die young), and an **old space** collected by mark-sweep-compact, largely concurrent/incremental (Orinoco) to shorten pauses. Objects surviving two scavenges are promoted.

### 275. What causes memory leaks in Node? 🟠
Unbounded caches/Maps, forgotten event listeners, timers holding closures, module-level accumulating arrays, global variables, unclosed streams/DB handles, and closures capturing large buffers.

```js
const cache = new Map(); // ❌ grows forever
const cache2 = new LRUCache({ max: 1000, ttl: 60_000 }); // ✅ bounded
```

### 276. How do you find a memory leak? 🟠
Take two heap snapshots under steady load, compare retained size by constructor, follow the retainer path to the GC root.

```js
require('node:v8').writeHeapSnapshot('/tmp/heap-1.heapsnapshot');
console.log(process.memoryUsage()); // rss, heapUsed, heapTotal, external, arrayBuffers
```

### 277. What does each `memoryUsage()` field mean? 🟠
`rss` = total resident memory · `heapTotal`/`heapUsed` = V8 heap · `external` = C++ objects bound to JS · `arrayBuffers` = Buffer/ArrayBuffer memory. A leak with flat `heapUsed` but rising `rss` points at Buffers or native code.

### 278. What are WeakMap/WeakRef/FinalizationRegistry for? 🔴
Metadata keyed by objects without preventing collection. `WeakMap` for per-object caches; `WeakRef`/`FinalizationRegistry` for caches of expensive resources — but finalizers are non-deterministic, so never rely on them for cleanup.

```js
const meta = new WeakMap();
meta.set(reqObject, { startedAt: Date.now() }); // no leak when reqObject is GC'd
```

### 279. What are hidden classes and why do they matter? 🔴
V8 assigns each object a shape; consistent property order and types let it use inline caches. Adding properties in different orders or `delete`-ing creates megamorphic sites and deoptimizes.

```js
// ✅ same shape
function P(x, y) { this.x = x; this.y = y; }
// ❌ p.z = 1 later creates a new hidden class
```

### 280. What deoptimizes V8 code? 🔴
Mixed argument types, `arguments` leaking, `try/catch` in old versions (fine now), `delete`, sparse/holey arrays, and changing an array's element kind (SMI → double → object).

### 281. What performance flags help diagnose V8? 🔴
```bash
node --trace-opt --trace-deopt app.js
node --prof app.js && node --prof-process isolate-*.log
node --max-old-space-size=4096 --max-semi-space-size=64 app.js
```

### 282. How do you speed up JSON handling? 🟠
`JSON.stringify` is a top hotspot for APIs. Use schema-based serializers (`fast-json-stringify`, Fastify does it automatically), avoid stringifying huge payloads on the main thread, and stream large arrays.

### 283. What is a realistic Node performance checklist? 🟠
Keep-alive agents · gzip/brotli at the edge · connection pooling · bounded caches with TTL · avoid sync I/O and sync crypto in handlers · offload CPU to workers · cluster/replicas per core · index your queries and avoid N+1 · measure event loop lag · pin a CPU/memory budget per pod.

---

## 11. Security (284–294)

### 284. What are the OWASP Top 10 risks in a Node API? 🟠
Broken access control, injection (SQL/NoSQL/command), insecure design, misconfiguration, vulnerable dependencies, auth failures, data integrity failures (unsafe deserialization), logging gaps, SSRF, and cryptographic failures.

### 285. How do you prevent SQL and NoSQL injection? 🟠
```js
// SQL: parameterized
await db.query('SELECT * FROM users WHERE email = $1', [email]);

// NoSQL: cast and validate — { $gt: '' } bypasses a naive equality check
await User.findOne({ email: String(req.body.email) });
```

### 286. What is prototype pollution? 🔴
Untrusted keys like `__proto__` or `constructor.prototype` merged into an object mutate `Object.prototype` globally, enabling auth bypass or RCE.

```js
function safeMerge(target, src) {
  for (const k of Object.keys(src)) {
    if (k === '__proto__' || k === 'constructor' || k === 'prototype') continue;
    target[k] = src[k];
  }
}
const dict = Object.create(null); // no prototype at all
```
Also: `node --frozen-intrinsics`, and schema validation that strips unknown keys.

### 287. How do you hash passwords? 🔵
```js
const bcrypt = require('bcrypt');
const hash = await bcrypt.hash(password, 12);
const ok = await bcrypt.compare(password, hash);
```
Or `argon2id` (preferred). Never MD5/SHA-1/plain SHA-256; those are too fast.

### 288. How do you compare secrets safely? 🟠
```js
crypto.timingSafeEqual(Buffer.from(a), Buffer.from(b)); // constant time, equal lengths required
```

### 289. How do you generate secure random values? 🔵
```js
crypto.randomUUID();
crypto.randomBytes(32).toString('base64url');
```
`Math.random()` is not cryptographically secure.

### 290. How do you encrypt data at rest in Node? 🟠
```js
const iv = crypto.randomBytes(12);
const c = crypto.createCipheriv('aes-256-gcm', key, iv);
const enc = Buffer.concat([c.update(plain, 'utf8'), c.final()]);
const tag = c.getAuthTag(); // store iv + tag + enc
```
Use authenticated encryption (GCM); never reuse an IV with the same key.

### 291. What does `helmet` do? 🔵
Sets defensive headers: CSP, HSTS, `X-Content-Type-Options: nosniff`, frame options, referrer policy.

```js
app.use(require('helmet')());
app.disable('x-powered-by');
```

### 292. How do you prevent SSRF? 🟠
Allowlist destination hosts, resolve DNS and reject private/link-local ranges (and re-check after redirects), disable redirects, and set timeouts.

### 293. How do you handle secrets in Node apps? 🟠
Environment variables injected by the platform, or a secret manager (Vault/AWS Secrets Manager) with rotation. Never commit `.env`; never log `process.env`; scrub secrets from error reports.

### 294. What is a ReDoS attack? 🟠
Catastrophic backtracking from nested quantifiers on attacker-controlled input freezes the event loop.

```js
/^(a+)+$/.test('aaaaaaaaaaaaaaaaaaaaaaaaaaX'); // exponential
```
Mitigate: avoid nested quantifiers, bound input length, use `re2` or a timeout.

---

## 12. Architecture, Deployment & System Design (295–300)

### 295. How do you structure a large Node/Express codebase? 🟠
Layer by feature, not by file type: `src/modules/orders/{router,controller,service,repository,schema}.js`. Controllers do HTTP only, services hold business logic and are framework-agnostic, repositories own persistence. Dependencies point inward.

```
src/
  modules/orders/{orders.router.js, orders.service.js, orders.repo.js}
  shared/{errors.js, logger.js, db.js}
  app.js  server.js
```
Keep `app.js` (wiring) separate from `server.js` (listening) so tests can import the app without binding a port.

### 296. How do you write a production Dockerfile for Node? 🟠
```dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build && npm prune --omit=dev

FROM node:20-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
USER node
EXPOSE 3000
CMD ["node", "dist/server.js"]
```
Run Node as PID 1's child (or use `--init`) so `SIGTERM` reaches it; don't wrap in `npm start`, which swallows signals.

### 297. How do you do zero-downtime deploys? 🟠
Rolling update + readiness probe + graceful shutdown: on `SIGTERM`, fail readiness first, keep serving in-flight requests for the drain period, `server.close()`, then exit. Combine with backward-compatible DB migrations (expand → migrate → contract).

### 298. How do you design a job/queue system in Node? 🔴
Producers push to Redis (BullMQ) or a broker (SQS/RabbitMQ/Kafka); separate worker processes consume with bounded concurrency. Requirements: at-least-once delivery → idempotent handlers, exponential backoff, a dead-letter queue, visibility timeouts, and metrics on queue depth and age.

```js
const { Queue, Worker } = require('bullmq');
const q = new Queue('emails', { connection });
await q.add('welcome', { userId }, { attempts: 5, backoff: { type: 'exponential', delay: 1000 } });
new Worker('emails', async job => sendEmail(job.data), { connection, concurrency: 10 });
```

### 299. How would you design a real-time notification service handling 100k concurrent connections? 🔴
Terminate WebSockets on horizontally scaled Node pods (WS is cheap in Node — memory per connection is the constraint, so tune `--max-old-space-size` and avoid per-connection closures). Use Redis Pub/Sub or NATS for fan-out between pods so any pod can deliver to any user. Keep a connection registry (`userId → podId`) in Redis with TTL heartbeats. Add per-connection send queues with backpressure and drop policies, batched writes, and a heartbeat to reap zombies. Persist undelivered notifications so reconnecting clients can replay from a cursor.

### 300. When would you *not* choose Node.js? 🔴
Heavy numeric/CPU work (ML training, video transcoding, scientific computing) where Go/Rust/C++ or a GPU pipeline wins; hard-real-time systems where GC pauses are unacceptable; teams with deep JVM/.NET expertise and no JS ecosystem need; and monolithic CPU-bound batch processing. Node's strength is I/O concurrency, a shared language across the stack, fast iteration, and the largest package ecosystem — that's the trade to argue in an interview.

---

## Quick revision cheat sheet

**Execution order:** sync → `process.nextTick` queue → microtasks (promises) → timers → pending → poll → check (`setImmediate`) → close.

**Thread pool (4 by default):** `fs`, `dns.lookup`, `crypto` KDFs, `zlib`. Sockets are *not* on it.

**Never in a request handler:** `*Sync` I/O, unbounded loops, `JSON.parse` on untrusted huge bodies, sync crypto, regex with nested quantifiers.

**Always:** `pipeline` over `pipe`, `allSettled` when partial failure is fine, bounded caches, `AbortSignal` on outbound calls, graceful shutdown, structured logs with a request ID.

**Memory triage:** rising `heapUsed` → JS leak (snapshots) · rising `external`/`arrayBuffers` → Buffer/native leak · rising `rss` only → fragmentation or native module.

*Good luck — walk the interviewer through the trade-off, not just the definition.*
