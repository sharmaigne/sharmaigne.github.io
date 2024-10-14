<script lang="ts">
  var height = 10;
  var width = 10;

  var cellSize = 20;
  type Cell = boolean;
  type Coord = Array<[number, number]>;
  type Grid = Cell[][];
  var grid: Grid;
  const initGrid = () =>
    (grid = Array.from({ length: height }, () => Array(width).fill(false)));
  initGrid();

  // hashmap of live cells, (x,y):neighbors
  var liveCells: { [key: string]: number } = {};

  const toggleCell = (x: number, y: number) => {
    grid[x][y] = !grid[x][y];

    if (grid[x][y]) liveCells[`${x},${y}`] = countLiveNeighbors(x, y);
    else delete liveCells[`${x},${y}`];

    informNeighbors(x, y, grid[x][y]);
    console.log(liveCells);
  };

  var speed = 2;
  var running = false;
  $: interval = speed * 10;
  var intervalId;
  $: {
    if (running) {
      // Clear the existing interval
      if (intervalId) clearInterval(intervalId);

      // Set a new interval with the updated value
      intervalId = setInterval(updateLiveCells, interval);
    }
  }
  function startSimulation() {
    if (!running) {
      running = true;
      intervalId = setInterval(updateLiveCells, speed * 10);
    }
  }

  function pauseSimulation() {
    running = false;
    if (intervalId) clearInterval(intervalId);
  }

  function stopSimulation() {
    pauseSimulation();
    grid = Array.from({ length: height }, () => Array(width).fill(false));
    liveCells = {};
  }

  function getNeigbors(x: number, y: number): Coord {
    var neighbors: Coord = [];

    for (let i = -1; i <= 1; i++)
      for (let j = -1; j <= 1; j++) {
        if (i === 0 && j === 0) continue; // skip the cell itself

        const nx = x + i;
        const ny = y + j;
        // Check if neighbor is within bounds
        if (nx >= 0 && nx < grid.length && ny >= 0 && ny < grid[0].length) {
          neighbors.push([nx, ny]);
        }
      }

    return neighbors;
  }

  // return how many neighbors are in liveCells
  function countLiveNeighbors(x: number, y: number): number {
    var liveNeighbors = 0;

    var neighbors = getNeigbors(x, y);
    for (let [i, j] of neighbors) if (`${i},${j}` in liveCells) liveNeighbors++;

    return liveNeighbors;
  }

  // update live cells when a neighbor is born or dies, like "informing" them
  function informNeighbors(x: number, y: number, state: boolean) {
    var neighbors = getNeigbors(x, y);

    for (let [i, j] of neighbors)
      if (`${i},${j}` in liveCells) liveCells[`${i},${j}`] += state * 2 - 1;
    // bool * 2 - 1 turns 0 1 to -1 1
  }

  function updateLiveCells() {
    var newLiveCells: { [key: string]: number } = {};
    // step1: get new live cells from neighbors of previous gen
    var x, y;

    // dealing with dead cells
    for (let cell in liveCells) {
      [x, y] = cell.split(",").map(Number);
      let neighbors = getNeigbors(x, y);

      for (let [i, j] of neighbors) {
        let key = `${i},${j}`;
        // ignore if alive or already accounted for or live neighbors != 3
        if (
          liveCells[key] ||
          newLiveCells[key] ||
          countLiveNeighbors(i, j) != 3
        )
          continue;

        newLiveCells[key] = 3;
        grid[i][j] = true;
      }
    }

    // dealing with the live cells
    for (let cell in liveCells) {
      [x, y] = cell.split(",").map(Number);
      if (liveCells[cell] > 1 && liveCells[cell] < 4)
        newLiveCells[cell] = liveCells[cell];
      else grid[x][y] = false;
    }

    liveCells = newLiveCells;
    for (let cell in newLiveCells) {
      [x, y] = cell.split(",").map(Number);
      if (cell in liveCells) liveCells[cell] = countLiveNeighbors(x, y);
      if (cell in liveCells) console.log("ccc");
    }
  }
</script>

<div class="wrapper">
  <table
    cellspacing="0"
    cellpadding="0"
    style="max-width: 100vw; max-height: 80vh; cursor: pointer"
  >
    {#each grid as row, x}
      <tr>
        {#each row as state, y}
          <td
            on:click={() => toggleCell(x, y)}
            class="cell {state ? 'alive' : 'dead'}"
            style="width: {cellSize}px; height: {cellSize}px"
          />
        {/each}
      </tr>
    {/each}
  </table>

  <div>
    <button on:click={startSimulation}> PLAY </button>

    <button> PAUSE </button>

    <button on:click={stopSimulation}>RESET</button>
  </div>

  <div>
    <label for="speed">speed: </label>
    <input
      type="range"
      name="speed"
      id="speed"
      min="2"
      max="100"
      bind:value={speed}
    />
  </div>
  <div>
    <label for="cell-size">grid size: </label>
    <input
      type="range"
      name="cell-size"
      id="cell-size"
      min="5"
      max="40"
      bind:value={cellSize}
    />
  </div>
  <div>
    <label for="height">height: </label>
    <input
      type="range"
      name="height"
      id="height"
      min="5"
      max="50"
      bind:value={height}
      on:change={initGrid}
    />
  </div>
  <div>
    <label for="width">width: </label>
    <input
      type="range"
      name="width"
      id="width"
      min="5"
      max="40"
      bind:value={width}
      on:change={initGrid}
    />
  </div>
</div>

<style>
  .wrapper {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-items: center;
  }
  .cell {
    border-style: solid;
    border-color: grey;
  }

  .alive {
    background-color: white;
  }

  .dead {
    background-color: black;
  }
</style>
