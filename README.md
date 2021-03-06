# mineflayer-navigate

A library to help your mineflayer bot navigate around the 3D world using
the A* algorithm.

See [https://github.com/superjoe30/mineflayer/](https://github.com/superjoe30/mineflayer/)

[YouTube Demo](http://www.youtube.com/watch?v=O6lQdmRz8eE)

## Usage

```js
var mineflayer = require('mineflayer');
var navigatePlugin = require('mineflayer-navigate')(mineflayer);
var bot = mineflayer.createBot({ username: 'Player' });
// install the plugin
navigatePlugin(bot);
// optional configuration
bot.navigate.blocksToAvoid[132] = true; // avoid tripwire
bot.navigate.blocksToAvoid[59] = false; // ok to trample crops
bot.navigate.on('pathFound', function (path) {
  bot.chat("found path. I can get there in " + path.length + " moves.");
});
bot.navigate.on('cannotFind', function (closestPoint) {
  bot.chat("unable to find path. getting as close as possible");
  bot.navigate.to(closestPoint);
});
bot.navigate.on('arrived', function () {
  bot.chat("I have arrived");
});
bot.navigate.on('interrupted', function() {
  bot.chat("stopping");
});
bot.on('chat', function(username, message) {
  // navigate to whoever talks
  if (username === bot.username) return;
  var target = bot.players[username].entity;
  if (message === 'come') {
    bot.navigate.to(target.position);
  } else if (message === 'stop') {
    bot.navigate.stop();
  }
});
```

## Documentation

### bot.navigate.to(point, options)

Finds a path to the specified location and goes there.

 * `point` - the block you want your feet to be standing on
 * `options` - See `bot.navigate.findPathSync`

#### event "pathPartFound" (path)

Emitted from `bot.navigate` when a partial path is found. `path` is an array
of nodes.

#### event "pathFound" (path)

Emitted from `bot.navigate` when a complete path is found. `path` is an array
of nodes.

#### event "cannotFind" (closestPoint)

Emitted when a path cannot be found.

 * `closestPoint` - a `vec3` instance - the closest point that you *could*
   navigate to.

#### event "arrived"

Emitted when the destination is reached.

#### event "stop"

Emitted when navigation has been aborted.


### bot.navigate.stop()

Aborts an in progress navigation job.

### bot.navigate.findPathSync(end, [options])

Finds a path to `end`. Can be used to see if it is possible to navigate to a
particular point.

Returns an object that looks like:

```js
{
  status: 'success', // one of ['success', 'noPath', 'timeout', 'tooFar']
  path: [startPoint, point1, point2, ..., endPoint],
}
```

The value of `status` has several meanings:

 * `success` - `path` is an array of points that can be passed to `walk()`.
 * `noPath` - there is no path to `end`. Try a larger `endRadius`. `path`
   is the path to the closest reachable point to end.
 * `timeout` - no path could be found in the allotted time. Try a larger
   `endRadius` or `timeout`. `path` is the path to the closest reachable
    point to end that could be found in the allotted time.
 * `tooFar` - `end` is too far away, so `path` contains the path to walk 100
   meters in the general direction of `end`.

Parameters:

 * `end` - the block you want your feet to be standing on
 * `options` - optional parameters which come with sensible defaults
   - `isEnd` - function(node) - passed on to the A* library. `node.point` is
     a vec3 instance.
   - `endRadius` - used for default `isEnd`. Effectively defaults to 0.
   - `timeout` - passed on to the A* library. Default 10 seconds.
   - `tooFarThreshold` - if `point` is greater than `tooFarThreshold`, this
     function will search instead for a path to walk 100 meters in the general
     direction of end.

### bot.navigate.walk(path, [callback])

*Note: does not emit events*

Walks the bot along the path and calls the callback function when it has
arrived.

 * `path` - array of points to be navigated.
 * `callback()` - (optional) - called when the bot has arrived.

## History

### 0.0.5

 * recommended API is now callback based
 * add `bot.navigate.findPathSync(end, [options])`
 * add `bot.navigate.walk(path, [callblack])`

### 0.0.4

 * add 'interrupted' event

### 0.0.3

 * fix bot looking at its feet while walking
 * possible speed improvement by using native array methods
 * `cannotFind` event now has `closestPoint` parameter, the closest point it
   *could* get to
 * `bot.navigate.blocksToAvoid` is a map of block id to boolean value which
   tells whether to avoid the block. comes with sensible defaults like
   avoiding fire and crops.

### 0.0.2

 * fix pathfinding very far away
