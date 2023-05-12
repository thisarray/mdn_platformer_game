# Jumping

A crucial element of every platformer game is jumping.

[jsgame0.js](https://github.com/thisarray/jsgame0) lacks a physics engine.
This makes the code lighter and eliminates the headaches of having to disable gravity like the original workshop.
But the downside is we have to implement the jump physics ourselves.

If you recall from physics, a jump applies upward acceleration to overcome gravity.
But gravity applies a constant downward acceleration.

What this means in code is we set an arbitrary value for gravity.
Next we set another arbitrary value for the jump that is larger than the value for gravity.
The hero gets this larger value set as its y velocity at the start of the jump.
(Due to the coordinate system in the screen, the actual y velocity is negative because 0 on the y-axis is at the top and the y-coordinate increases going down the screen.)
In each iteration of the game loop, the y velocity is reduced by gravity to pull the hero down.

```js
/*
 * Constant arbitrary number added to the y velocity to simulate gravity.
 */
const GRAVITY = 1;

class Hero extends Actor {
  constructor(x, y) {
    super('hero_stopped');
    this.center = [x, y];

    this.landed = true;
    this.yVelocity = 0;
  }

  move(direction, platforms) {
    let oldX = this.x;
    this.x += direction * 2.5; // Move 2.5 pixels
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
    const JUMP_SPEED = 20;
    let canJump = this.landed;
    if (canJump) {
      this.yVelocity = -JUMP_SPEED;
      this.landed = false;
    }
    return canJump;
  }

  /*
   * Update the effects of gravity on this hero.
   */
  update(dt, platforms) {
    let oldY = this.y;
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
  }
}
```

You may be bothered by the arbitrary values we set for gravity and jump.
Where do these values come from?
The answer is trial and error.
The developer plays the game with different values until it feels right.
For a game on the moon, the developer may set a lower gravity so the jump is higher.

```js
function update(dt) {
  hero.update(dt, platforms);
}

function on_key_down(key, mask, string) {
  if (key === keys.LEFT) {
    // Move hero left
    hero.move(-1, platforms);
  }
  else if (key === keys.RIGHT) {
    // Move hero right
    hero.move(1, platforms);
  }
  if (key === keys.UP) {
    if (hero.jump()) {
      sounds.jump.play();
    }
  }
}

window.addEventListener('load', (event) => {
  screen.init();
});
```

To simulate the constant effect of gravity, we add an update() function which calls the update() method of the hero to apply gravity.
We also add the up arrow to on_key_down() so the hero jumps.

To provide additional feedback, the original MDN platformer game plays a sound.
I already snuck in the code to play the sound above.
But we have to add the HTML5 markup to actually load the sound via an audio element.

```html
<section id="imageLoader" class="hidden">
  <img class="hidden" src="images/background.png" alt="background" data-name="background">
  <img class="hidden" src="images/grass_1x1.png" alt="grass_1x1" data-name="grass_1x1">
  <img class="hidden" src="images/grass_2x1.png" alt="grass_2x1" data-name="grass_2x1">
  <img class="hidden" src="images/grass_4x1.png" alt="grass_4x1" data-name="grass_4x1">
  <img class="hidden" src="images/grass_6x1.png" alt="grass_6x1" data-name="grass_6x1">
  <img class="hidden" src="images/grass_8x1.png" alt="grass_8x1" data-name="grass_8x1">
  <img class="hidden" src="images/ground.png" alt="ground" data-name="ground">
  <img class="hidden" src="images/hero_stopped.png" alt="hero_stopped" data-name="hero_stopped">
</section>
<section id="soundLoader" class="hidden">
  <audio class="hidden" controls preload="auto" src="sounds/jump.wav" data-name="jump">Your browser does not support the audio element.</audio>
</section>
```

If you play the game, you will notice the hero moves left and right and jumps up and down.
You can try to move the hero in midair but it requires you to repeatedly press the left or the right arrow key.
This does not feel like how a typical platformer game works.
If the hero is moving in a direction and jumps, it should keep moving in that direction while in the air.

To achieve this, we move the detection of the left and the right arrow keys into the update() method of the hero.
on_key_down() will just handle jumping when the up arrow key is pressed.

```js
  /*
   * Update the effects of gravity on this hero.
   */
  update(dt, platforms) {
    if (keyboard[keys.LEFT]) {
      // Move hero left
      this.move(-1, platforms);
    }
    else if (keyboard[keys.RIGHT]) {
      // Move hero right
      this.move(1, platforms);
    }

    let oldY = this.y;
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
  }
```

The new code demonstrates another way to detect if a key is pressed.
The keyboard global object accepts all the keys in the keys enumeration as an attribute.
The return value is true if the key is currently being pressed.
So in each iteration of the game loop, if the left arrow key is currently being pressed, then we keep moving the hero left.
If the right arrow key is currently being pressed, then we keep moving the hero right.

```js
function on_key_down(key, mask, string) {
  if (key === keys.UP) {
    if (hero.jump()) {
      sounds.jump.play();
    }
  }
}
```

After making the changes, the movement should feel more like a typical platformer game.
You can even change the hero's direction in midair.
