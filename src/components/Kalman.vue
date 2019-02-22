<template>
	<div>
		<h2 class="title">Kalman Filter demo</h2>
		<h3 class="subtitle">Antonio Vivace, May 2019</h3>
		<canvas @mouseleave="mouseLeave" @pointermove="mouseOver" ref="ccont" :width="width" :height="height"></canvas>
		<br><br><br>
		<span class="stats">
			{{ Number((fps).toFixed(0)) }} FPS
			{{ status }}
			{{ states.length }}
		</span>
	</div>
</template>
<script>
export default {
	data: () => ({
		// Canvas size
		width: 600,
		height: 600,
		ctx: null,
		// Performance
		lastCalledTime: null,
		fps: 0, // (Measured) framerate
		framecount: 0,
		status: "paused",
		states: []
	}),
	mounted() {

		this.init();
		this.frame();

	},
	methods: {
		getRandomInt(max) {
			return Math.floor(Math.random() * Math.floor(max));
		},
		mouseLeave() {
			console.log("Mouse left, stopping")
			this.status = "paused"
		},
		mouseOver(event) {
			this.status = "running"
			let canvas = this.$refs.ccont;
			var rect = canvas.getBoundingClientRect();

			this.clientX = event.clientX - rect.left;
			this.clientY = event.clientY - rect.top;
		},
		init() {
			var A = window.$M([
				[1, 0, 0.2, 0],
				[0, 1, 0, 0.2],
				[0, 0, 1, 0],
				[0, 0, 0, 1]
			]);

			var B = window.$M([
				[1, 0, 0, 0],
				[0, 1, 0, 0],
				[0, 0, 1, 0],
				[0, 0, 0, 1]
			]);

			var H = window.$M([
				[1, 0, 0, 0],
				[0, 1, 0, 0],
				[0, 0, 1, 0],
				[0, 0, 0, 1]
			]);

			var Q = window.$M([
				[0.001, 0, 0, 0],
				[0, 0.001, 0, 0],
				[0, 0, 0, 0],
				[0, 0, 0, 0]
			]);

			var R = window.$M([
				[0.1, 0, 0, 0],
				[0, 0.1, 0, 0],
				[0, 0, 0.1, 0],
				[0, 0, 0, 0.1]
			]);

			var last_x = window.$V([0, 0, 0, 0]);

			var last_P = window.$M([
				[0, 0, 0, 0],
				[0, 0, 0, 0],
				[0, 0, 0, 0],
				[0, 0, 0, 0]
			]);

			console.log(A)
			var c = this.$refs.ccont;
			this.ctx = c.getContext("2d", { alpha: false });
			this.ctx.fillStyle = "#37474f";
			this.ctx.fillRect(0, 0, 600, 600);

			this.State = class State {
				constructor(realInput, noisyInput) {
					this.realInput = realInput;
					this.noisyInput = noisyInput;
					this.predictedPoint = null;
					this.dead = false;
				}

				display() {
					this.realInput.display("red")
					this.noisyInput.display("#558b2f")
				}

				update() {
					this.realInput.update()
					this.noisyInput.update()
					if (this.realInput.ttl == 0)
						this.dead = true
				}

			}

			this.Point = class Point {
				constructor(x, y, ctx) {
					this.x = x;
					this.y = y;
					this.ttl = 520;
					this.ctx = ctx;
				}

				display(color) {
					this.ctx.fillStyle = color;
					this.ctx.fillRect(this.x, this.y, 5, 5);



				}

				update() {
					this.ttl--;
				}
			}

		},
		frame() {

			let start = performance.now();

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
			let ms = performance.now() - start;
			setTimeout(this.frame, 1000/this.framerate - ms);

		}
	}
}
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.stats {
	font-size: 1.5rem;
}

.title {
	font-weight: 700;
	font-size: 4rem;
	letter-spacing: -0.07em;
	line-height: 0;
}

.subtitle {
	font-weight: 500;
	font-size: 1.7rem;
	letter-spacing: -0.07em;
	line-height: 0;
}
</style>