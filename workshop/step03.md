# Creating Platforms

An Actor instance is what [jsgame0.js](https://github.com/thisarray/jsgame0) uses for an element on screen.
It is based around an image.
Depending on the image, an Actor instance can be the hero, an enemy, or just elements that make up the world.

To create platforms, we are going to create an Actor instance for each platform.
But first we need to make the images available so we need to load them.

Modify the imageLoader section to load the images for the various sizes of platforms.

```html
<section id="imageLoader" class="hidden">
  <img class="hidden" src="images/background.png" alt="background" data-name="background">
  <img class="hidden" src="images/grass_1x1.png" alt="grass_1x1" data-name="grass_1x1">
  <img class="hidden" src="images/grass_2x1.png" alt="grass_2x1" data-name="grass_2x1">
  <img class="hidden" src="images/grass_4x1.png" alt="grass_4x1" data-name="grass_4x1">
  <img class="hidden" src="images/grass_6x1.png" alt="grass_6x1" data-name="grass_6x1">
  <img class="hidden" src="images/grass_8x1.png" alt="grass_8x1" data-name="grass_8x1">
  <img class="hidden" src="images/ground.png" alt="ground" data-name="ground">
</section>
```

Rather than hardcode the location of the screen elements into our game, we are going to use [JavaScript Object Notation (JSON)](https://en.wikipedia.org/wiki/JSON) for the level description.
This is called **decoupling** and makes it easier for us to change a level or add new levels in the future.

Change the head section above the style tag to include the 2 levels.

```html
  <title>Recreation of MDN platformer game using jsgame0.js</title>
  <script src="jsgame0.js"></script>
  <script id="level00" type="application/json">
{
    "platforms": [
        {"image": "ground", "x": 0, "y": 546},
        {"image": "grass_4x1", "x": 420, "y": 420}
    ],
    "decoration": [
        {"frame": 2, "x": 630, "y": 504},
        {"frame": 2, "x": 663, "y": 504},
        {"frame": 2, "x": 697, "y": 504},
        {"frame": 3, "x": 756, "y": 504},
        {"frame": 1, "x": 84, "y": 504},
        {"frame": 0, "x": 252, "y": 504},
        {"frame": 4, "x": 462, "y": 378}
    ],
    "coins": [
        {"x": 147, "y": 525}, {"x": 189, "y": 525},
        {"x": 399, "y": 399}, {"x": 357, "y": 420}, {"x": 336, "y": 462},
        {"x": 819, "y": 525}, {"x": 861, "y": 525}, {"x": 903, "y": 525}
    ],
    "hero": {"x": 21, "y": 525},
    "spiders": [
    ],
    "door": {"x": 231, "y": 546},
    "key": {"x": 525, "y": 336}
}
  </script>
  <script id="level01" type="application/json">
{
    "platforms": [
        {"image": "ground", "x": 0, "y": 546},
        {"image": "grass_8x1", "x": 0, "y": 420},
        {"image": "grass_2x1", "x": 420, "y": 336},
        {"image": "grass_1x1", "x": 588, "y": 504},
        {"image": "grass_8x1", "x": 672, "y": 378},
        {"image": "grass_4x1", "x": 126, "y": 252},
        {"image": "grass_6x1", "x": 462, "y": 168},
        {"image": "grass_2x1", "x": 798, "y": 84}
    ],
    "decoration": [
        {"frame": 0, "x": 84, "y": 504}, {"frame": 1, "x": 420, "y": 504},
        {"frame": 3, "x": 672, "y": 504}, {"frame": 4, "x": 595, "y": 462},
        {"frame": 2, "x": 142, "y": 378}, {"frame": 1, "x": 168, "y": 378},
        {"frame": 0, "x": 714, "y": 336},
        {"frame": 4, "x": 420, "y": 294},
        {"frame": 1, "x": 515, "y": 126}, {"frame": 3, "x": 525, "y": 126}
    ],
    "coins": [
        {"x": 231, "y": 524}, {"x": 273, "y": 524}, {"x": 315, "y": 524}, {"x": 357, "y": 524},
        {"x": 819, "y": 524}, {"x": 861, "y": 524}, {"x": 903, "y": 524}, {"x": 945, "y": 524},
        {"x": 399, "y": 294}, {"x": 357, "y": 315}, {"x": 336, "y": 357},
        {"x": 777, "y": 357}, {"x": 819, "y": 357}, {"x": 861, "y": 357}, {"x": 903, "y": 357}, {"x": 945, "y": 357},
        {"x": 189, "y": 231}, {"x": 231, "y": 231},
        {"x": 525, "y": 147}, {"x": 567, "y": 147}, {"x": 609, "y": 147}, {"x": 651, "y": 147},
        {"x": 819, "y": 63}, {"x": 861, "y": 63}
    ],
    "hero": {"x": 21, "y": 525},
    "spiders": [{"x": 121, "y": 399}, {"x": 800, "y": 362}, {"x": 500, "y": 147}],
    "door": {"x": 169, "y": 546},
    "key": {"x": 903, "y": 105}
}
  </script>
  <style type="text/css" media="screen">
```

Now that the platform images are available and we know where to place them on screen, we can actually create them in the game.

```js
WIDTH = 960;
HEIGHT = 600;

let platforms;

function loadLevel(levelName) {
  let levelData = JSON.parse(document.querySelector(levelName).textContent);
  platforms = [];
  for (let platform of levelData['platforms']) {
    let actor = new Actor(platform['image']);
    actor.topleft = [platform['x'], platform['y']];
    platforms.push(actor);
  }
}

function reset() {
  loadLevel('#level01');
}

function draw() {
  screen.blit('background', [0, 0]);
  for (let platform of platforms) {
    platform.draw();
  }
}

window.addEventListener('load', (event) => {
  screen.init();
});
```

The new addition declares a variable to hold an Array of platform Actor instances.
The first line of loadLevel() parses the JSON level description.
Based on the information in "platforms", platform Actor instances are created and added to the Array.
For these platforms to actually show up on screen, we have to draw them in the draw() function.

loadLevel() is a separate function so we can change the level simply by changing the name passed to it.
We specify the level to load in the reset() function.
reset() is called when the user clicks in the screen canvas element to start the game.

[See a live example of the code up to this point.](https://thisarray.github.io/mdn_platformer_game/03.html)

[Continue...](step04.md)
