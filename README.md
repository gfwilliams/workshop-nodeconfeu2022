[nodeconf 2022](https://www.nodeconf.eu/workshops/integrate-external-hardware-into-your-website) -  Integrate external hardware into your website
-----------------------------------------------------------------

It's trivial to use a Mac, PC or Android phone to make pretty much any website
control real life hardware...

While not a long-term solution, this can be a great proof of concept.

Want to display some kind of statistics from your company's internal network,
but on real-life hardware? This is a great way to do that without requiring
and changes to the site or API access.

This workshop contains two main parts:

 * [**Writing a Bangle.js app**](app.md) - we'll get to grips with Bangle.js, and write a simple app
 * [**Extending a website**](website.md) - choose a website, integrate it with Bangle.js

For information on the hardware supplied for the workshop, see [The hardware page](hardware.md)

**Note:** If a Bangle.js is connected to an Android phone via Gadgetbridge then you can do
HTTP requests direct from it (http://www.espruino.com/Gadgetbridge#http-requests)
which could be the best solution if the information you're available
is accessible via a public API.
