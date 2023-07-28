# The Protagonist

To create the main character or the hero of the game, we do more of the same and create an Actor instance for it.

Again we modify the imageLoader section to load its image.

```html
  <img class="hidden" src="images/ground.png" alt="ground" data-name="ground">
  <img class="hidden" src="images/hero_stopped.png" alt="hero_stopped" data-name="hero_stopped">
</section>
```

Next we modify the game to create the hero.

```js
WIDTH = 960;
HEIGHT = 600;

let platforms, hero;

function loadLevel(levelName) {
  let levelData = JSON.parse(document.querySelector(levelName).textContent);
  platforms = [];
  for (let platform of levelData['platforms']) {
    let actor = new Actor(platform['image']);
    actor.topleft = [platform['x'], platform['y']];
    platforms.push(actor);
  }
  hero = new Actor('hero_stopped');
  hero.center = [levelData['hero']['x'], levelData['hero']['y']];
}

function reset() {
  loadLevel('#level01');
}

function draw() {
  screen.blit('background', [0, 0]);
  for (let platform of platforms) {
    platform.draw();
  }
  hero.draw();
}

window.addEventListener('load', (event) => {
  screen.init();
});
```

You may have noticed that unlike the platforms whose positions are set using topleft, the position of the hero is set using center.
This is because the JSON level description is not consistent and specifies the topleft coordinates of a platform but the center coordinates of the hero.
We keep this inconsistency to agree with the original MDN platformer game.

If we were creating our own game, then we can dictate the standard for the coordinates.
Having the coordinates vary in the JSON level description is one way bugs can creep into the code because it is difficult to keep track of where the coordinates are on different Actor instances.

[See a live example of the code up to this point.](https://thisarray.github.io/mdn_platformer_game/04.html)

[Continue...](step05.md)
