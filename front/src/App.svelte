<script>
    let global_state = 'start'
    let num_cells= 10
    let snake = {
        /* 'position': [[0,0], [1,0],[2,0]], */
        'position': [0,0],

        'direction': 'up'
    }
    let baby = {
        'position': [2,1],
        'direction': 'up'
    }
    let row = Array.from([ num_cells ], (_, i) => i + 1);
    let cells = []


    for (let i = 0; i < num_cells; i++) {
      for (let j = 0; j < num_cells; j++) {
        cells.push([i,j]);
      }
    }

    function arrayContainsArray(outerArray, innerArray) {
      return outerArray.some(outerElement => {
        if (Array.isArray(outerElement) && outerElement.length === innerArray.length) {
          return outerElement.every((value, index) => value === innerArray[index]);
        }
        return false;
      });
    }
    
    function randomInteger() {
        return Math.floor(Math.random() * num_cells)
    }
    function random_cell() {
        return [randomInteger(), randomInteger()]
    }

    function isSnake(cell, snake) {
        return cell[0] == snake.position[0] && cell[1] == snake.position[1]
    }
    function isBaby(cell, baby) {
        return cell[0] == baby.position[0] && cell[1] == baby.position[1]
    }
    function handleKeyDown(event) {
        if (['a', 'ArrowLeft'].includes(event.key)) {
            snake.direction = 'left';
        } else if (['d', 'ArrowRight'].includes(event.key)) {
            snake.direction = 'right';
        } else if (['w', 'ArrowUp'].includes(event.key)) {
            snake.direction = 'up';
        } else if (['s', 'ArrowDown'].includes(event.key)) {
            snake.direction = 'down';
        }
      }

    function start_game() {
        global_state = 'playing'
        console.log('starting game');
        let interval = setInterval(game_loop,200);
    }

    function updateSnake() {

        if (snake.direction === 'right') {
            snake.position[1]++;
        } else if (snake.direction === 'down') {
            snake.position[0]++;
        } else if (snake.direction === 'left') {
            snake.position[1]--;
        } else if (snake.direction === 'up') {
            snake.position[0]--;
        }

        if (snake.position[0] < 0) {
            snake.position[0] += num_cells
        }
        if (snake.position[1] < 0) {
            snake.position[1] += num_cells
        }

        snake.position[0] %= num_cells;
        snake.position[1] %= num_cells;
    }

    function updateBaby() {
        baby.position = random_cell()
    }

    function eatBaby() {
        return snake.position[0] == baby.position[0] && snake.position[1] == baby.position[1]
    }

    function game_loop() {
        updateSnake()
        if (eatBaby()) {
            updateBaby()
        }
    }
</script>


<style>
  :global(.grid) {
    display: grid;
    grid-template-columns: repeat(var(--grid-size), 1fr);
    grid-template-rows: repeat(var(--grid-size), 1fr);
    gap: 1px;
    border: 1px solid red;
    width: 300px;
    height: 300px;
  }
  .cell {
    border: 2px solid black;
  }

  .cell.snake{
      background-color: black;
  }

  .cell.baby{
      background-color: pink;
  }
</style>


<svelte:window on:keydown={handleKeyDown} />
{#if global_state == 'start'}
    <h1> hello snake lord lets go yeah </h1>
{/if}

<div class="grid" style="--grid-size: {num_cells}">
    {#each cells as cell}
        <div class="cell" 
            class:snake={isSnake(cell, snake)}
            class:baby={isBaby(cell, baby)}
        ></div>
    {/each}
</div>

{#if global_state == 'start'}
    <button on:click={start_game}>
        Start game
    </button>
{/if}
