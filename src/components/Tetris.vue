<template>
  <div>
    <p>Score: {{score}}</p>
    <canvas id="canvas"></canvas>
    <button @click="start">Start</button>
    <button @click="pause">Pause</button>
  </div>
</template>

<script>
export default {
  name: 'Tetris',
  mounted() {
    /* Setup canvas */
    this.ctx = document.querySelector('#canvas').getContext('2d');
    this.ctx.canvas.width = this.tetrisCfgs.column * this.tetrisCfgs.unitSize;
    this.ctx.canvas.height = this.tetrisCfgs.row * this.tetrisCfgs.unitSize;

    /* Initialize Tetris */
    // init data array
    for(let i = 0; i < this.tetrisCfgs.row; i++) {
      let tetrisRow = [];
      for(let j = 0; j < this.tetrisCfgs.column; j++) {
        tetrisRow.push({
          status: 0,
          x: j * this.tetrisCfgs.unitSize,
          y: i * this.tetrisCfgs.unitSize
        });
      }
      this.tetrisGrid.push(tetrisRow);
    }

    // init score
    this.score = 0;

    /* Start Game Loop */
    // this.start();
    console.log(this.tetrisGrid)
    console.log(this.ctx)

  },
  data () {
    return {
      canvas: null,
      ctx: null,
      tetrisCfgs: {
        row: 20,
        column: 10,
        unitSize: 30,
      },
      tetrisGrid: [],
      score: 0,
      loop: null,
      // For brick life cycle
      touchBottom: true,
      // Brick on control
      brick: {},
    }
  },
  computed: {
    brickCollide() {
      return this.brick.coord.some(coord => {
        return this.tetrisGrid[coord.y+1][coord.x].status === 2;
      });
    },
    brickPatterns() {
      // let center = this.tetrisCfgs.column / 2 - 1;
      let center = 4;
      let patterns = [
        [{x: center-1, y:0}, {x: center, y:0}, {x: center+1, y:0}, {x: center+2, y:0}],
        [{x: center-1, y:0}, {x: center, y:0}, {x: center, y:1}, {x: center+1, y:0}]
      ];
      return patterns;
    }
  },
  methods: {
    game() {
      this.update();
      this.render();
      this.loop = requestAnimationFrame(this.game);
    },
    update() {
      // bricks life cycle
      if(!this.touchBottom) {
        this.dropBrick();
      } else {
        this.generateBrick();
        this.touchBottom = false;
      }
    },
    render() {
      // Clear the previous frame
      this.ctx.clearRect(0, 0, this.ctx.canvas.width, this.ctx.canvas.height);
      // Render bricks
      for(let i = 0; i < this.tetrisCfgs.row; i++) {
        for(let j = 0; j < this.tetrisCfgs.column; j++) {
          if(this.tetrisGrid[i][j].status !== 0) {
            let sX = this.tetrisGrid[i][j].x;
            let sY = this.tetrisGrid[i][j].y;
            this.ctx.fillRect(sX, sY, this.tetrisCfgs.unitSize, this.tetrisCfgs.unitSize);
          }
        }
      }
    },
    start() {
      this.loop = requestAnimationFrame(this.game);
    },
    pause() {
      cancelAnimationFrame(this.loop);
    },
    generateBrick() {
      console.log('生成方塊')
      // Create a new brick
      this.brick = {};
      // Choose a random pattern 0 or 1
      let patternIndex = Math.floor(Math.random() * this.brickPatterns.length);
      // Set brick coordination
      this.brick.coord = this.brickPatterns[patternIndex].slice(0);
      // Set brick base line
      this.brick.base = this.brick.coord.reduce((max, cur) => Math.max(max, cur.y), 0);
      this.brick.coord.forEach(coord => {
        this.tetrisGrid[coord.y][coord.x].status = 1;
      });
    },
    dropBrick() {
      // Drop Brick if haven't touch ground yet
      if(this.brick.base < this.tetrisCfgs.row - 1 && !this.brickCollide) {
        console.log('真的有在掉')
        // Clear previous position
        this.brick.coord.forEach(coord => {
          this.tetrisGrid[coord.y][coord.x].status = 0;
        });
        // Update brick position
        this.brick.coord = this.brick.coord.map(coord => {
          coord.y += 1;
          return coord;
        });
        // Update brick base line
        this.brick.base = this.brick.coord.reduce((max, cur) => Math.max(max, cur.y), 0);
        // Mark new position
        this.brick.coord.forEach(coord => {
          this.tetrisGrid[coord.y][coord.x].status = 1;
        });
      } else {
        console.log('掉完了')
        // Release the controlled brick
        this.brick.coord.forEach(coord => {
          this.tetrisGrid[coord.y][coord.x].status = 2;
        });
        // Switch flag: brick has touched the ground
        this.touchBottom = true;
      }
    },
  }
}
</script>

<style lang="scss" scoped>
#canvas {
  border: 2px solid black;
}
</style>
