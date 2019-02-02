<template>
	<div >
	<h2>kalman</h2>
	<canvas @mouseleave="mouseLeave" @pointermove="mouseOver" ref="ccont" :width="width" :height="height"></canvas>
	<br><br><br>
	{{ Number((fps).toFixed(0)) }} FPS
  {{ status }}
  {{ states.length }}
	</div>
</template>

<script>
export default {
	data: () =>  ({
    // Canvas size
    width: 600,
    height: 600,
		ctx: null,
    // (Expected) framerate
    framerate: 60,
    // Performance
		lastCalledTime: null,
		fps: 0, // (Measured) framerate
		framecount: 0,
    status: "paused",
    states: []
	}),
	mounted () {

		this.init();
		this.frame();

	},
	methods: {
		getRandomInt(max) {
			return Math.floor(Math.random() * Math.floor(max));
		},
    mouseLeave(){
      console.log("Mouse left, stopping")
      this.status = "paused"
    },
		mouseOver(event){
      this.status = "running"
			let canvas = this.$refs.ccont;
			var rect = canvas.getBoundingClientRect();

			this.clientX = event.clientX - rect.left;
			this.clientY = event.clientY - rect.top;
		},
		init() {
			var c = this.$refs.ccont;
			this.ctx = c.getContext("2d");
      this.ctx.fillStyle = 'blue';
      this.ctx.fillRect(0, 0, 600, 600);

      this.State = class State {
        constructor(realInput) {
          this.realInput = realInput;
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

      this.Point = class Point {
        constructor(x, y, ctx){
          this.x = x;
          this.y = y;
          this.ttl = 600;
          this.ctx = ctx;
        }

        display() {
          this.ctx.fillStyle = 'red';
          this.ctx.fillRect(this.x, this.y, 6, 6);
          
          
          
        }

        update() {
          this.ttl--;
        }
      }

		},
		frame() {
			let ctx = this.ctx
      this.ctx.fillStyle = 'blue';
      this.ctx.fillRect(0, 0, 600, 600);

      let a = new this.Point(this.clientX - 3, this.clientY - 3, ctx)
      this.states.push(new this.State(a))

      this.states.forEach(function(state, i){
        //console.log(i)
        state.display();
        state.update();
        if (state.dead){
            delete this.state
            this.states.splice(i, 1)
        }
      }.bind(this))
			

			// Compute the real framerate
			let delta = (performance.now() - this.lastCalledTime)/1000;
			this.lastCalledTime = performance.now();

			// Update the fps counter every 5 rendered frames
			// i.e. every 1000/expectedFrameRate * 5 milliseconds 
			if (this.framecount % 5 == 0){
				this.fps = 1/delta;
        this.framecount = 0;
      }
      // We done
      this.framecount++;
			// See ya in 1000/desiredFramerate milliseconds
			setTimeout(this.frame, 1000/this.framerate);

		}
	}
}


</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
