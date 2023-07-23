# Project Layout

For [jsgame0.js](https://github.com/thisarray/jsgame0) to work, it expects assets in certain places.
This is a requirement of Pygame Zero upon which it was based.

Images should be placed in an "images" directory.
Sounds should be placed in a "sounds" directory.
You can place music either in the "sounds" directory or in a dedicated "music" directory depending on how you like to organize the files.
I have chosen to keep the music in the "sounds" directory to mimic the original MDN platformer game.

Make sure your game directory looks like this:

```bash
game
|-- images
|-- sounds
|-- index.html
|-- jsgame0.js
```

Copy the following boilerplate and save it as index.html.

```html
<!DOCTYPE html>

<html lang="en-US">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Recreation of MDN platformer game using jsgame0.js</title>
  <script src="jsgame0.js"></script>
  <style type="text/css" media="screen">
body {
  background-color: white;
  color: black;
}
.hidden {
  display: none;
}
#original {
  margin-left: 1em;
}
  </style>
</head>

<body>
<section id="imageLoader" class="hidden">
  <img class="hidden" src="images/background.png" alt="background" data-name="background">
</section>

<main>
<h1>Recreation of MDN platformer game using jsgame0.js</h1>

<canvas id="screen">
The game screen appears here if your browser supports the Canvas API.
</canvas>
<section id="controls">
  <button type="button" id="reset">Reset</button>
  <button type="button" id="pause">Pause</button>
</section>

<p>Recreation of MDN platformer game using <a href="https://github.com/thisarray/jsgame0">jsgame0.js</a>.</p>

<h2>Attribution</h2>

<p>Images were made by <a href="https://www.kenney.nl/">Kenney</a> and published under <a href="https://creativecommons.org/share-your-work/public-domain/cc0/">CC0 - Public Domain license</a>.</p>

<p>Audio assets have been released in the public domain under a <a href="https://creativecommons.org/share-your-work/public-domain/cc0/">CC0 license</a>.</p>

</main>

<script>
WIDTH = 960;
HEIGHT = 600;

window.addEventListener('load', (event) => {
  screen.init();
});
</script>
</body>

</html>
```

When you open "index.html" in your browser, you should see a black canvas element with a play arrow.
This canvas element will be the screen of the game.
We will be editing index.html as we develop the game.

## Explanation of the Boilerplate

The boilerplate does a lot for us.

```html
<!DOCTYPE html>

<html lang="en-US">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Recreation of MDN platformer game using jsgame0.js</title>
  <script src="jsgame0.js"></script>
```

This is the standard HTML5 preamble to tell the browser the page uses HTML5, is in US English, and contains characters encoded using UTF-8.
The viewport meta tag tells the browser not to zoom in on the page.
The title tag contains the title of the page.
The script tag tells the browser where to find the JavaScript file for [jsgame0.js](https://github.com/thisarray/jsgame0) relative to the page.

```html
<section id="imageLoader" class="hidden">
  <img class="hidden" src="images/background.png" alt="background" data-name="background">
</section>
```

This section loads background.png using an image tag.
As we will see later, more image tags will be added to load additional images.
Thanks to the image tag, the page does not require a local web server to serve images.

The tags get a CSS class of "hidden" to hide them.
Without this CSS class, background.png would show up on the page outside of the screen which is not what we want.
This does not stop the browser from fetching the images though.

```html
<script>
WIDTH = 960;
HEIGHT = 600;

window.addEventListener('load', (event) => {
  screen.init();
});
</script>
```

The part between the script tag is where we write our JavaScript code for the game.
The WIDTH and the HEIGHT tell [jsgame0.js](https://github.com/thisarray/jsgame0) how big to make the screen canvas element.

We wait in an event listener for the page to finish loading to make sure all the assets are available before running.

[Continue...](step02.md)
