<p align="center">
  <img src="https://i.imgur.com/o0u6dto.png" width="300" height="300" alt="stockroom">
  <br>
  <a href="https://www.npmjs.org/package/stockroom"><img src="https://img.shields.io/npm/v/stockroom.svg?style=flat" alt="npm"></a> <a href="https://travis-ci.org/developit/stockroom"><img src="https://travis-ci.org/developit/stockroom.svg?branch=master" alt="travis"></a>
</p>

# Stockroom

> Offload your store management to a worker.

Stockroom seamlessly runs a [Unistore] store (and its actions) in a Web Worker, setting up optimized bidirectional sync so you can also use & subscribe to it on the main thread.

- **Easy** same API as [unistore] - a simple add-on
- **Opt-in** centralized actions with the option of running on the main thread
- **Convenient** action selector shorthand - no action creator needed for simple actions
- **Gracefully degrades** - feature-detect Worker support and fall back to `stockroom/inline`


## Table of Contents

- [Install](#install)
- [Usage](#usage)
- [API](#api)
- [License](#license)


## Install

Stockroom requires that you install [unistore](https://github.com/developit/unistore) (300b) as a peer dependency.

```sh
npm install --save unistore stockroom
```


## Usage

We'll have two files: `index.js` and `worker.js`.  The first is what we import from our app, so it runs on the main thread - it imports our worker (using [worker-loader] or [workerize-loader]) and passes it to Stockroom to create a store instance around it.

**index.js**:

```js
import createStore from 'stockroom'
import StoreWorker from 'worker-loader!./worker'

let store = createStore(new StoreWorker())

let increment = store.action('increment')
store.subscribe(console.log)

// Let's run a registered "increment" action in the worker.
// This will eventually log a state update to the console - `{ count: 1 }`
increment()
```


The second file is our worker code, which runs in the background thread. Here we import Stockroom's worker-side "other half", `stockroom/worker`.  This function returns a store instance just like `createStore()` does in [Unistore], but sets things up to synchronize with the main/parent thread.  It also adds a `registerActions` method to the store, which you can use to define globally-available actions for that store.  These actions can be triggered from the main thread by invoking `store.action('theActionName')` and calling the funtion it returns.

**worker.js**:

```js
import createStore from 'stockroom/worker'

let store = createStore({
  count: 0
})

store.registerActions( store => ({
  increment: ({ count }) => ({ count: count+1 })
}) )

export default store  // if you wish to use `stockroom/inline`
```


### API

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

#### module:stockroom

The main stockroom module, which runs on the main thread.

##### createStore

Given a Web Worker instance, sets up RPC-based synchronization with a WorkerStore running within it.

**Parameters**

- `worker` **[Worker](https://developer.mozilla.org/docs/Web/JavaScript)** An instantiated Web Worker (eg: `new Worker('./store.worker.js')`)

**Examples**

```javascript
import createStore from 'stockroom'
import StoreWorker from 'worker-loader!./store.worker'
let store = createStore(new StoreWorker)
```

Returns **Store** synchronizedStore - a mock unistore store instance sitting in front of the worker store.

#### module:stockroom/inline

Used to run your whole store on the main thread.
Useful non-worker environments or as a fallback.

##### createInlineStore

For SSR/prerendering, pass your exported worker store through this enhancer
 to make an inline synchronous version that runs in the same thread.

**Parameters**

- `workerStore` **WorkerStore** The exported `store` instance that would have been invoked in a Worker

**Examples**

```javascript
let store
if (SUPPORTS_WEB_WORKERS) {
	let createStore = require('stockroom/inline')
	store = createStore(require('./store.worker'))
}
else {
	let createStore = require('stockroom')
	let StoreWorker = require('worker-loader!./store.worker')
	store = createStore(new StoreWorker())
}
export default store
```

Returns **Store** inlineStore - a unistore instance with centralized actions

#### module:stockroom/worker

The other half of stockroom, which runs inside a Web Worker.

##### createWorkerStore

Creates a unistore instance for use in a Web Worker that synchronizes itself to the main thread.

**Parameters**

- `initialState` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** Initial state to populate (optional, default `{}`)

**Examples**

```javascript
import createWorkerStore from 'stockroom/worker'
let initialState = { count: 0 }
let store = createWorkerStore(initialState)
store.registerActions({
	increment(state) {
		return { count: state.count + 1 }
	}
})
```

Returns **WorkerStore** workerStore (enhanced unistore store)


### License

[MIT License](https://oss.ninja/mit/developit) © [Jason Miller](https://jasonformat.com/)


[unistore]: https://github.com/developit/unistore
[preact]: https://github.com/developit/preact
[worker-loader]: https://github.com/webpack-contrib/worker-loader
[workerize-loader]: https://github.com/developit/workerize-loader