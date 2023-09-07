# Switching Levels

In the last step, we called reset() if the hero is standing in front of the door with the key.

```js
    if (this.landed && this.hasKey && this.colliderect(door)) {
      sounds.door.play();
      reset();
    }
```

To switch the level, we merely have to change the reset() call to something that loads another level.

There are many ways to do this.
loadLevel() already accepts the name of a level so you can just pass the name of the new level.
You can also modify the level description so it states what the next level should be.
Then pass it to loadLevel() to load the new level.

I have chosen to use a level index that starts at 0 and increments by 1.
This requires small modifications to loadLevel() and reset().

```js
let level, platforms, hero, coins, spiders, door, key;

function loadLevel() {
  let levelElement = document.querySelector('#level' + level.toFixed().padStart(2, '0')),
      levelData;
  if (levelElement == null) {
    level = 0;
    levelElement = document.querySelector('#level' + level.toFixed().padStart(2, '0'));
  }
  levelData = JSON.parse(levelElement.textContent);

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

function reset() {
  level = 0;
  loadLevel();
}
```

The new code cycles through the levels which I assume are named with 2 digits: "#level00", "#level01", ..., "#level99".
As soon as it fails to find a level, the level index is reset to 0 for "#level00".
I like this solution because I can add new levels without changing the code or the level descriptions for the first 2 levels.
I just have to embed the JSON for the new levels and number them correctly.

In the update() method of the Hero class, loadLevel() is called to reset and to switch the level.
If the level index is not incremented, then the current level is reloaded to reset it.

```js
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
          clock.schedule_unique(loadLevel, 0.3);
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
      level++;
      loadLevel();
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
```

[See a live example of the code up to this point.](https://thisarray.github.io/mdn_platformer_game/15.html)

The method you choose is up to you.
By the way, this is how all video games are structured.
One level leads to the next sometimes with narrative in between to advance the plot as needed.

If you are stuck on how to do what you want with [jsgame0.js](https://github.com/thisarray/jsgame0), try looking through [my many ports](https://thisarray.github.io/).
Perhaps one of them has an example of how to do what you want.
