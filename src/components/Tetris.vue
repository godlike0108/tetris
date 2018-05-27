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

    // load controller
    this.loadController();

    /* Start Game Loop */
    // this.start();
    console.log(this.tetrisGrid)
    console.log(this.ctx)

  },
  data () {
    return {
      lastTime: 0,
      current: 0,
      elapsed: 0,
      delta: 0,
      iFPS: 5,
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
      gamePad: {
        right: false,
        down: false,
        left: false
      }
    }
  },
  computed: {
    brickPatterns() {
      let center = this.tetrisCfgs.column / 2 - 1;
      let patterns = [
        [{x: center-1, y:0}, {x: center, y:0}, {x: center+1, y:0}, {x: center+2, y:0}],
        [{x: center-1, y:0}, {x: center, y:0}, {x: center, y:1}, {x: center+1, y:0}],
        [{x: center-1, y:1}, {x: center, y:0}, {x: center, y:1}, {x: center+1, y:0}],
        [{x: center, y:0}, {x: center, y:1}, {x: center+1, y:0}, {x: center+1, y:1}],
        [{x: center, y:0}, {x: center+1, y:0}, {x: center+1, y:1}],
        [{x: center-1, y:0}, {x: center, y:0}, {x: center+1, y:0}],
        [{x: center, y:0}, {x: center+1, y:0}],
        [{x: center, y:0}]
      ];
      return patterns;
    },
  },
  methods: {
    game(timestamp) {
      this.current = timestamp;
      this.elapsed = this.current - this.lastTime;

      if(this.elapsed < 1000 / this.iFPS) {
        this.loop = requestAnimationFrame(this.game);
        return;
      };

      this.update();
      this.render();
      this.lastTime = this.current;
      this.loop = requestAnimationFrame(this.game);
    },
    update() {
      // bricks life cycle
      if(!this.touchBottom) {
        this.dropBrick();

        switch(true) {
          case this.gamePad.down: this.dropBrick(); break;
          case this.gamePad.left: this.moveBrick('left'); break;
          case this.gamePad.right: this.moveBrick('right'); break;
        }

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
      // Create a new brick
      this.brick = {};
      // Choose a random pattern 0 or 1
      let patternIndex = Math.floor(Math.random() * this.brickPatterns.length);
      // Set brick coordination
      this.brick.coord = JSON.parse(JSON.stringify(this.brickPatterns[patternIndex]));
      // Set brick limits
      this.brick.bottomLimit = this.brick.coord.reduce((max, cur) => Math.max(max, cur.y), 0);
      this.brick.leftLimit = this.brick.coord.reduce((min, cur) => Math.min(min, cur.x), this.tetrisCfgs.column - 1);
      this.brick.rightLimit = this.brick.coord.reduce((max, cur) => Math.max(max, cur.x), 0);
      this.brick.coord.forEach(coord => {
        this.tetrisGrid[coord.y][coord.x].status = 1;
      });
    },
    dropBrick() {
      // Drop Brick if haven't touch ground yet
      if(!this.brickTouchBottom() && !this.brickCollideBot()) {
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
        this.brick.bottomLimit += 1;
        // Mark new position
        this.brick.coord.forEach(coord => {
          this.tetrisGrid[coord.y][coord.x].status = 1;
        });
      } else {
        // Release the controlled brick
        this.brick.coord.forEach(coord => {
          this.tetrisGrid[coord.y][coord.x].status = 2;
        });
        // Switch flag: brick has touched the ground
        this.touchBottom = true;
      }
    },
    moveBrick(dir) {
      // Clear previous position
      this.brick.coord.forEach(coord => {
        this.tetrisGrid[coord.y][coord.x].status = 0;
      });

      // Update brick position
      if(dir === 'left' && !this.brickCollideLeft()) {
        this.brick.coord = this.brick.coord.map(coord => {
          coord.x -= 1;
          return coord;
        });
        this.brick.leftLimit -= 1;
        this.brick.rightLimit -= 1;
      }

      if(dir === 'right' && !this.brickCollideRight()) {
        this.brick.coord = this.brick.coord.map(coord => {
          coord.x += 1;
          return coord;
        });
        this.brick.leftLimit += 1;
        this.brick.rightLimit += 1;
      }
      console.log(this.brickCollideLeft(), this.brickCollideRight(), this.brick.leftLimit, this.brick.rightLimit)

      // Mark new position
      this.brick.coord.forEach(coord => {
        this.tetrisGrid[coord.y][coord.x].status = 1;
      });
    },
    loadController() {
      addEventListener('keydown', e => {
        if(e.keyCode === 40) {
          this.gamePad.down = true;
        }

        if(e.keyCode === 37) {
          this.gamePad.left = true;
        } else if(e.keyCode === 39) {
          this.gamePad.right = true;
        }
      });
      addEventListener('keyup', e => {
        if(e.keyCode === 40) {
          this.gamePad.down = false;
        }

        if(e.keyCode === 37) {
          this.gamePad.left = false;
        } else if(e.keyCode === 39) {
          this.gamePad.right = false;
        }
      });
    },
    brickTouchBottom() {
      return this.brick.bottomLimit >= this.tetrisCfgs.row - 1;
    },
    brickCollideBot() {
      return this.brick.coord.some(coord => {
        return this.tetrisGrid[coord.y+1][coord.x].status === 2;
      });
    },
    brickCollideLeft() {
      if(this.brick.leftLimit <= 0) return true;
      return this.brick.coord.some(coord => {
        return this.tetrisGrid[coord.y][coord.x-1].status === 2;
      });
    },
    brickCollideRight() {
      if(this.brick.rightLimit >= this.tetrisCfgs.column - 1) return true;
      return this.brick.coord.some(coord => {
        return this.tetrisGrid[coord.y][coord.x+1].status === 2;
      });
    },
  }
}
</script>

<style lang="scss" scoped>
#canvas {
  border: 2px solid black;
}
</style>
