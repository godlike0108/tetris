<template>
  <div>
    <p>{{gameMsg}}</p>
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
  },
  data () {
    return {
      lastTime: 0,
      current: 0,
      elapsed: 0,
      delta: 0,
      tick: 0,
      gameSpeed: 30,
      inputDelay: 300,
      inputReady: true,
      iFPS: 60,
      canvas: null,
      ctx: null,
      tetrisCfgs: {
        row: 20,
        column: 10,
        unitSize: 30,
      },
      tetrisCenter: 0,
      tetrisGrid: [],
      gameMsg: null,
      score: 0,
      loop: null,
      // For brick life cycle
      touchBottom: true,
      // Brick on control
      brick: {},
      brickPatterns: [],
      gamePad: {
        up: false,
        right: false,
        down: false,
        left: false
      }
    }
  },
  methods: {
    game(timestamp) {
      this.current = timestamp;
      this.elapsed = this.current - this.lastTime;
      this.tick = this.tick + 1; 

      if(this.elapsed < 1000 / this.iFPS) {
        this.loop = requestAnimationFrame(this.game);
        return;
      };
      this.loop = requestAnimationFrame(this.game);

      this.update();
      this.render();
      this.lastTime = this.current;
    },
    init() {
      // init brick patterns
      this.initBrickPatterns(this.tetrisCfgs.column / 2 - 1);

      // init data array
      this.tetrisGrid = [];
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
      // init gameMsg
      this.gameMsg = null;

      // load controller
      this.loadController();
    },
    update() {
      // bricks life cycle
      if(!this.touchBottom) {
        if(this.tick >= this.gameSpeed) {
          this.dropBrick();
          this.tick %= this.gameSpeed;
        }


        switch(true) {
          case this.gamePad.up: this.rotateBrick(); break;
          case this.gamePad.down: this.dropBrick(); break;
          case this.gamePad.left: this.moveBrick('left'); break;
          case this.gamePad.right: this.moveBrick('right'); break;
        }

      } else {
        this.generateBrick();
        this.touchBottom = false;
      }
      this.examineGameOver();
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
      this.init();
      this.loop = requestAnimationFrame(this.game);
    },
    pause() {
      cancelAnimationFrame(this.loop);
    },
    initBrickPatterns(center) {
      this.brickPatterns = [
          [
            [{x: center-1, y:0}, {x: center, y:0}, {x: center+1, y:0}, {x: center+2, y:0}],
            [{x: center, y:-1}, {x: center, y:0}, {x: center, y:1}, {x: center, y:2}]
          ],
          [
            [{x: center-1, y:0}, {x: center, y:0}, {x: center, y:1}, {x: center+1, y:0}],
            [{x: center-1, y:0}, {x: center, y:-1}, {x: center, y:0}, {x: center, y:1}],
            [{x: center-1, y:0}, {x: center, y:0}, {x: center, y:-1}, {x: center+1, y:0}],
            [{x: center, y:-1}, {x: center, y:0}, {x: center, y:1}, {x: center+1, y:0}]
          ],
          [
            [{x: center-1, y:1}, {x: center, y:0}, {x: center, y:1}, {x: center+1, y:0}],
            [{x: center-1, y:-1}, {x: center-1, y:0}, {x: center, y:0}, {x: center, y:1}]
          ],
          [
            [{x: center, y:0}, {x: center, y:1}, {x: center+1, y:0}, {x: center+1, y:1}]
          ],
          [
            [{x: center-1, y:0}, {x: center, y:0}, {x: center, y:1}],
            [{x: center-1, y:0}, {x: center, y:0}, {x: center, y:-1}],
            [{x: center, y:-1}, {x: center, y:0}, {x: center+1, y:0}],
            [{x: center, y:0}, {x: center+1, y:0}, {x: center, y:1}]
          ],
          [
            [{x: center-1, y:0}, {x: center, y:0}, {x: center+1, y:0}],
            [{x: center, y:-1}, {x: center, y:0}, {x: center, y:1}]
          ],
          [
            [{x: center-1, y:0}, {x: center, y:0}, {x: center+1, y:0}],
            [{x: center, y:-1}, {x: center, y:0}, {x: center, y:1}]
          ],
          [
            [{x: center, y:0}, {x: center+1, y:0}],
            [{x: center, y:0}, {x: center, y:1}]
          ],
          [
            [{x: center, y:0}]
          ]
      ];
    },
    generateBrick() {
      // Create a new brick
      this.brick = {};
      // Choose a random pattern
      let patternIndex = Math.floor(Math.random() * this.brickPatterns.length);
      // Set brick coordination
      this.brick.coord = JSON.parse(JSON.stringify(this.brickPatterns[patternIndex][0]));
      // Set brick properties
      this.brick.type = patternIndex;
      this.brick.dir = 0;
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
        // Update brick properties
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
        this.brick.coord = [];
        // Switch flag: brick has touched the ground
        this.touchBottom = true;
        // Clear if any row is full
        this.clearRow();
      }
    },
    moveBrick(dir) {
      if(dir === 'left' && !this.brickCollideLeft()) {
        // Clear previous position
        this.brick.coord.forEach(coord => {
          this.tetrisGrid[coord.y][coord.x].status = 0;
        });
        // Update brick position
        this.brick.coord = this.brick.coord.map(coord => {
          coord.x -= 1;
          return coord;
        });
        // Update brick base line
        this.brick.leftLimit -= 1;
        this.brick.rightLimit -= 1;
        // Mark new position
        this.brick.coord.forEach(coord => {
          this.tetrisGrid[coord.y][coord.x].status = 1;
        });
      }

      if(dir === 'right' && !this.brickCollideRight()) {
        // Clear previous position
        this.brick.coord.forEach(coord => {
          this.tetrisGrid[coord.y][coord.x].status = 0;
        });
        // Update brick position
        this.brick.coord = this.brick.coord.map(coord => {
          coord.x += 1;
          return coord;
        });
        // Update brick base line
        this.brick.leftLimit += 1;
        this.brick.rightLimit += 1;
        // Mark new position
        this.brick.coord.forEach(coord => {
          this.tetrisGrid[coord.y][coord.x].status = 1;
        });
      }
    },
    rotateBrick() {
      if(!this.inputReady) return;
      this.inputReady = false;
      // Get the diff between rotate patterns
      let delta = [];
      let type = this.brick.type;
      let dir = this.brickPatterns[this.brick.type].length;
      let nextDir = (this.brick.dir + 1) % dir;
      for(let i = 0; i < this.brickPatterns[type][this.brick.dir].length; i++) {
        let coord = {
          x: this.brickPatterns[type][nextDir][i].x - this.brickPatterns[type][this.brick.dir][i].x,
          y: this.brickPatterns[type][nextDir][i].y - this.brickPatterns[type][this.brick.dir][i].y
        }
        delta.push(coord);
      }

      // make a preview of rotated brick
      let preview = [];
      for(let i = 0; i < this.brick.coord.length; i++) {
        let newCoord = {
          x: this.brick.coord[i].x + delta[i].x,
          y: this.brick.coord[i].y + delta[i].y
        };
        preview.push(newCoord);
      }

      // preview can't collide with wall
      let collideWithWall = preview.some(coord => {
        return coord.x < 0 || coord.x >= this.tetrisCfgs.column || coord.y < 0 || coord.y >= this.tetrisCfgs.row;
      })
      if(collideWithWall) {
        this.inputReady = true;
        return;
      }

      // preview can't collide with other bricks
      let collideWithBricks = preview.some(coord => {
        return this.tetrisGrid[coord.y][coord.x].status === 2;
      })
      if(collideWithBricks) {
        this.inputReady = true;
        return;
      }

      // if no collision, update brick position
      this.brick.coord.forEach(coord => {
        this.tetrisGrid[coord.y][coord.x].status = 0;
      });
      this.brick.coord = JSON.parse(JSON.stringify(preview));
      this.brick.coord.forEach(coord => {
        this.tetrisGrid[coord.y][coord.x].status = 1;
      });
      this.brick.bottomLimit = this.brick.coord.reduce((max, cur) => Math.max(max, cur.y), 0);
      this.brick.leftLimit = this.brick.coord.reduce((min, cur) => Math.min(min, cur.x), this.tetrisCfgs.column - 1);
      this.brick.rightLimit = this.brick.coord.reduce((max, cur) => Math.max(max, cur.x), 0);
      // update direction
      this.brick.dir = nextDir;

      // set input to ready
      setTimeout(() => {this.inputReady = true;}, this.inputDelay)
    },
    loadController() {
      addEventListener('keydown', e => {
        if(e.keyCode === 38) {
          this.gamePad.up = true;
        }

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
        if(e.keyCode === 38) {
          this.gamePad.up = false;
        }

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
    clearRow() {
      let clearedRow = 0;
      let result = this.tetrisGrid.map(row => {
        let rowIsFull = row.every(col => col.status === 2);
        if(rowIsFull) {
          clearedRow++;
          return row.map(col => {
            let obj = {
              status: 0
            }
            return obj;
          });
        } else {
          return JSON.parse(JSON.stringify(row));
        }
      });
      let newRow = [];
      for(let i = 0; i < this.tetrisCfgs.column; i++) {
       newRow.push({status: 0}) ;
      }
      for(let i = 0; i < clearedRow; i++) {
        result.pop();
        result.unshift(newRow);
      }

      if(clearedRow > 0) {
        for(let i = 0; i < this.tetrisCfgs.row; i++) {
          for(let j = 0; j < this.tetrisCfgs.column; j++) {
           this.tetrisGrid[i][j].status = result[i][j].status;
          }
        }
      }
      this.score += clearedRow * 5;
    },
    examineGameOver() {
      let touchTop = this.tetrisGrid[0].some(col => col.status === 2);
      if(touchTop) {
        this.gameMsg = 'Game Over';
        this.pause();
      }
    }
  }
}
</script>

<style lang="scss" scoped>
#canvas {
  border: 2px solid black;
}
</style>
