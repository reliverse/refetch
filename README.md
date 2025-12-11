# 🧬 refetch

> @reliverse/refetch is your high-performance fetch API for modern CLIs and web apps. Crafted for devs who care about precision, speed, and zero-bloat DX.

[📦 npm](https://npmjs.com/package/@reliverse/refetch) • [✨ github](https://github.com/blefnk/refetch) • [💬 discord](https://discord.gg/pb8ukbwpsj)

## what is this?

A better fetch API. Works on node, browser, and workers.

## 🚀 Quick Start

Install:

```bash
# npm
npm i ofetch

# yarn
yarn add ofetch
```

Import:

```js
// ESM / Typescript
import { ofetch } from "ofetch";

// CommonJS
const { ofetch } = require("ofetch");
```

## ✔️ Works with Node.js

We use [conditional exports](https://nodejs.org/api/packages.html#packages_conditional_exports) to detect Node.js
and automatically use [unjs/node-fetch-native](https://github.com/unjs/node-fetch-native). If `globalThis.fetch` is available, will be used instead. To leverage Node.js 17.5.0 experimental native fetch API use [`--experimental-fetch` flag](https://nodejs.org/dist/latest-v17.x/docs/api/cli.html#--experimental-fetch).

## ✔️ Parsing Response

`ofetch` will smartly parse JSON and native values using [destr](https://github.com/unjs/destr), falling back to the text if it fails to parse.

```js
const { users } = await ofetch("/api/users");
```

For binary content types, `ofetch` will instead return a `Blob` object.

You can optionally provide a different parser than `destr`, or specify `blob`, `arrayBuffer`, or `text` to force parsing the body with the respective `FetchResponse` method.

```js
// Use JSON.parse
await ofetch("/movie?lang=en", { parseResponse: JSON.parse });

// Return text as is
await ofetch("/movie?lang=en", { parseResponse: (txt) => txt });

// Get the blob version of the response
await ofetch("/api/generate-image", { responseType: "blob" });
```

## ✔️ JSON Body

If an object or a class with a `.toJSON()` method is passed to the `body` option, `ofetch` automatically stringifies it.

`ofetch` utilizes `JSON.stringify()` to convert the passed object. Classes without a `.toJSON()` method have to be converted into a string value in advance before being passed to the `body` option.

For `PUT`, `PATCH`, and `POST` request methods, when a string or object body is set, `ofetch` adds the default `content-type: "application/json"` and `accept: "application/json"` headers (which you can always override).

Additionally, `ofetch` supports binary responses with `Buffer`, `ReadableStream`, `Stream`, and [compatible body types](https://developer.mozilla.org/en-US/docs/Web/API/fetch#body). `ofetch` will automatically set the `duplex: "half"` option for streaming support!

**Example:**

```js
const { users } = await ofetch("/api/users", {
  method: "POST",
  body: { some: "json" },
});
```

## ✔️ Handling Errors

`ofetch` Automatically throws errors when `response.ok` is `false` with a friendly error message and compact stack (hiding internals).

A parsed error body is available with `error.data`. You may also use `FetchError` type.

```ts
await ofetch("https://google.com/404");
// FetchError: [GET] "https://google/404": 404 Not Found
//     at async main (/project/playground.ts:4:3)
```

To catch error response:

```ts
await ofetch("/url").catch((error) => error.data);
```

To bypass status error catching you can set `ignoreResponseError` option:

```ts
await ofetch("/url", { ignoreResponseError: true });
```

## ✔️ Auto Retry

`ofetch` Automatically retries the request if an error happens and if the response status code is included in `retryStatusCodes` list:

**Retry status codes:**

- `408` - Request Timeout
- `409` - Conflict
- `425` - Too Early ([Experimental](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Early-Data))
- `429` - Too Many Requests
- `500` - Internal Server Error
- `502` - Bad Gateway
- `503` - Service Unavailable
- `504` - Gateway Timeout

You can specify the amount of retry and delay between them using `retry` and `retryDelay` options and also pass a custom array of codes using `retryStatusCodes` option.

The default for `retry` is `1` retry, except for `POST`, `PUT`, `PATCH`, and `DELETE` methods where `ofetch` does not retry by default to avoid introducing side effects. If you set a custom value for `retry` it will **always retry** for all requests.

The default for `retryDelay` is `0` ms.

```ts
await ofetch("http://google.com/404", {
  retry: 3,
  retryDelay: 500, // ms
  retryStatusCodes: [ 404, 500 ], // response status codes to retry
});
```

## ✔️ Timeout

You can specify `timeout` in milliseconds to automatically abort a request after a timeout (default is disabled).

```ts
await ofetch("http://google.com/404", {
  timeout: 3000, // Timeout after 3 seconds
});
```

## ✔️ Type Friendly

The response can be type assisted:

```ts
const article = await ofetch<Article>(`/api/article/${id}`);
// Auto complete working with article.id
```

## ✔️ Adding `baseURL`

By using `baseURL` option, `ofetch` prepends it for trailing/leading slashes and query search params for baseURL using [ufo](https://github.com/unjs/ufo):

```js
await ofetch("/config", { baseURL });
```

## ✔️ Adding Query Search Params

By using `query` option (or `params` as alias), `ofetch` adds query search params to the URL by preserving the query in the request itself using [ufo](https://github.com/unjs/ufo):

```js
await ofetch("/movie?lang=en", { query: { id: 123 } });
```

## ✔️ Interceptors

Providing async interceptors to hook into lifecycle events of `ofetch` call is possible.

You might want to use `ofetch.create` to set shared interceptors.

### `onRequest({ request, options })`

`onRequest` is called as soon as `ofetch` is called, allowing you to modify options or do simple logging.

```js
await ofetch("/api", {
  async onRequest({ request, options }) {
    // Log request
    console.log("[fetch request]", request, options);

    // Add `?t=1640125211170` to query search params
    options.query = options.query || {};
    options.query.t = new Date();
  },
});
```

### `onRequestError({ request, options, error })`

`onRequestError` will be called when the fetch request fails.

```js
await ofetch("/api", {
  async onRequestError({ request, options, error }) {
    // Log error
    console.log("[fetch request error]", request, error);
  },
});
```

### `onResponse({ request, options, response })`

`onResponse` will be called after `fetch` call and parsing body.

```js
await ofetch("/api", {
  async onResponse({ request, response, options }) {
    // Log response
    console.log("[fetch response]", request, response.status, response.body);
  },
});
```

### `onResponseError({ request, options, response })`

`onResponseError` is the same as `onResponse` but will be called when fetch happens but `response.ok` is not `true`.

```js
await ofetch("/api", {
  async onResponseError({ request, response, options }) {
    // Log error
    console.log(
      "[fetch response error]",
      request,
      response.status,
      response.body
    );
  },
});
```

### Passing array of interceptors

If necessary, it's also possible to pass an array of function that will be called sequentially.

```js
await ofetch("/api", {
  onRequest: [
    () => {
      /* Do something */
    },
    () => {
      /* Do something else */
    },
  ],
});
```

## ✔️ Create fetch with default options

This utility is useful if you need to use common options across several fetch calls.

**Note:** Defaults will be cloned at one level and inherited. Be careful about nested options like `headers`.

```js
const apiFetch = ofetch.create({ baseURL: "/api" });

apiFetch("/test"); // Same as ofetch('/test', { baseURL: '/api' })
```

## 💡 Adding headers

By using `headers` option, `ofetch` adds extra headers in addition to the request default headers:

```js
await ofetch("/movies", {
  headers: {
    Accept: "application/json",
    "Cache-Control": "no-cache",
  },
});
```

## 🍣 Access to Raw Response

If you need to access raw response (for headers, etc), you can use `ofetch.raw`:

```js
const response = await ofetch.raw("/sushi");

// response._data
// response.headers
// ...
```

## 🌿 Using Native Fetch

As a shortcut, you can use `ofetch.native` that provides native `fetch` API

```js
const json = await ofetch.native("/sushi").then((r) => r.json());
```

## 🕵️ Adding HTTP(S) Agent

In Node.js (>= 18) environments, you can provide a custom dispatcher to intercept requests and support features such as Proxy and self-signed certificates. This feature is enabled by [undici](https://undici.nodejs.org/) built-in Node.js. [read more](https://undici.nodejs.org/#/docs/api/Dispatcher) about the Dispatcher API.

Some available agents:

- `ProxyAgent`: A Proxy Agent class that implements the Agent API. It allows the connection through a proxy in a simple way. ([docs](https://undici.nodejs.org/#/docs/api/ProxyAgent))
- `MockAgent`: A mocked Agent class that implements the Agent API. It allows one to intercept HTTP requests made through undici and return mocked responses instead. ([docs](https://undici.nodejs.org/#/docs/api/MockAgent))
- `Agent`: Agent allows dispatching requests against multiple different origins. ([docs](https://undici.nodejs.org/#/docs/api/Agent))

**Example:** Set a proxy agent for one request:

```ts
import { ProxyAgent } from "undici";
import { ofetch } from "ofetch";

const proxyAgent = new ProxyAgent("http://localhost:3128");
const data = await ofetch("https://icanhazip.com", { dispatcher: proxyAgent });
```

**Example:** Create a custom fetch instance that has proxy enabled:

```ts
import { ProxyAgent, setGlobalDispatcher } from "undici";
import { ofetch } from "ofetch";

const proxyAgent = new ProxyAgent("http://localhost:3128");
const fetchWithProxy = ofetch.create({ dispatcher: proxyAgent });

const data = await fetchWithProxy("https://icanhazip.com");
```

**Example:** Set a proxy agent for all requests:

```ts
import { ProxyAgent, setGlobalDispatcher } from "undici";
import { ofetch } from "ofetch";

const proxyAgent = new ProxyAgent("http://localhost:3128");
setGlobalDispatcher(proxyAgent);

const data = await ofetch("https://icanhazip.com");
```

**Example:** Allow self-signed certificates (USE AT YOUR OWN RISK!)

```ts
import { ProxyAgent } from "undici";
import { ofetch } from "ofetch";

// Note: This makes fetch unsecure against MITM attacks. USE AT YOUR OWN RISK!
const unsecureProxyAgent = new ProxyAgent({ requestTls: { rejectUnauthorized: false } });
const unsecureFetch = ofetch.create({ dispatcher: unsecureProxyAgent });

const data = await unsecureFetch("https://www.squid-cache.org/");
```

On older Node.js version (<18), you might also use use `agent`:

```ts
import { HttpsProxyAgent } from "https-proxy-agent";

await ofetch("/api", {
  agent: new HttpsProxyAgent("http://example.com"),
});
```

### `keepAlive` support (only works for Node < 18)

By setting the `FETCH_KEEP_ALIVE` environment variable to `true`, an HTTP/HTTPS agent will be registered that keeps sockets around even when there are no outstanding requests, so they can be used for future requests without having to re-establish a TCP connection.

**Note:** This option can potentially introduce memory leaks. Please check [node-fetch/node-fetch#1325](https://github.com/node-fetch/node-fetch/pull/1325).

## 📦 Bundler Notes

- All targets are exported with Module and CommonJS format and named exports
- No export is transpiled for the sake of modern syntax
  - You probably need to transpile `ofetch`, `destr`, and `ufo` packages with Babel for ES5 support
- You need to polyfill `fetch` global for supporting legacy browsers like using [unfetch](https://github.com/developit/unfetch)

## ❓ FAQ

**Why export is called `ofetch` instead of `fetch`?**

Using the same name of `fetch` can be confusing since API is different but still, it is a fetch so using the closest possible alternative. You can, however, import `{ fetch }` from `ofetch` which is auto-polyfill for Node.js and using native otherwise.

**Why not have default export?**

Default exports are always risky to be mixed with CommonJS exports.

This also guarantees we can introduce more utils without breaking the package and also encourage using `ofetch` name.

**Why not transpiled?**

By transpiling libraries, we push the web backward with legacy code which is unneeded for most of the users.

If you need to support legacy users, you can optionally transpile the library in your build pipeline.

## refetch's helper package



A redistribution of [node-fetch v3](https://github.com/node-fetch/node-fetch) (+ more!) for better backward and forward compatibility.

**Why this package?**

- We can no longer `require('node-fetch')` with the latest version. This stopped popular libraries from upgrading and dependency conflicts between `node-fetch@2` and `node-fetch@3`.
- With upcoming versions of Node.js, native `fetch` is being supported. We are prepared for native fetch support using this package yet keep supporting older Node versions.
- With the introduction of native fetch to Node.js via [undici](https://github.com/nodejs/undici) there is no easy way to support http proxies!

**Features:**

✅ Prefer to **native globals** when available (See Node.js [experimental fetch](https://nodejs.org/dist/latest-v17.x/docs/api/cli.html#--experimental-fetch)).

✅ Compact build and less install size with **zero dependencies** [![][packagephobia-s-src]][packagephobia-s-href] <sup>vs</sup> [![][packagephobia-s-alt-src]][packagephobia-s-alt-href]

✅ Support both **CommonJS** (`require`) and **ESM** (`import`) usage

✅ Use native version if imported without `node` condition using [conditional exports](https://nodejs.org/api/packages.html#packages_conditional_exports) with **zero bundle overhead**

✅ Polyfill support for Node.js

✅ Compact and simple proxy supporting both Node.js versions without native fetch using [HTTP Agent](https://github.com/TooTallNate/proxy-agents/tree/main/packages/proxy-agent) and versions with native fetch using [Undici Proxy Agent](https://undici.nodejs.org/#/docs/api/ProxyAgent)

## Usage

Install `node-fetch-native` dependency:

```sh
# npm
npm i node-fetch-native

# yarn
yarn add node-fetch-native

# pnpm
pnpm i node-fetch-native
```

You can now either import or require the dependency:

```js
// ESM
import fetch from "node-fetch-native";

// CommonJS
const fetch = require("node-fetch-native");
```

More named exports:

```js
// ESM
import {
  fetch,
  Blob,
  FormData,
  Headers,
  Request,
  Response,
  AbortController,
} from "node-fetch-native";

// CommonJS
const {
  fetch,
  Blob,
  FormData,
  Headers,
  Request,
  Response,
  AbortController,
} = require("node-fetch-native");
```

## Force using non-native version

Sometimes you want to explicitly use none native (`node-fetch`) implementation of `fetch` in case of issues with the native/polyfill version of `globalThis.fetch` with Node.js or runtime environment.

You have two ways to do this:

- Set the `FORCE_NODE_FETCH` environment variable before starting the application.
- Import from `node-fetch-native/node`

## Disable runtime check

Once the `node-fetch-native/node` module is loaded, it pushes a log warning if the current runtime differs from the Node.js. Set the `DISABLE_NODE_FETCH_NATIVE_WARN` environment variable to turn this check off.

## Polyfill support

Using the polyfill method, we can ensure global fetch is available in the environment and all files. Natives are always preferred.

**Note:** I don't recommend this if you are authoring a library! Please prefer the explicit methods.

```js
// ESM
import "node-fetch-native/polyfill";

// CJS
require("node-fetch-native/polyfill");

// You can now use fetch() without any import!
```

## Proxy Support

Node.js has no built-in support for HTTP Proxies for fetch (see [nodejs/undici#1650](https://github.com/nodejs/undici/issues/1650) and [nodejs/node#8381](https://github.com/nodejs/node/issues/8381))

This package bundles a compact and simple proxy-supported solution for both Node.js versions without native fetch using [HTTP Agent](https://github.com/TooTallNate/proxy-agents/tree/main/packages/proxy-agent) and versions with native fetch using [Undici Proxy Agent](https://undici.nodejs.org/#/docs/api/ProxyAgent).

By default, `https_proxy`, `http_proxy`, `HTTPS_PROXY`, and `HTTP_PROXY` environment variables will be checked and used (in order) for the proxy and if not any of them are set, the proxy will be disabled. You can override it using the `url` option passed to `createFetch` and `createProxy` utils.

By default, `no_proxy` and `NO_PROXY` environment variables will be checked and used for the (comma-separated) list of hosts to ignore the proxy for. You can override it using the `noProxy` option passed to `createFetch` and `createProxy` utils. The entries starting with a dot will be used to check the domain and also any subdomain.

> [!NOTE]
> Using export conditions, this utility adds proxy support for Node.js and for other runtimes, it will simply return native fetch.

### `fetch` with proxy support

You can simply import `{ fetch }` from `node-fetch-native/proxy` with a preconfigured `fetch` function that has proxy support.

```ts
import { fetch } from "node-fetch-native/proxy";

console.log(await fetch("https://icanhazip.com").then((r) => r.text()));
```

### `createFetch` utility

You can use the `createFetch` utility to instantiate a `fetch` instance with custom proxy options.

```ts
import { createFetch } from "node-fetch-native/proxy";

const fetch = createFetch({ url: "http://localhost:9080" });

console.log(await fetch("https://icanhazip.com").then((r) => r.text()));
```

### `createProxy` utility

`createProxy` returns an object with `agent` and `dispatcher` keys that can be passed as fetch options.

```ts
import { fetch } from "node-fetch-native";
import { createProxy } from "node-fetch-native/proxy";

const proxy = createProxy();
// const proxy = createProxy({ url: "http://localhost:8080" });

console.log(
  await fetch("https://icanhazip.com", { ...proxy }).then((r) => r.text()),
);
```

## Alias to `node-fetch`

Using this method, you can ensure all project dependencies and usages of `node-fetch` can benefit from improved `node-fetch-native` and won't conflict between `node-fetch@2` and `node-fetch@3`.

### npm

Using npm [overrides](https://docs.npmjs.com/cli/v8/configuring-npm/package-json#overrides):

```jsonc
// package.json
{
  "overrides": {
    "node-fetch": "npm:node-fetch-native@latest",
  },
}
```

### yarn

Using yarn [selective dependency resolutions](https://classic.yarnpkg.com/lang/en/docs/selective-version-resolutions/):

```jsonc
// package.json
{
  "resolutions": {
    "node-fetch": "npm:node-fetch-native@latest",
  },
}
```

### pnpm

Using [pnpm.overrides](https://pnpm.io/package_json#pnpmoverrides):

```jsonc
// package.json
{
  "pnpm": {
    "overrides": {
      "node-fetch": "npm:node-fetch-native@latest",
    },
  },
}
```

## license

mit © [blefnk nazar kornienko](https://github.com/blefnk)  
part of the [reliverse](https://github.com/reliverse) ecosystem.

*p.s. this readme was generated with [@reliverse/redocs](https://github.com/reliverse/redocs)*
