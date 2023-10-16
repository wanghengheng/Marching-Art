<script>
  import { onMount } from 'svelte'
  import { MetaTags } from 'svelte-meta-tags'
  import './lib/styles.scss'

  let grid = [];

  let resolution = 50
  let showGrid = true
  let mode = "draw" // draw, erase, both, paint
  let darkMode = false
  let settings = false
  let info = false
  
  let images = []
  for (let i = 0; i < 16; i++) {
    const image = new Image(resolution, resolution)
    image.src = `/images/square${i}.png`
    if (i == 15) {
      image.onload = () => drawImages()
    }
    
    images.push(image)
  }

  function redrawImages() {
    ctx.clearRect(0, 0, canvas.width, canvas.height)
    
    drawImages()
  }
  
  let canvas;
  let ctx;

  let gridColumns;
  let gridRows;

  onMount(() => {
    ctx = canvas.getContext('2d')
    ctx.canvas.width  = window.innerWidth;
    ctx.canvas.height = window.innerHeight;
    
    gridColumns = Math.ceil(window.innerWidth/resolution + 1);
    gridRows = Math.ceil(window.innerHeight/resolution + 1);

    console.log(`Grid of ${gridColumns} columns & ${gridRows} rows`)

    resetGrid()
  })

  function resetGrid() {
    grid = []
    
    for (let i = 0; i < gridRows; i++) {
      let gridLine = []
      for (let i = 0; i < gridColumns; i++) {
        gridLine.push(0)
      }
      grid.push(gridLine)
    }

    redrawImages()
  }

  function resizeGrid() {
    gridColumns = Math.ceil(window.innerWidth/resolution + 1);
    gridRows = Math.ceil(window.innerHeight/resolution + 1);

    console.log(`Grid of ${gridColumns} columns & ${gridRows} rows`)

    for (let i = 0; i < grid.length; i++) {
      while (grid[i].length < gridColumns) {
        grid[i].push(0);
      }
    }
    while (grid.length < gridRows) {
      grid.push(new Array(gridColumns).fill(0));
    }

    redrawImages()
  }

  function randomizeGrid() {
    grid = []
    for (let i = 0; i < gridRows; i++) {
      let gridLine = []
      for (let i = 0; i < gridColumns; i++) {
        gridLine.push(Math.floor(Math.random() * 2))
      }
      grid.push(gridLine)
    }

    redrawImages()
  }
  
  function drawImages() {
    grid.map((row, y) => {
      row.map((cell, x) => {
        let state;
        try {
          state = parseInt(`${grid[y][x]}${grid[y+1][x]}${grid[y+1][x+1]}${grid[y][x+1]}`, 2)
        } catch (error) {}

        try {
          ctx.drawImage(images[state], x * resolution, y * resolution, resolution, resolution)
        } catch (error) {}

        if (showGrid) {
          if (cell === 1) {
            ctx.fillStyle = 'rgb(255 255 255 / 0.3)';
          } else {
            ctx.fillStyle = 'rgb(200 200 200 / 0.7)';
          }

          if (resolution > 24) {
            ctx.fillRect(x * resolution - 1, y * resolution - 1, 2, 2);
          } else {
            ctx.fillRect(x * resolution, y * resolution, 1, 1);
          }
        }
      })
    })
  }
  
  let mouseDown = false
  let rightClick = false

  let lastCell = {x: 9999, y: 9999}
  
  function handleMouseDown(event) {
    mouseDown = true
    if (event.button === 2) {
      rightClick = true
      lastCell = {x: 9999, y: 9999}
      onCanvasClick(event)
    } else {
      rightClick = false
    }
  }
  function handleMouseUp() {mouseDown = false}

  function onCanvasMouseDown(event) {
    if (!mouseDown) {return;}
    
    const x = event.clientX;
    const y = event.clientY;
    
    let cell = {
      x: Math.round(x/resolution),
      y: Math.round(y/resolution)
    }

    if (lastCell.x == cell.x && lastCell.y == cell.y) {
      return;
    } else {
      lastCell = cell

      drawAt(cell.x, cell.y)
    }
  }
  function onCanvasClick(event) {
    const x = event.clientX;
    const y = event.clientY;
    
    let cell = {
      x: Math.round(x/resolution),
      y: Math.round(y/resolution)
    }
    lastCell = cell

    drawAt(cell.x, cell.y)
  }

  function drawAt(x, y) {
    const newGrid = Array.from(grid)
    
    if (mode == "draw") {
      if (rightClick) {
        newGrid[y][x] = 0
      } else {
        newGrid[y][x] = 1
      }
    } else if (mode == "erase") {
      newGrid[y][x] = 0
    } else if (mode == "both") {
      if (newGrid[y][x]) {
        newGrid[y][x] = 0
      } else {
        newGrid[y][x] = 1
      }
    } else if (mode == "paint") {
      newGrid[y][x] = 1
      newGrid[y-1][x] = 1
      newGrid[y+1][x] = 1
      newGrid[y][x-1] = 1
      newGrid[y][x+1] = 1
    }

    grid = newGrid

    redrawImages()
  }

  function toggleGridDots() {
    showGrid = !showGrid
    redrawImages()
  }

  function exportCanvas() {
    const dataURL = canvas.toDataURL('image/png');
    const link = document.createElement('a');
    link.download = 'canvas.png';
    link.href = dataURL;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
  }

  // Metadata
  let title = 'Marching Art';
  let description = "Circular pixel art that connects using marching squares.";
  let image = 'https://marching-art.samalander.repl.co/card.png';
