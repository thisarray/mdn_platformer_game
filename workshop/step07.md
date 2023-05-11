# Collisions

If you play the game we have so far, you will notice another problem is the hero passes through the platforms.

The solution may seem daunting.
Fortunately, each Actor instance has a colliderect() method that detects if it collides or intersects with another Actor instance.
So we can find out if the hero collides with a platform by looping through the Array of platforms and checking if colliderect() returns true.
If it does, then we know the movement of the hero is blocked.
In that case, we just keep the hero where it was.

```js
class Hero extends Actor {
  constructor(x, y) {
    super('hero_stopped');
    this.center = [x, y];
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
}
```

We modify the move() method to detect collisions with the platforms.
At the start of the method, we store the old position of the hero.
This way we can reset to the old position if the hero is blocked.
We can use this for the edges of the screen as well.

The Array of platforms is declared in the global scope so the move() method can access it without being passed in.
I included it as an argument to make it explicit what the hero is checked against.
As a result, on_key_down() is updated as well.

```js
function on_key_down(key, mask, string) {
  if (key === keys.LEFT) {
    // Move hero left
    hero.move(-1, platforms);
  }
  else if (key === keys.RIGHT) {
    // Move hero right
    hero.move(1, platforms);
  }
}
```
