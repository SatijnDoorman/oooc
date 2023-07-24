<script>
  // todo
  // mag niet terug

  // grid
  let num_cells = 15;
  let row = Array.from([num_cells], (_, i) => i + 1);
  let cells = [];
  for (let i = 0; i < num_cells; i++) {
    for (let j = 0; j < num_cells; j++) {
      cells.push([i, j]);
    }
  }

  // game loop
  let interval;
  let animationFrame;
  let difficulty = 20;

  //game state

  let global_state;
  let snake;
  let baby;
  let mouthOpen = false;
  let score = 0;

  function initializeGame() {
    global_state = "start";
    let centerCell1 = [Math.floor(num_cells / 2), Math.floor(num_cells / 2)];
    let centerCell2 = [...centerCell1];
    centerCell2[1] += 1;
    let centerCell3 = [...centerCell1];
    centerCell3[1] += 2;

    snake = {
      position: [centerCell1, centerCell2, centerCell3],
      direction: "up",
    };
    let initialBaby = [...centerCell1];
    initialBaby[0] += 4;

    baby = {
      position: initialBaby,
      direction: "up",
    };
  }
  initializeGame();

  function arrayContainsArray(outerArray, innerArray) {
    return outerArray.some((outerElement) => {
      if (
        Array.isArray(outerElement) &&
        outerElement.length === innerArray.length
      ) {
        return outerElement.every(
          (value, index) => value === innerArray[index]
        );
      }
      return false;
    });
  }

  function randomInteger() {
    return Math.floor(Math.random() * num_cells);
  }
  function random_cell() {
    return [randomInteger(), randomInteger()];
  }

  function isSnake(cell, snake) {
    return arrayContainsArray(snake.position, cell);
  }
  function isBaby(cell, baby) {
    return cell[0] == baby.position[0] && cell[1] == baby.position[1];
  }

  function isHead(cell, snake) {
    return cell[0] == snake.position[0][0] && cell[1] == snake.position[0][1];
  }

  function handleKeyDown(event) {
    if (["a", "ArrowLeft"].includes(event.key)) {
      if (snake.direction != "right") {
        snake.direction = "left";
      }
    } else if (["d", "ArrowRight"].includes(event.key)) {
      if (snake.direction != "left") {
        snake.direction = "right";
      }
    } else if (["w", "ArrowUp"].includes(event.key)) {
      if (snake.direction != "down") {
        snake.direction = "up";
      }
    } else if (["s", "ArrowDown"].includes(event.key)) {
      if (snake.direction != "up") {
        snake.direction = "down";
      }
    }
  }

  function start_game() {
    initializeGame();
    global_state = "playing";
    console.log("starting game");

    animationFrame = requestAnimationFrame(game_loop);

    // interval = setInterval(game_loop, difficulty);
  }

  function die() {
    global_state = "start";
    console.log("game over");
    cancelAnimationFrame(animationFrame);
    // clearInterval(interval);
  }

  function updateSnake() {
    let newhead = [...snake.position[0]];

    let newsnake = JSON.parse(JSON.stringify(snake));

    if (snake.direction === "right") {
      newhead[1]++;
    } else if (snake.direction === "down") {
      newhead[0]++;
    } else if (snake.direction === "left") {
      newhead[1]--;
    } else if (snake.direction === "up") {
      newhead[0]--;
    }

    newsnake.position.unshift(newhead);
    newsnake.position = [...newsnake.position];
    return newsnake;
  }

  function updateBaby() {
    while (true) {
      baby.position = random_cell();
      if (!arrayContainsArray(snake.position, baby.position)) {
        break;
      }
    }
  }

  function eatBaby() {
    let head = snake.position[0];
    return head[0] == baby.position[0] && head[1] == baby.position[1];
  }

  function hitsSelf(newsnake) {
    let head = newsnake.position[0];
    let body = [...newsnake.position];
    body.shift();
    return arrayContainsArray(body, head);
  }

  function hitsWall(newsnake) {
    let head = newsnake.position[0];

    return (
      head[0] < 0 || head[0] >= num_cells || head[1] < 0 || head[1] >= num_cells
    );
  }

  let lastUpdate = 0;
  function game_loop() {
    if (lastUpdate % difficulty === 0) {
      let newsnake = updateSnake();

      if (hitsSelf(newsnake) || hitsWall(newsnake)) {
        die();
        return;
      }
      snake = newsnake;
      if (eatBaby()) {
        updateBaby();
        mouthOpen = true;
        score++;
        if (score % 5 === 0) {
          difficulty = Math.max(1, difficulty - 5);
          alert("NOG SNELLER!");
        }
      } else {
        mouthOpen = false;
        snake.position.pop();
        snake.position = [...snake.position];
      }
    }

    lastUpdate++;
    requestAnimationFrame(game_loop);
  }
</script>

<svelte:window on:keydown={handleKeyDown} />
{#if global_state == "start"}
  <h1>hello snake lord lets go yeah</h1>
{/if}

<p>Score: {score}</p>

<div class="grid" style="--grid-size: {num_cells}">
  {#each cells as cell}
    <div
      class="cell"
      class:snake={isSnake(cell, snake)}
      class:baby={isBaby(cell, baby)}
      class:head={isHead(cell, snake)}
    >
      {#if isHead(cell, snake)}
        <div class="smile" alt="smile" class:open={mouthOpen} />
      {/if}
    </div>
  {/each}
</div>

{#if global_state == "start"}
  <button on:click={start_game}> Start game </button>
{/if}

<style>
  :global(.grid) {
    display: grid;
    grid-template-columns: repeat(var(--grid-size), 1fr);
    grid-template-rows: repeat(var(--grid-size), 1fr);
    gap: 1px;
    width: 300px;
    height: 300px;
    background-image: url("/tropical-jungle-1.jpg");
    background-size: cover;
  }

  button {
    position: relative;
    z-index: 124789;
  }

  .cell {
    border: 0px solid black;
  }

  .cell.snake {
    background-color: greenyellow;
    border-radius: 50%;
    border: 2px solid green;
  }

  .cell.snake.head {
    background-color: red;
    position: relative;
  }

  .cell.snake.head .smile {
    width: 100%;
    height: 100%;
    animation: scream 2s infinite 0.5s;
    position: absolute;
    background-image: url("/smile.png");
    background-size: contain;
    background-position: bottom;
    background-repeat: no-repeat;
    transform: scale(1.5);
    transition: 0.3s background-image;
  }

  .cell.snake.head .smile.open {
    background-image: url("/smile-open.png");
  }

  .cell.baby {
    background-image: url("/baby4.jpg");
    background-size: 180%;
    background-position: 50% 0;
    border-radius: 50%;
    transform: scale(2);
    animation: scream 5s infinite;
  }

  @keyframes scream {
    0% {
      transform: scale(1);
    }

    90% {
      transform: scale(1);
    }
    95% {
      transform: scale(1.5);
    }
    100% {
      transform: scale(1);
    }
  }

  @keyframes bite {
    0% {
      background-image: url("/smile.png");
    }

    90% {
      background-image: url("/smile.png");
    }
    95% {
      background-image: url("/smile-open.png");
    }
    100% {
      background-image: url("/smile.png");
    }
  }
</style>
