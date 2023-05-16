# Scoreboard

Right now the player collects coins but we do nothing with that information.
It would be nice to track how many coins the player has collected.
The original MDN platformer game draws the information using a bitmap font which [jsgame0.js](https://github.com/thisarray/jsgame0) does not support.
Fortunately, we can fake it by treating the image like a spritesheet and doing what we have been doing so far.

```html
<section id="imageLoader" class="hidden">
  <img class="hidden" src="images/background.png" alt="background" data-name="background">
  <img class="hidden" src="images/coin_animated.png" alt="coin_animated" data-name="coin_animated">
  <img class="hidden" src="images/coin_icon.png" alt="coin_icon" data-name="coin_icon">
  <img class="hidden" src="images/grass_1x1.png" alt="grass_1x1" data-name="grass_1x1">
  <img class="hidden" src="images/grass_2x1.png" alt="grass_2x1" data-name="grass_2x1">
  <img class="hidden" src="images/grass_4x1.png" alt="grass_4x1" data-name="grass_4x1">
  <img class="hidden" src="images/grass_6x1.png" alt="grass_6x1" data-name="grass_6x1">
  <img class="hidden" src="images/grass_8x1.png" alt="grass_8x1" data-name="grass_8x1">
  <img class="hidden" src="images/ground.png" alt="ground" data-name="ground">
  <img class="hidden" src="images/hero_stopped.png" alt="hero_stopped" data-name="hero_stopped">
  <img class="hidden" src="images/numbers.png" alt="numbers" data-name="numbers">
  <img class="hidden" src="images/spider.png" alt="spider" data-name="spider">
</section>
```

After adding the images to be loaded, we store the location of each character in the spritesheet in a Map object.
This way we can easily find a character's location in the spritesheet.

```js
/*
 * Number constant for the y velocity of a jump.
 */
const HERO_JUMP_SPEED = 20;

/*
 * Map a string to its corresponding sprite in numbers.png.
 */
const NUMBERS_MAP = new Map([
  ['0', new Rect(0, 0, 20, 26)],
  ['1', new Rect(20, 0, 20, 26)],
  ['2', new Rect(40, 0, 20, 26)],
  ['3', new Rect(60, 0, 20, 26)],
  ['4', new Rect(80, 0, 20, 26)],
  ['5', new Rect(100, 0, 20, 26)],
  ['6', new Rect(0, 26, 20, 26)],
  ['7', new Rect(20, 26, 20, 26)],
  ['8', new Rect(40, 26, 20, 26)],
  ['9', new Rect(60, 26, 20, 26)],
  ['x', new Rect(80, 26, 20, 26)]
]);

class Hero extends Actor {
```

In the draw() function, we draw the coin followed by "x" and the digits.
We use a JavaScript cheat that when a String and a Number are concatenated, the Number is converted to a String.
Next we loop through the characters of the string and draw the appropriate portion of the spritesheet.

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

  // Draw the HUD
  screen.blit('coin_icon', [10, 10]);
  let x = 10 + 38,
      value = 'x' + hero.coinPickupCount;
  for (let c of value) {
    screen.blit('numbers', NUMBERS_MAP.get(c), new Rect(x, 16, 20, 26));
    x += 20;
  }
}
```

All that remains is to add the count of coins the player has collected to the Hero class.
The count is initialized to 0 when the Hero instance is created.
Each time the hero collides with a coin, the count is incremented.

```js
class Hero extends Actor {
  constructor(x, y) {
    super('hero_stopped');
    this.center = [x, y];

    this.landed = true;
    this.yVelocity = 0;

    this.coinPickupCount = 0;
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
  }
}
```
