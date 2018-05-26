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
    const tetrisRow = [];
    for(let i = 0; i < this.tetrisCfgs.row; i++) {
      for(let j = 0; j < this.tetrisCfgs.column; j++) {
        tetrisRow.push({
          exist: false,
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
    }
  },
  methods: {
    game() {
      this.update();
      this.render();
      this.loop = requestAnimationFrame(this.game);
    },
    update() {
      this.test++;
    },
    render() {
      // Clear the previous frame
      this.ctx.clearRect(0, 0, this.ctx.canvas.width, this.ctx.canvas.height);


      console.log('Looping')
    },
    start() {
      this.loop = requestAnimationFrame(this.game);
    },
    pause() {
      cancelAnimationFrame(this.loop);
    }
  }
}
</script>

<style lang="scss" scoped>
#canvas {
  border: 2px solid black;
}
</style>
