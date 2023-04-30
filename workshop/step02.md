# Game Loop

The **game loop** is the heart of every video game.
During each iteration of the game loop, the state of the game is updated and the new state is drawn to the screen.
Update is done in the update() function.
Drawing is done in the draw() function.

Edit index.html to add the draw() function to the JavaScript.

```js
WIDTH = 960;
HEIGHT = 600;

function draw() {
  screen.blit('background', [0, 0]);
}

window.addEventListener('load', (event) => {
  screen.init();
});
```

In draw(), the background image is drawn to the screen in each iteration of the game loop.
If you open "index.html" in your browser and click on the canvas, you will see the background image take up the screen.
