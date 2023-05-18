# Hero Animation

Up to this point, the hero has been a static image that does not change.
But we can animate it using a spritesheet as well.
To do this, the Hero class extends the Sprite class to inherit most of the functionality.
Of course, we need to swap out the proper frames for when the hero is stopped, jumping, falling, or running.
The new Hero class is moved below the declaration of the Sprite class.

```js
class Hero extends Sprite {
  constructor(x, y) {
    super('hero', x, y, 36, 42, [0], 8);

    this.landed = true;
    this.yVelocity = 0;

    this.coinPickupCount = 0;
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
  update(dt, platforms, coins, spiders) {
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

Most of the changes are in the update() method.
If the hero has a non-zero y velocity, then the hero is either jumping if it is negative or falling if it is positive.
Otherwise, we know the hero is running if the left arrow key or the right arrow key is pressed.
Finally, the hero defaults to being stopped.

There is an additional check of the frame order to avoid resetting the frame order if the hero is already in that state.
Without this check, the index would get reset to the first frame in every iteration of the game loop.
This would defeat the animation and make it look like there is no change.

## Facing the Right Direction

The spritesheet for the hero always faces right.
Phaser has the ability to flip the sprite but not [jsgame0.js](https://github.com/thisarray/jsgame0).
Instead, we use an external tool to flip the spritesheet so the hero faces left and save it as "hero_left.png".
Then depending on the direction of the hero, we merely change the spritesheet used ("hero.png" or "hero_left.png").
The code that does this is in the move method().

The external tool is just some JavaScript that draws to a canvas element.
The detail are outside the scope of this workshop.

To load the new spritesheet, replace the old static image with these new spritesheets.

```html
  <img class="hidden" src="images/hero_stopped.png" alt="hero_stopped" data-name="hero_stopped">
```

```html
  <img class="hidden" src="images/hero.png" alt="hero" data-name="hero">
  <img class="hidden" src="images/hero_left.png" alt="hero_left" data-name="hero_left">
```
