# Collecting Coins

A staple of every platformer game is picking up coins.
[jsgame0.js](https://github.com/thisarray/jsgame0) has minimal support for spritesheets.
In particular, it does not support a spritesheet being used in an Actor instance so we have to extend the Actor class to support it.

```js
class Coin extends Actor {
  constructor(x, y) {
    super('coin_animated');
    this._width = 22;
    this._height = 22;
    this.pos = [x, y];

    this.frameOrder = [0, 1, 2, 1];
    this.fps = 6;

    this.index = 0;
    this.elapsed = 0;
  }

  /*
   * Override width and height to reflect the size of each sprite.
   */
  get width() {
    return this._width;
  }
  get height() {
    return this._height;
  }

  /*
   * Override draw to use the spritesheet.
   */
  draw() {
    screen.blit(this.name, new Rect(this.frameOrder[this.index] * this.width, 0, this.width, this.height), new Rect(this));
  }

  update(dt) {
    this.elapsed += dt;
    if (this.elapsed > (1 / this.fps)) {
      this.index += 1;
      if (this.index >= this.frameOrder.length) {
        this.index = 0;
      }
      this.elapsed = 0;
    }
  }
}

let platforms, hero, coins;

function loadLevel(levelName) {
  let levelData = JSON.parse(document.querySelector(levelName).textContent);
  platforms = [];
  for (let platform of levelData['platforms']) {
    let actor = new Actor(platform['image']);
    actor.topleft = [platform['x'], platform['y']];
    platforms.push(actor);
  }
  hero = new Hero(levelData['hero']['x'], levelData['hero']['y']);
  coins = [];
  for (let coin of levelData['coins']) {
    coins.push(new Coin(coin['x'], coin['y']));
  }
}
```

The Coin class allows us to create Coin instances in the game.
We use the same logic as the platforms to detect when the hero picks up a coin.
When the hero collides with a coin, a sound is played and the coin is removed from the Array.

```js
  /*
   * Update the effects of gravity on this hero.
   */
  update(dt, platforms, coins) {
    if (keyboard[keys.LEFT]) {
      // Move hero left
      this.move(-1, platforms);
    }
    else if (keyboard[keys.RIGHT]) {
      // Move hero right
      this.move(1, platforms);
    }

    let oldY = this.y,
        i = 0;
    this.yVelocity += GRAVITY;

    this.y += this.yVelocity;
    for (let platform of platforms) {
      if (this.colliderect(platform)) {
        this.y = oldY;
        this.landed = true;
        this.yVelocity = 0;
        break;
      }
    }

    for (let coin of coins) {
      if (this.colliderect(coin)) {
        sounds.coin.play();
        coins.splice(i, 1);
      }
      i++;
    }
  }
```

Of course, we also have to draw and update the coin in the game loop.
Updating each coin allows it to cycle through the frames of its spritesheet to make it look like it is spinning.

```js
function draw() {
  screen.blit('background', [0, 0]);
  for (let platform of platforms) {
    platform.draw();
  }
  hero.draw();
  for (let coin of coins) {
    coin.draw();
  }
}

function update(dt) {
  hero.update(dt, platforms, coins);
  for (let coin of coins) {
    coin.update(dt);
  }
}
```
