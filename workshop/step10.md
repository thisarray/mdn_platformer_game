# Walking Enemies

Another staple of every platformer game is walking enemies.
The spider enemies are more spritesheet Actor instances so we can reuse the code we wrote for coins.
To leverage code reuse, we promote the old Coin class to a Sprite class.
A new Coin class extends this Sprite class as does a Spider class.

```js
class Sprite extends Actor {
  constructor(name, x, y, width, height, frameOrder, fps) {
    super(name);
    this._width = width;
    this._height = height;
    this.center = [x, y];

    this.frameOrder = frameOrder;
    this.fps = fps;

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

class Coin extends Sprite {
  constructor(x, y) {
    super('coin_animated', x, y, 22, 22, [0, 1, 2, 1], 6);
  }
}

class Spider extends Sprite {
  constructor(x, y) {
    super('spider', x, y, 42, 32, [0, 1, 2], 8);

    this.platform = null;
    this.direction = 1;
  }

  update(dt) {
    super.update(dt);

    let rect;
    if (this.direction < 0) {
      rect = new Rect(this.left + this.direction, this.bottom, 1, this.height);
    }
    else {
      rect = new Rect(this.right + this.direction, this.bottom, 1, this.height);
    }
    if (!this.platform.colliderect(rect)) {
      // If there is no platform in front of the spider
      this.direction *= -1;
    }

    // Move at half of the speed of the hero
    this.x += this.direction * HERO_SPEED / 2;

    if (this.left < 0) {
      this.direction *= -1;
      this.x += this.direction * HERO_SPEED;
    }
    if (this.right > WIDTH) {
      this.direction *= -1;
      this.x += this.direction * HERO_SPEED;
    }
  }
}
```

The original MDN platformer game uses invisible walls to keep Spider instances from falling off platforms.
Invisible walls clutter the game.
Instead we will make the Spider instances smarter.

Each Spider instance will look one pixel wide ahead and below itself using a Rect object.
If that Rect object collides with a platform, then the Spider instance continues walking in the same direction.
If there is no platform, then the Spider instance turns around.

How do we know the platform the Spider instance is patrolling?
When we create a Spider instance while loading a level, it is assigned the platform below it.

The Spider instance also turns around if it walks outside the screen.

```js
let platforms, hero, coins, spiders;

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
  spiders = [];
  for (let spider of levelData['spiders']) {
    let s = new Spider(spider['x'], spider['y']),
        rect = new Rect(s.x, s.bottom, s.width, s.height);
    // Find the platform the current spider is patrolling
    for (let platform of platforms) {
      if (platform.colliderect(rect)) {
        s.platform = platform;
        break;
      }
    }
    spiders.push(s);
  }
}
```

Finally, we update draw() and update() so the Spider instances are drawn and updated.

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
  for (let spider of spiders) {
    spider.draw();
  }
}

function update(dt) {
  hero.update(dt, platforms, coins);
  for (let coin of coins) {
    coin.update(dt);
  }
  for (let spider of spiders) {
    spider.update(dt);
  }
}
```

To match the speed of the spider enemies to the hero so they are not too fast or too slow, we make the speed of the hero a constant.
This way we can set the speed of the spider enemies relative to the speed of the hero.

```js
/*
 * Number of pixels to move the hero.
 */
const HERO_SPEED = 2.5;

class Hero extends Actor {
  constructor(x, y) {
    super('hero_stopped');
    this.center = [x, y];

    this.landed = true;
    this.yVelocity = 0;
  }

  move(direction, platforms) {
    let oldX = this.x;
    this.x += direction * HERO_SPEED;
    if (this.left < 0) {
      this.x = oldX;
    }
    if (this.right > WIDTH) {
      this.x = oldX;
    }
    for (let platform of platforms) {
      if (this.colliderect(platform)) {
        this.x = oldX;
        break;
      }
    }
  }
```

[See a live example of the code up to this point.](https://thisarray.github.io/mdn_platformer_game/10.html)

[Continue...](step11.md)
