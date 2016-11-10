# rethinkdb-changefeed-reconnect

[![Code Climate GPA]()
[![Code Climate Issue Count]()
[![Test Coverage]()

Simple helper module for having [RethinkDB][rethinkdb] changefeed listeners automatically reconnect if they lose their connection (i.e., if the database server goes down or restarts).

Right now, there is a single (default) function exported: `processChangeFeedWithAutoReconnect`.


## Getting Started

1. Install the module in your project

        $ npm install --save rethinkdb-changefeed-reconnect


## How to Use

Basically, you need to implement 3 functions and configure your options:

1. `getFeed()`: this should return a RethinkDB `.changes()` query Promise
2. `onFeedItem(item)`: this is the function that processes each new data item
3. `onError(err)`: this function is where errors will be sent

The available options are:

- `changefeedName`: used for logging purposes so you can differentiate your different feeds
- `attemptDelay`: how long (in milliseconds) to wait between reconnect attempts
- `maxAttempts`: how many reconnect attempts to try before giving up
- `silent`: whether or not you want the logger to be silenced (default = false)
- `logger`: must be an object with 4 functions: `log`, `info`, `warn`, and `error` (default = global.console)

### Example

```javascript
import rethinkdbdash from 'rethinkdbdash'
import processChangefeed from 'rethinkdb-changefeed-reconnect'

const r = rethinkdbdash({servers: {host: 'localhost', port: 28015, db: 'todo'}, silent: true})

processChangefeed(newTasksFeed, handleNewTask, handleError, {
  changefeedName: 'my changefeed',
  attemptDelay: 3000,
  maxAttempts: 3,
  silent: false,
  logger: global.console,
})

function newTasksFeed() {
  r.table('tasks')
    .changes()
    .filter(r.row('old_val').eq(null))
}

function handleNewTask(task) {
  console.log({task})
}

function handleError(err) {
  console.error(err.stack)
}
```

See also [the provided example](/example/index.babel.js).

You can run it like so: `node example`

Then, after you run it, stop your RethinkDB server and watch what gets logged (assuming you passed `silent: false` in your options).


## Notes

This module was developed on top of [rethinkdbdash][rethinkdbdash].


## Contributing

Read the [instructions for contributing](./.github/CONTRIBUTING.md).

1. Clone the repository.

2. Run the setup tasks:

        $ npm install
        $ npm test


## License

See the [LICENSE](./LICENSE) file.


[rethinkdb]: https://github.com/rethinkdb/rethinkdb
[rethinkdbdash]: https://github.com/neumino/rethinkdbdash
