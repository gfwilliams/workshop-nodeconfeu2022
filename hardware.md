Workshop Hardware
===================

We're providing a few different bits of hardware for the workshop...

## Bangle.js 2

![](http://www.espruino.com/refimages/boards_Bangle.js2_thumb.png)

We've got around 30 Bangle.js 2 available for workshops. One or two of these are devices
that couldn't be sold because one sensor wasn't working - if so, the sensor
that's not working is written on the back.

For full info on using these, see: https://www.espruino.com/Bangle.js2

## Pixl.js

![](http://www.espruino.com/refimages/boards_Pixl.js_thumb.png)

This board has an LCD and 4 buttons that can be programmed.

http://www.espruino.com/Pixl.js

## Pixl.js Conference Badge

![](http://www.espruino.com/refimages/boards_Pixl.js_Multicolour_thumb.png)

These are based on badges that were used for Nodeconf in 2018. They have
a few more features than Pixl.js, and you can tell them apart by the front-facing buttons.

http://www.espruino.com/Pixl.js+Multicolour

## Analog Meter

This meter is controlled by a Puck.js Lite (`Puck.js e91e`).

To make the meter move, issue the command `analogWrite(D1,0.5)` with a
value between 0 and 1.

You can use use the red (`LED1`) and green (`LED2`) LEDs.

http://www.espruino.com/Puck.js

## Big Red Light

This is controlled by a Puck.js (`Puck.js d876`)

The light is controlled with the `FET` output. So use `FET.set()`,
`FET.reset()`, or `FET.write(x)`.

http://www.espruino.com/Puck.js

## Small Flag

This is controlled with an MDBT42Q board (`MDBT42Q 4A34`)

All you have to do is send the command `flag()` whenever
you want the flag to raise.

To make that happen it's pre-programmed with this code:

```
var s = require("servo").connect(D14);
s.move(1,3000); // move to position 0 over 1 second
var timeout;
function flag() {
  if (timeout) clearTimeout();
  s.move(0.2,2000);
  timeout = setTimeout(function() {
    timeout = undefined;
    s.move(1,2000);
  },2000);
}
setWatch(flag, BTN, {repeat:true, edge:"falling", debounce:20});
```

http://www.espruino.com/MDBT42Q

## LED Matrix

This is controlled with an MDBT42Q board (`MDBT42Q b2ee`)

All you have to do is send the command `show("Hello")` with
whatever text you want to show.

To make that happen it's pre-programmed with this code:

```
require("Font6x8").add(Graphics);

var spi = new SPI();
spi.setup({mosi:D4, sck:D11});
var disp = require("MAX7219").connect(spi, D5, 4 /* 4 chained devices */);

var g = Graphics.createArrayBuffer(32, 8, 1);
g.flip = function() { disp.raw(g.buffer); }; // To send to the display

function show(txt) {
  g.setFont("6x8").setFontAlign(0,-1);
  g.drawString(txt, 16, 0);
  g.flip();
}

show("Hello");
```

... but this could be extended easily to handle scrolling!

http://www.espruino.com/MDBT42Q