</script>

<MetaTags 
  title={title} 
  description={description} 
  twitter={{
    cardType: 'summary_large_image',
    image: image,
    imageAlt: 'Marching Art'
  }}
/>

<main class={darkMode ? "dark" : "light"} on:contextmenu|preventDefault>
  <nav id="nav">
    <button on:click={() => {info = !info}} class={info ? "selected" : null}>
      <img src="/icons/info.svg"/>
    </button>
    <button on:click={() => {darkMode = !darkMode}}>
      <img src={darkMode ? "/icons/light-mode.svg" : "/icons/dark-mode.svg"}/>
    </button>
    <button on:click={() => {settings = !settings}} class={settings ? "selected" : null}>
      <img src="/icons/settings.svg"/>
    </button>
  </nav>
  <div id="nav-bottom" class={(info || settings) ? null : "both-hidden"}>
    <div id="info" class={info ? null : "hidden"}>
      <h2>What's this?</h2>
      <p>Marching Art is a pixel-y art app that lets you draw with circular pixels that connect to each other. It uses a marching squares algorithm to create each of the connections, inspired by <a href="https://www.youtube.com/watch?v=0ZONMNUKTfU" target="_blank">this video</a> by The Coding Train. <a href="https://en.wikipedia.org/wiki/Marching_squares" target="_blank">Learn more about marching squares</a></p>
      <small>Crafted by <a href="https://www.samalander.dev/?ref=marching-art" target="_blank">Sam Cheng</a></small>
    </div>
    <div id="settings" class={settings ? null : "hidden"}>
      <label for="grid-size">Grid Size</label>
      <div class="group">
        <span>{resolution}px</span>
        <input type="range" id="grid-size" on:change={resizeGrid} bind:value={resolution} min="16" max="128"/>
      </div>
      <label for="grid-dots">Hide Grid Dots</label>
      <label class="switch">
        <input type="checkbox">
        <span class="slider" on:click={toggleGridDots}></span>
      </label>
      <label>Tools</label>
      <button on:click={resetGrid}>Clear all</button>
      <button on:click={randomizeGrid}>Randomize</button>
      <button on:click={exportCanvas}>Export to PNG</button>
    </div>
  </div>
  <aside id="toolbar">
    <div id="tools">
      <button data-tooltip="Draw" on:click={() => {mode = "draw"}} class={mode == "draw" ? "selected" : null}>
        <img src="/icons/draw.svg"/>
      </button>
      <button data-tooltip="Paint" on:click={() => {mode = "paint"}} class={mode == "paint" ? "selected" : null}>
        <img src="/icons/paint.svg"/>
      </button>
      <button data-tooltip="Erase" on:click={() => {mode = "erase"}} class={mode == "erase" ? "selected" : null}>
        <img src="/icons/erase.svg"/>
      </button>
      <button data-tooltip="Toggle" on:click={() => {mode = "both"}} class={mode == "both" ? "selected" : null}>
        <img src="/icons/both.svg"/>
      </button>
    </div>
  </aside>
  
  <canvas style="background: white;" bind:this={canvas} on:click={onCanvasClick} on:mousemove={onCanvasMouseDown} on:mousedown|preventDefault={handleMouseDown} on:mouseup={handleMouseUp}/>
</main>
