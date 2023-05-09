# Limiting the Movement

If you play the game we have so far, you will notice the hero does not stop at the edge of the screen.
Instead the hero keeps going.

To fix this, modify the move() method so the x position of the hero cannot be less than the left side (0) or greater than the right side (WIDTH) of the screen.

```js
class Hero extends Actor {
  constructor(x, y) {
    super('hero_stopped');
    this.center = [x, y];
  }

  move(direction) {
    this.x += direction * 2.5; // Move 2.5 pixels
    if (this.left < 0) {
      this.left = 0;
    }
    if (this.right > WIDTH) {
      this.right = WIDTH;
    }
  }
}
```
