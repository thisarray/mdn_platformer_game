# Keyboard Controls

The game is no fun if we cannot control the hero so we add keyboard controls.

Before that, it looks like we will be adding more and more functionality to the hero.
From a code organization perspective, it will be better to create a Hero class that extends the Actor class to keep the code together.
What this means is that a Hero instance will look and act like an Actor instance but with additional functionality.

```js
WIDTH = 960;
HEIGHT = 600;

class Hero extends Actor {
  constructor(x, y) {
    super('hero_stopped');
    this.center = [x, y];
  }

  move(direction) {
    this.x += direction * 2.5; // Move 2.5 pixels
  }
}

let platforms, hero;

function loadLevel(levelName) {
  let levelData = JSON.parse(document.querySelector(levelName).textContent);
  platforms = [];
  for (let platform of levelData['platforms']) {
    let actor = new Actor(platform['image']);
    actor.topleft = [platform['x'], platform['y']];
    platforms.push(actor);
  }
  hero = new Hero(levelData['hero']['x'], levelData['hero']['y']);
}
```

Since we are creating a new class, we can change how it is initialized in loadLevel().

To detect when a key is pressed, we only need to write a on_key_down() function.

```js
function on_key_down(key, mask, string) {
  if (key === keys.LEFT) {
    // Move hero left
    hero.move(-1);
  }
  else if (key === keys.RIGHT) {
    // Move hero right
    hero.move(1);
  }
}
```

The key argument stores the key that was pressed.
It can be matched against the keys enumeration to figure out which key was pressed.
If the left arrow key was pressed, then we move the hero left.
If the right arrow key was pressed, then we move the hero right.

[See a live example of the code up to this point.](https://thisarray.github.io/mdn_platformer_game/05.html)

[Continue...](step06.md)
