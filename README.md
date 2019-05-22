# node-migrate

Migrations simplified.

## Installation

```
npm install node-migrate
```

## Usage

### Command line

```
Usage: node-migrate [options] [command]

Options:
  -d, --dir         Specify migrations directory - default: migrations
  -h, --help        Print usage
  -s, --store       Specify state storage
  -v, --version     Print version
```

### Programatic use

```js
const EventEmitter = require('events');
const migrate = require('node-migrate');
const fsStore = require('node-migrate-fs');

const emitter = new EventEmitter()
  .on('start', file => {})
  .on('finish', file => {})
  .on('error', (err, file) => {});

migrate({
  dir: 'migrations/db',
  store: fsStore,
  emitter
})
  .then(() => {})
  .catch(err => {});
```

## Migrations

Just create file in your migration directory (default: `migrations`) and export a function.

```js
// add-countries.js

module.exports = async () => {
  // migration code
};
```

Recommended way is to prefix your migration file names with timestamp e.g. `1558514504-add-countries.js` as migration order is alphabetical.

## Stores

To save the state of the migrations which have been run `node-migrate` uses stores. You can use existing stores

- [node-migrate-fs](https://github.com/ct0r/node-migrate-fs)
- [node-migrate-mongo](https://github.com/ct0r/node-migrate-mongo)

or implement your own.

### Simple store

Export a function which returns object with `get()` and `set(state)` functions and in `get` simply returns what goes to `set`.

```js
module.exports = () => ({
  get() {
    // ...
  },
  set(state) {
    // ...
  }
});
```

### Store with cli support

To create reusable store configurable via cli you need to export `cli` object with `spec` (same like in [arg](https://github.com/zeit/arg)) and `getOptions` which maps arguments to options.

```js
module.exports = ({ path }) => ({
  get() {
    // ...
  },
  set() {
    // ...
  }
});

module.exports.cli = {
  spec: {
    '--store-path': String
  },
  getOptions: args => ({
    path: args['--store-path']
  })
};
```

then you can use it via cli

```
node-migrate --store path/to/my/store --store-path my/path
```

## Examples

### Basic

```json
{
  "scripts": {
    "migrate": "node-migrate --store node-migrate-fs"
  }
}
```

```
npm run migrate
```

### Multiple data sources

```json
{
  "scripts": {
    "migrate": "migrate-fs && migrate-db",
    "migrate-fs": "node-migrate --dir migrations/fs --store node-migrate-fs --store-path migrations/fs/.state",
    "migrate-db": "node-migrate --dir migrations/db --store node-migrate-mongo --store-url mongodb://my-server/my-database"
  }
}
```

```
npm run migrate
```
