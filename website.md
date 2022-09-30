[nodeconf 2022](https://www.nodeconf.eu/workshops/integrate-external-hardware-into-your-website) -  Web Bluetooth Bookmarklets
==============================================================================

We could develop a complete webpage (feel free!) and there's even a handy library for Web Bluetooth comms with Espruino - and a tutorial at http://www.espruino.com/Web+Bluetooth

However most of you may already have a website you're interested in adding some kind of code to.

Here we're going to write a bookmarklet to pull out some data we're interested in from a website, and to then send it via Web Bluetooth to an external device.

While this isn't suitable for production, it's a great way to prototype thing - or perhaps you just want to use an old Android phone to show build status in a more tangible way.

## What you need

* Chrome/Chromium web browser
* An HTTPS website (so Web Bluetooth can work)


## Let's get some data

First, let's have a go at getting some data from a website. The general idea is you find a website that automatically updates itself, and then query the DOM every so often to scrape the relevant data.

The general process is:

* Go to your website (it needs to be served with HTTPS so we can use Web Bluetooth)
* Find the item you're interested in and click `inspect`
* Check for useful looking classes or IDs in the element (or parent/child elements) that might flag the element as what you want
* Try various `document.querySelector("...")` calls in the dev console until you can extract the data you need
* Call this in `setInterval` and check for any changes, then use those to send updates to the device

I'll give three examples:

### YouTube

While we can get the view count direct off YouTube using this method, YouTube doesn't update it very often so we'll use a third party site.


This is a simple one, but it's not ideal as the view count doesn't auto-update very often. Let's try and get the view count:

* Go to https://livecounts.io/youtube-live-view-counter and enter our video URL: https://www.youtube.com/watch?v=dQw4w9WgXcQ
* You should get this website: https://livecounts.io/youtube-live-view-counter/dQw4w9WgXcQ
* Right-click on the view counter and click `inspect`
* The devtools should pop up and you'll see something like `<span class="odometer-digit">...`, And outside that you'll see `<div class="odometer...`

So all we need to do now is check the text inside the odometer - let's try and get the value programmatically:

* In the Devtools, go to the `Console`
* Type `document.querySelector(".odometer")` and you should see the element got selected and written to the console
* Now try `document.querySelector(".odometer").innerText` and you should get something like `'1\n,\n2\n8\n5\n,\n5\n6\n1\n,\n5\n3\n8'`
* Now, we'll just filter out anything that's not numeric: `document.querySelector(".odometer").innerText.replace(/[^0-9]/g,"")` and we get a number like `1285563306`

When the view count changes, the number does a quick animation, and during that period it's hard to extract the correct number. So in this case we
can check for the `odometer-animating` class and ignore if it is set with `document.querySelector(".odometer").classList.contains("odometer-animating")`

```
var lastViewCount;

function changed(value) {
  console.log("Changed!", value);
}

function poll() {
  if (document.querySelector(".odometer").classList.contains("odometer-animating")) return;
  var count = document.querySelector(".odometer").innerText.replace(/[^0-9]/g,"");
  if (!lastViewCount || count!=lastViewCount) {
    lastViewCount = count;
    changed(count);
  }    
}
```

If you run `poll()` now, it'll write any change since the last time `poll` was called into the console.

### Slack

The obvious thing to so with Slack is to get notified whenever there is a new message... Sure, you could use their API - or... you could get the data direct from the Slack website.

Every so often (when we call `poll`), we'll just check if there are any new blocks of class `c-message__message_blocks`...

* Go to https://slack.com/
* Sign in and go to a slack
* When asked to open the URL, decline, and click the `Use the web version ` link
* Now Open Chrome Devtools and paste the following into the console:

```
var knownMessages = [];

function changed(value) {
  console.log("Changed!", value);
}

function poll() {
  var m = document.querySelectorAll(".c-message__message_blocks");
  for (var i=0;i<m.length;i++) {
    if (knownMessages.includes(m[i])) continue;
    knownMessages.push(m[i]);
    var msg = m[i].innerText.trim();
    changed(msg);
  }
}
```

If you run `poll()` now, it'll write any new messages since the last time `poll` was called into the console.

**Note:** For simplicity, this will store all messages that are received in `knownMessages` - so over a (long) time the webpage will use up memory and eventually crash.

### GitHub Actions

You can also check GitHub actions for the current status of the latest action.

* Go to https://github.com/espruino/Espruino/actions/workflows/build.yml
* In Chrome's Devtools you can now enter `document.querySelector(".d-table .checks-list-item-icon svg").attributes['aria-label'].value`
  * The value will be `'completed successfully'`, `'failed'`, `'queued'` or `'currently running'`

By running this in a `poll()` function that we call every so often, we can detect when this changes and push the data to the Bangle.

```
var lastState;

function changed(value) {
  console.log("Changed!", value);
}

function poll() {
  var state = document.querySelector(".d-table .checks-list-item-icon svg").attributes['aria-label'].value;
  if (!lastState || state!=lastState) {
    lastState = state;
    changed(state);
  }    
}
```

If you run `poll()` now, it'll write any state changes since the last time `poll` was called into the console.


## Sending our data to a Bangle

Normally, we'd just use a helper library like the [Puck.js library](http://www.espruino.com/Web+Bluetooth) to write data by Bluetooth,
but because we're in a bookmarklet and we can't easily add external code, we're going to have to go direct to the Web Bluetooth API.

Luckily it's not *that* bad! Paste this into the dev console of the page you were interested in getting information from:

```
var bluetoothDevice, bluetoothServer, bluetoothService, bluetoothTX;
function bluetoothConnect() {
  // First, put up a window to choose our device
  navigator.bluetooth.requestDevice({ filters: [{services: ["6e400001-b5a3-f393-e0a9-e50e24dcca9e"]},{namePrefix: "Bangle.js"}]}).then(device => {
    // Now connect to it
    console.log('Connecting to GATT Server...');
    bluetoothDevice = device;
    return device.gatt.connect();
  }).then(function(server) {
    // now get the 'UART' bluetooth service, so we can read and write!
    console.log("Connected");    
    bluetoothServer = server;
    return server.getPrimaryService("6e400001-b5a3-f393-e0a9-e50e24dcca9e");
  }).then(function(service) {
    // get the transmit service
    bluetoothService = service;
    return bluetoothService.getCharacteristic("6e400002-b5a3-f393-e0a9-e50e24dcca9e");
  }).then(function(char) {
    bluetoothTX = char;
    // get the receive service (for debugging!)
    return bluetoothService.getCharacteristic("6e400003-b5a3-f393-e0a9-e50e24dcca9e");
  }).then(function(bluetoothRX) {
    // respond to changes in the characteristic and write them to the console
    bluetoothRX.addEventListener('characteristicvaluechanged', function(event) {
      var ua = new Uint8Array(event.target.value.buffer);
      var str = "";
      ua.forEach(v => str += String.fromCharCode(v));
      console.log("BT> "+JSON.stringify(str));
    });
    return bluetoothRX.startNotifications();
  }).then(function() {    
    console.log("Completed!");
//    setInterval(poll,1000);
  });
}

function bluetoothWrite(str) {
  // FIXME - does not split what it written based on MTU!
  var u = new Uint8Array(str.length);
  for (var i=0;i<str.length;i++)
    u[i] = str.charCodeAt(i);
  console.log("Writing ",JSON.stringify(str));
  bluetoothTX.writeValue(u.buffer).then(function() {
    console.log("Written!");
  })
}
```

Now, you can run `bluetoothConnect()` - normally you can only call this from
a user input on the webpage (because of security for `navigator.bluetooth.requestDevice`)
but there's an exception for when code is run from the dev console.

Choose your Bangle.js from the list (the 4 digits after `Bangle.js` should correspond to the 4
digits in the top right of the screen). If all has gone well, this should show on the console:

```
Connecting to GATT Server...
Connected
Completed!
```

* Now, you can type:

```
bluetoothWrite("E.showMessage('Boom')\n")
```

And the message `Boom` should be shown on your Bangle's screen!

This works because you're injecting the string of text into Bangle.js's REPL.
The text is interpreted and executed as a line of JavaScript just as if you
typed it in the left-hand side of the Web IDE.

**Note:** This has run the code in the current app, so if the clock was running, at some point
the screen will redraw with the updated time.

To fix this, we can either reset the Bangle first (by sending `reset()` and waiting for 500ms)
or could even load an app that we created earlier by sending `load("myapp.app.js")`.

Since we're now connected (and we have our `poll` function from before), let's just hack something up quickly.

* Run `bluetoothWrite("reset()\n")` - this'll reset the Bangle to a blank state

Now we'll update the `changed` function to send an update to the Bangle...

* Paste this into the dev console:

```
function changed(value) {
  console.log("Changed!", value);
  bluetoothWrite("E.showMessage("+JSON.stringify(value)+")\n");
}
```

* Now type `poll()`

If the value has changed, `poll()` should call `changed(...)` which will
then show a message on the Bangle!

The next step is to automate it...

* Just add `setInterval(poll, 1000)` and the website will be checked for
updates every second.


## Putting it all together

A more option is to send an event on a global object. Then, if our
app is running we can handle it, otherwise it is ignored:

We could send `E.emit('myapp', 123)`, which will then call any event handlers
which were previously added with `E.on('myapp', ...)`.



## Next steps?

We've used Bangle.js here because it's easy to use, but now you can
escape from the browser you can control all kinds of different things.

There are a few bits of more tangible hardware at the front that
you can experiment with!
