[nodeconf 2022](https://www.nodeconf.eu/workshops/integrate-external-hardware-into-your-website) -  Tensorflow + Web Bluetooth
-----------------------------------------------------------------

After Patty's awesome Tensoflow workshops on Tuesday, I thought I'd add a quick addition.

* Patty's workshop part 1 : https://pattyocallaghan.com/tensorflowjs-workshop
* Patty's workshop part 2 : https://github.com/pattyneta/tensorflowjs-workshop

Google provides a website called [Teachable Machine](https://teachablemachine.withgoogle.com/)
that will allow you to train your own neural network in a matter of seconds, and
even provides you with the code you'll need to use it.

**So... We're going to use Tensorflow.js, with Bangle.js, to make a website to
remind you not to pick your nose.**

For a much better explanation of what's going on, check out Patty's workshop.
This is just a bit of fun...

* First, go to https://teachablemachine.withgoogle.com/
* Choose `Get Started`, `Image Project`, and `Standard Image model`. Now you have two classes
* Click on the text `Class 1` and type `normal`
* Click on `Webcam` under it (give permission if needed)
* Make sure you're in frame and then hold the `Hold To Record` button and make
some facial expressions, look up, down, left, right etc - the aim is to
train the model with your normal look
* Click on the text `Class 2` and type `picking`
* Now train `picking` with the webcam too, ensuring that you only press
the `Hold To Record` button with your finger in frame near your nose
* When you're happy click the `Train` button and wait a minute or two
* After this you can try out your nose picking detector
* When it's done, click `Export Model`
* Click `Upload by cloud model`
* Now go to Github and create a new project so you can have an HTTPS-hosted website (or just host the file off `localhost` via HTTP).
  +  **Note:** The p5.js hosting option shown in Teachable Machine will not work with Web Bluetooth
* Copy the HTML code from Teachable Machine's Export page into a new `index.html` file
* Right before `<script type="text/javascript">` add the line `<script src="https://www.puck-js.com/puck.js"></script>`
* At the end of the code, replace the `predict` function with:

```JS
    // run the webcam image through the image model
    async function predict() {
        // predict can take in an image, video or canvas html element
        const prediction = await model.predict(webcam.canvas);
        for (let i = 0; i < maxPredictions; i++) {
            const classPrediction =
                prediction[i].className + ": " + prediction[i].probability.toFixed(2);
            labelContainer.childNodes[i].innerHTML = classPrediction;
        }
        // Check if it's likely the nose was picked for a while
        if (prediction[1].probability > 0.5) {
          pickCounter++;
          // If so, send some JS to Bangle.js to make it buzz
          if (pickCounter==20 && connected)
            Puck.write("Bangle.buzz()\n");
        } else {
          pickCounter = 0;
        }
    }
    // Web Bluetooth connections can only be started in response
    // to a user's action, so this boilerplate just puts something
    // over the whole screen that must be clicked first
    let connected = false;
    let pickCounter = 0;
    Puck.modal(function() {
      Puck.write("\n", function() { // force a connection
        connected = true;
      });
    });
```

* Save, and you're done. Load the page up in your browser, and you must now click on the page once and choose your Bangle.js
from the Web Bluetooth menu, then click the `Start` button to enable the webcam.

Now, hopefully if you start to raise your finger to pick your nose, Tensorflow
will detect it and will make the Bangle.js buzz to warn you!
