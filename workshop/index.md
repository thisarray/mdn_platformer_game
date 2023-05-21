# Recreation of MDN platformer game using jsgame0.js

This workshop closely follows the [HTML5 Games Workshop](https://mozdevs.github.io/html5-games-workshop/)
but altered for [jsgame0.js](https://github.com/thisarray/jsgame0) instead of [Phaser](https://phaser.io).

[Play the jsgame0.js version of the MDN platformer game](../).

[jsgame0.js](https://github.com/thisarray/jsgame0) is a collection of JavaScript objects following the Pygame Zero specifications.
It makes porting Pygame Zero scripts from Python to JavaScript straightforward.
There are still language quirks like the [remainder operator (%)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Remainder) that may trip you up.
The game runs in the browser using the [Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API).

I don't know anyone that uses [jsgame0.js](https://github.com/thisarray/jsgame0) besides myself.
I have not taught this workshop.
So following the philosophy of untested code is broken, please consider this workshop broken.
Nonetheless, this workshop gives you a better understanding of how [jsgame0.js](https://github.com/thisarray/jsgame0) works.

If you do go through this workshop, please create pull requests for areas where it is difficult or unclear.
Thank you.

## Prerequisites

Since the game is written in JavaScript, knowledge of the language is necessary.
At the very least, read the [JavaScript Guide at MDN](https://developer.mozilla.org/en-US/docs/Learn/JavaScript).

Knowledge of CSS and HTML is also helpful to understand how the boilerplate works.
But you can get by with just copy and paste.

## About the art assets

The graphic and audio assets of the game in this guide have been released in the public domain under a [CC0 license](https://creativecommons.org/share-your-work/public-domain/cc0/).
These assets are:

- The images have been created by [Kenney](https://www.kenney.nl/), and are part of his [_Platformer Art: Pixel Redux_ set](https://opengameart.org/content/platformer-art-pixel-redux) (they have been scaled up, and some of them have minor edits).
- The background music track, [_Happy Adventure_](https://opengameart.org/content/happy-adventure-loop), has been created by [Rick Hoppmann](https://www.tinyworlds.org/).
- The sound effects have been randomly generated with the [Bfxr](https://www.bfxr.net/) synth.
