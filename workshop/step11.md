# Death

The spider enemies now patrol platforms but nothing happens when the hero runs into them.
Killing spider enemies or having the spider enemies kill the hero is a simple matter of detecting when the 2 collide.
So we use the same collision detection as platforms and coins.
If the hero has a positive y velocity, that is, the hero is falling down, then the hero is stomping the spider.
Otherwise, the hero runs into the spider and is killed by it.

```js
/*
 * Number constant for the y velocity of a jump.
 */
const HERO_JUMP_SPEED = 20;

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
  update(dt, platforms, coins, spiders) {
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
  }
}
```

When the hero stomps a spider enemy, there is a small bounce as feedback.
The y velocity of a jump is made a constant so we can calculate the bounce relative to it.

The update() function also needs to be modified to pass the spiders.

```js
function update(dt) {
  hero.update(dt, platforms, coins, spiders);
  for (let coin of coins) {
    coin.update(dt);
  }
  for (let spider of spiders) {
    spider.update(dt);
  }
}
```

To implement the death animation for a Spider instance, we need to modify the Sprite class.
Up to this point, a Sprite instance loops its animation once it reaches the end of the frameOrder Array.
The death animation for a spider enemy only runs once.
So we need a Boolean flag to track whether to loop the animation and only reset the frame index to 0 for looping animations.

```js
class Sprite extends Actor {
  constructor(name, x, y, width, height, frameOrder, fps, loop = true) {
    super(name);
    this._width = width;
    this._height = height;
    this.center = [x, y];

    this.frameOrder = frameOrder;
    this.fps = fps;
    this.loop = loop;

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
    if (this.index < this.frameOrder.length) {
      screen.blit(this.name, new Rect(this.frameOrder[this.index] * this.width, 0, this.width, this.height), new Rect(this));
    }
  }

  update(dt) {
    this.elapsed += dt;
    if (this.elapsed > (1 / this.fps)) {
      this.index += 1;
      if (this.index >= this.frameOrder.length) {
        if (this.loop) {
          this.index = 0;
        }
      }
      this.elapsed = 0;
    }
  }
}
```

The loop argument gets a default value of true because in most cases we want the animation looped.

In the case of a spider enemy that has been killed, the loop instance variable is set to false.
The frame order and animation information are altered.
The index is reset and the death animation runs through ONCE from the beginning.

```js
class Spider extends Sprite {
  constructor(x, y) {
    super('spider', x, y, 42, 32, [0, 1, 2], 8);

    this.platform = null;
    this.direction = 1;
    this.dead = false;
  }

  kill() {
    this.dead = true;
    this.frameOrder = [0, 4, 0, 4, 0, 4, 3, 3, 3, 3, 3, 3];
    this.fps = 12;
    this.loop = false;
    this.index = 0;
    this.elapsed = 0;
  }

  update(dt) {
    super.update(dt);

    if (this.dead) {
      return;
    }

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
