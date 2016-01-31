---
layout: layout
---

# Advanced Usage

These methods are primarily for internal use, but can be handy in some
unusual situations.  If you find yourself using them frequently, you
*may* be Doing It Wrong.  However, if you find them useful, you should
feel perfectly comfortable using them.

Please [let us know](https://github.com/isaacs/node-tap/issues) if you
frequently encounter situations requiring advanced usage, because this
may indicate a shortcoming in the "non-advanced" [API](/api/).

## t.stdin()

Parse standard input as if it was a child test named `/dev/stdin`.

Returns a Promise which resolves with the parent when the input stream
is completed.

This is primarily for use in the test runner, so that you can do
`some-tap-emitting-program | tap other-file.js - -Rnyan`.

## t.spawn(command, arguments, [options], [name], [extra])

Sometimes, instead of running a child test directly inline, you might
want to run a TAP producting test as a child process, and treat its
standard output as the TAP stream.

Returns a Promise which resolves with the parent when the child
process is completed.

That's what this method does.

It is primarily used by the executable runner, to run all of the
filename arguments provided on the command line.

The `options` object is passed to `child_process.spawn`, and can
contain stuff like stdio directives and environment vars.

The `extra` arg is the same that would be passed to any assertion or
child test, with the addition of the following fields:

* `bail`: Set to `true` to bail out on the first failure.  This is
  done by checking the output and then forcibly killing the process,
  but also sets the `TAP_BAIL` environment variable, which node-tap
  uses to set this field internally as well.
* `timeout`: The number of ms to allow the child process to continue.
  If it goes beyond this time, the child process will be forcibly
  killed.

## t.addAssert(name, length, fn)

This is used for creating assertion methods on the `Test` class.

It's a little bit advanced, but it's also super handy sometimes.  All
of the assert methods below are created using this function, and it
can be used to create application-specific assertions in your tests.

The name is the method name that will be created.  The length is the
number of arguments the assertion operates on.  (The `message` and
`extra` arguments will alwasy be appended to the end.)

For example, you could have a file at `test/setup.js` that does the
following:

```javascript
var tap = require('tap')

// convenience
if (module === require.main) {
  tap.pass('ok')
  return
}

// Add an assertion that a string is in Title Case
// It takes one argument (the string to be tested)
tap.Test.prototype.addAssert('titleCase', 1, function (str, message, extra) {
  message = message || 'should be in Title Case'
  // the string in Title Case
  // A fancier implementation would avoid capitalizing little words
  // to get `Silence of the Lambs` instead of `Silence Of The Lambs`
  // But whatever, it's just an example.
  var tc = str.toLowerCase().replace(/\b./, function (match) {
    return match.toUpperCase()
  })

  // should always return another assert call, or
  // this.pass(message) or this.fail(message, extra)
  return this.equal(str, tc, message, extra)
})
```

Then in your individual tests, you'd do this:

```javascript
require('./setup.js') // adds the assert
var tap = require('tap')
tap.titleCase('This Passes')
tap.titleCase('however, tHis tOTaLLy faILS')
```

## t.endAll()

Call the `end()` method on all child tests, and then on this one.

## t.current()

Return the currently active test.