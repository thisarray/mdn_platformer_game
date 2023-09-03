# Door and Key

To give the player a way to win, the original MDN platformer game added a door and a key.
This is something we have done multiple times already.

## Add the HTML to Load the Images and Sounds

```html
  <img class="hidden" src="images/door.png" alt="door" data-name="door">
  <img class="hidden" src="images/grass_1x1.png" alt="grass_1x1" data-name="grass_1x1">
  <img class="hidden" src="images/grass_2x1.png" alt="grass_2x1" data-name="grass_2x1">
  <img class="hidden" src="images/grass_4x1.png" alt="grass_4x1" data-name="grass_4x1">
  <img class="hidden" src="images/grass_6x1.png" alt="grass_6x1" data-name="grass_6x1">
  <img class="hidden" src="images/grass_8x1.png" alt="grass_8x1" data-name="grass_8x1">
  <img class="hidden" src="images/ground.png" alt="ground" data-name="ground">
  <img class="hidden" src="images/hero.png" alt="hero" data-name="hero">
  <img class="hidden" src="images/hero_left.png" alt="hero_left" data-name="hero_left">
  <img class="hidden" src="images/key.png" alt="key" data-name="key">
  <img class="hidden" src="images/key_icon.png" alt="key_icon" data-name="key_icon">
```

```html
  <audio class="hidden" controls preload="auto" src="sounds/door.wav" data-name="door">Your browser does not support the audio element.</audio>
  <audio class="hidden" controls preload="auto" src="sounds/jump.wav" data-name="jump">Your browser does not support the audio element.</audio>
  <audio class="hidden" controls preload="auto" src="sounds/key.wav" data-name="key">Your browser does not support the audio element.</audio>
```

## Create the Sprite and Actor Instances for the Door and the Key

When a level is loaded, we create the appropriate Sprite instance for the door because it uses a spritesheet and Actor instance for the key.
Note the door coordinates specify its midbottom.
Once again, we keep this inconsistency to agree with the original MDN platformer game.

```js
let platforms, hero, coins, spiders, door, key;

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
  door = new Sprite('door', levelData['door']['x'], levelData['door']['y'], 42, 66, [0], 1);
  door.midbottom = [levelData['door']['x'], levelData['door']['y']];
  key = new Actor('key');
  key.center = [levelData['key']['x'], levelData['key']['y']];
}
```

## Update draw() to Draw the Door and the Key

```js
function draw() {
  screen.blit('background', [0, 0]);
  door.draw();
  if (!hero.hasKey) {
    key.draw();
  }
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

  // Draw the HUD
  if (hero.hasKey) {
    screen.blit('key_icon', new Rect(34, 0, 34, 30), new Rect(10, 10, 34, 30));
  }
  else {
    screen.blit('key_icon', new Rect(0, 0, 34, 30), new Rect(10, 10, 34, 30));
  }
  screen.blit('coin_icon', [51, 10]);
  let x = 51 + 38,
      value = 'x' + hero.coinPickupCount;
  for (let c of value) {
    screen.blit('numbers', NUMBERS_MAP.get(c), new Rect(x, 16, 20, 26));
    x += 20;
  }
}
```

A key icon to the left of the scoreboard shows whether the player has collected the key.
This is easily drawn but requires us to move the scoreboard to the right to make room for it.

```js
class Hero extends Sprite {
  constructor(x, y) {
    super('hero', x, y, 36, 42, [0], 8);

    this.landed = true;
    this.yVelocity = 0;

    this.coinPickupCount = 0;
    this.hasKey = false;
  }

  move(direction, platforms) {
    if (direction < 0) {
      this.name = 'hero_left';
    }
    else {
      this.name = 'hero';
    }

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

  jump() {
    let canJump = this.landed;
    if (canJump) {
      this.yVelocity = -HERO_JUMP_SPEED;
      this.landed = false;
    }
    return canJump;
  }

  /*
   * Update the effects of gravity on this hero.
   */
  update(dt, platforms, coins, spiders, door, key) {
    super.update(dt);

    let oldY = this.y,
        i = 0,
        moving = false;
    if (keyboard[keys.LEFT]) {
      // Move hero left
      this.move(-1, platforms);
      moving = true;
    }
    else if (keyboard[keys.RIGHT]) {
      // Move hero right
      this.move(1, platforms);
      moving = true;
    }

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
        this.coinPickupCount++;
        sounds.coin.play();
        coins.splice(i, 1);
      }
      i++;
    }

    for (let spider of spiders) {
      if (spider.dead) {
        continue;
      }
      if (this.colliderect(spider)) {
        if (this.yVelocity > 0) {
          // Kill enemies when hero is falling
          spider.kill();
          this.yVelocity = -Math.floor(HERO_JUMP_SPEED / 4);
          this.landed = false;
        }
        else {
          // Game over -> restart the game
          clock.schedule_unique(reset, 0.3);
        }
        sounds.stomp.play();
      }
    }

    if (this.colliderect(key)) {
      sounds.key.play();
      this.hasKey = true;
    }

    if (this.landed && this.hasKey && this.colliderect(door)) {
      sounds.door.play();
      reset();
    }

    if (this.yVelocity < 0) {
      // hero is jumping
      if (this.frameOrder[0] !== 3) {
        this.frameOrder = [3];
        this.index = 0;
        this.elapsed = 0;
      }
    }
    else if ((this.yVelocity > 0) && (!this.landed)) {
      // hero is falling
      if (this.frameOrder[0] !== 4) {
        this.frameOrder = [4];
        this.index = 0;
        this.elapsed = 0;
      }
    }
    else if (moving) {
      // hero is running
      if (this.frameOrder[0] !== 1) {
        this.frameOrder = [1, 2];
        this.index = 0;
        this.elapsed = 0;
      }
    }
    else {
      if (this.frameOrder[0] !== 0) {
        this.frameOrder = [0];
        this.index = 0;
        this.elapsed = 0;
      }
    }
  }
}
```

Detecting when the hero picks up the key is the same as when the hero picks up a coin.
If the hero is standing by the door with the key, then the level resets.

[See a live example of the code up to this point.](https://thisarray.github.io/mdn_platformer_game/14.html)

[Continue...](step15.md)
