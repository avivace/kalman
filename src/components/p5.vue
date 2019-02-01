<template>
  <div>
    Using p5.js with Vue binding
    <br>
    <input v-model="x" placeholder="x">
    <input v-model="y" placeholder="y">
    <br><br>
    <div ref="canvas"></div>
    {{ states.length }}
    {{ Math.round(framerate) }}
  </div>
</template>

<script>
export default {
  data: function () {
    return {
      script: null,
      ps: null,
      x: 200,
      y: 200,
      a: 0,
      canvas: null,
      states: [],
      framerate: 0,
    }
  },
  mounted () {
    this.script = (p) => {
      //this.states = []
      /*
      A state is a list of points:
      RealInput: Real registered mouse position
      NoisyInput: Set of randomised noisy input around RealInput
      PredictedPoint: Output of the filter
      */
      class State {
        constructor() {
          this.realInput = new Point(p.mouseX, p.mouseY);
          this.noisyInput = []
          this.predictedPoint = null;
          this.dead = false;
        }

      display() {
        this.realInput.display()
      }

      update() {
        this.realInput.update()
        if (this.realInput.ttl == 0)
          this.dead = true
      }

      }

      class Point {
        constructor(x, y){
          this.x = x;
          this.y = y;
          this.ttl = 60;
        }

        display() {
          p.fill(this.ttl + 180)
          p.ellipse(this.x, this.y, 8, 8)
        }

        update() {
          this.ttl--;
        }
      }



      p.setup = () => {
        this.canvas = p.createCanvas(400, 400)
        this.canvas.parent(this.$refs.canvas)
        p.frameRate(60)
      }

      p.draw = () => {
        
        p.background(0)
        this.states.push(new State())

        p.fill(255)
        p.ellipse(p.mouseX, p.mouseY, 8, 8)
        
        
        for( let i = 0; i < this.states.length; i++) {
          this.states[i].update();
          this.states[i].display();
          if (this.states[i].dead){
            delete this.states[i]
            this.states.splice(i, 1)
          }
        }
        
        this.framerate = p.frameRate()
        
      }
    }

    const P5 = require('p5')
    this.ps = new P5(this.script)
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
