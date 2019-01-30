<template>
  <div>
    Using p5.js with Vue binding
    <br>
    <input v-model="x" placeholder="x">
    <input v-model="y" placeholder="y">
    <br><br>
    <div ref="canvas"></div>
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
      canvas: null
    }
  },
  mounted () {
    this.script = (p) => {
      this.fade = 1

      p.setup = () => {
        this.canvas = p.createCanvas(400, 400)
        this.canvas.parent(this.$refs.canvas)
        p.frameRate(60)
      }

      p.draw = () => {
        p.background(0)
        p.fill(p.random(0,this.a),p.random(0,this.a),p.random(0,this.a))
        p.rect(this.x, this.y, p.random(-100,100), p.random(-100,100))

        if (this.fade)  {
          this.a += 3
        } else {
          this.a -= 3
        }

        
        if (this.a < 150){
          this.fade = 1
        } else if (this.a == 255) {
          this.fade = 0
        }
      }
    }

    const P5 = require('p5')
    this.ps = new P5(this.script)
    //console.log(this.ps)
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
