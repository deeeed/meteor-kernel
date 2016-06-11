Disable changes on Meteor.* functions.
- See https://github.com/DispatchMe/meteor-kernel/issues/5

------------------
kernel
======

Improves performance by runing tasks and reactivity in animation frame directly or throttled.

The kernel helps you run functions in sync using the animation frame. It also allows you to defer or throttle tasks, having them run when theres cpu time for it.

The reason for this is to flatten ui functions to help keep a stable frame rate.

## Api
The first part of the api is a bit more hidden since its actually overwriting Meteor behaviour pr. default regarding:

* `Blaze.autorun`
* `Meteor.setTimeout`
* `Meteor.setInterval`
* `Meteor.clearTimeout`
* `Meteor.clearInterval`
* `Meteor.defer`

`Kernel.autorun` This flattens out spikes a bit when invalidations are triggered. The first run is run at intialization but is otherwise async / defered, so the computation isnt run just af triggering `Tracker.flush()` or an invalidation.

### Three ways to run functions in the animation frame
The api consists of 3 functions with some alias functions. They all return the `Kernel` object so they are chainable.

`Kernel.now` - Returns the high resolution timestamp.

`Kernel.timed` - Run a function at a certain time.
```js
  var runAt = Kernel.now() + 1000; // Run a sec from now
  Kernel.timed(function(runAt, timestamp, lasttimestamp, frame) {
    // Run this function a sec from now
  }, runAt);
```

`Kernel.onRender` alias `Kernel.run` - Run a function in the next animation frame.
```js
  Kernel.run(function(timestamp, lastTimestamp, currentFrameNumber) {
    // Run this function in the next animation frame
  });
```

`Kernel.defer` alias `Kernel.then` - Run a function when theres cpu time for it.
```js
  Kernel.defer(function(timestamp, lastTimestamp, currentFrameNumber) {
    // Run this function when theres time for it in the animation frame
  });
```

`Kernel.each` - Iterate over an array or collection executing a callback when theres cpu time for it.
Heres an example added chained functions
```js
  Kernel
  .then(function() {
    console.log('We are going to iterate over list');
  })
  .each(list, function(value, key) {
    // Iterate over list and only run this function when theres time for
    // it in the animation frame
  })
  .then(function() {
    console.log('We are done iterating over list');
  });
```

`Kernel.frameRateLimit` - Default is `0` but it can be set to `1000/60` to limit the kernel only to run 60 times a sec.

`Kernel.deferedTimeLimit` - Time limit / window for running defered functions.

`Kernel.currentFrame` - Frame counter

`Kernel.maxDeferedLength` - The limit of allowed defered functions in queue, if set to `-1` the limit is disabled. Its default value is `100` functions.
