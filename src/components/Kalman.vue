<template>
	<div>
		<h2 class="title">Kalman Filter demo</h2>
		<h3 class="subtitle">Antonio Vivace, May 2019</h3>
		<canvas
			@mouseleave="mouseLeave"
			@pointermove="mouseOver"
			ref="ccont"
			:width="width"
			:height="height"
		></canvas>
		<br /><br /><br />
		<!--
		<span class="stats">
			{{ Number(fps.toFixed(0)) }} FPS
			{{ status }}
			{{ states.length }}
		</span>
		-->
	</div>
</template>
<script>
export default {
	data: () => ({
		// Canvas size
		width: 800,
		height: 800,
		ctx: null,
		// Performance
		lastCalledTime: null,
		fps: 0, // (Measured) framerate
		framecount: 0,
		status: "paused",
		states: [],
		framerate: 60
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
			console.log("Mouse left, stopping");
			this.status = "paused";
		},
		mouseOver(event) {
			this.status = "running";
			let canvas = this.$refs.ccont;
			var rect = canvas.getBoundingClientRect();

			this.clientX = event.clientX - rect.left;
			this.clientY = event.clientY - rect.top;
		},
		init() {
			// Sylvester is available under the window context, since we imported 
			//  it globally in the html template.
			let m = window.$M;
			let v = window.$V;

			var A = m([
				[1, 0, 0.2, 0],
				[0, 1, 0, 0.2],
				[0, 0, 1, 0],
				[0, 0, 0, 1]
			]);

			var B = m([
				[1, 0, 0, 0],
				[0, 1, 0, 0],
				[0, 0, 1, 0],
				[0, 0, 0, 1]
			]);

			var H = m([
				[1, 0, 0, 0],
				[0, 1, 0, 0],
				[0, 0, 1, 0],
				[0, 0, 0, 1]
			]);

			var Q = m([
				[0.001, 0, 0, 0],
				[0, 0.001, 0, 0],
				[0, 0, 0, 0],
				[0, 0, 0, 0]
			]);

			var R = m([
				[0.1, 0, 0, 0],
				[0, 0.1, 0, 0],
				[0, 0, 0.1, 0],
				[0, 0, 0, 0.1]
			]);

			var last_x = v([0, 0, 0, 0]);

			var last_P = m([
				[0, 0, 0, 0],
				[0, 0, 0, 0],
				[0, 0, 0, 0],
				[0, 0, 0, 0]
			]);

			var c = this.$refs.ccont;
			this.ctx = c.getContext("2d", { alpha: false });
			this.ctx.fillStyle = "#37474f";
			this.ctx.fillRect(0, 0, 800, 800);

			this.State = class State {
				constructor(realInput, noisyInput) {
					this.realInput = realInput;
					this.noisyInput = noisyInput;
					this.predictedPoint = null;
					this.dead = false;
				}

				display() {
					//this.realInput.display("blue");
					this.noisyInput.ttl;
					this.noisyInput.display("#558b2f");
				}

				update() {
					this.realInput.update();
					this.noisyInput.update();
					if (this.realInput.ttl == 0) this.dead = true;
				}
			};

			this.Point = class Point {
				constructor(x, y, ctx) {
					this.x = x;
					this.y = y;
					this.ttl = 520;
					this.ctx = ctx;
				}

				display(color, opacity) {
					// Map remaining TTL to 0-255, and use it as Alpha channel (ttl -> 0, alpha -> 1)

					let alpha = Math.round(this.ttl * 255/520).toString(16)
					
					this.ctx.fillStyle = color + alpha;
					this.ctx.fillRect(this.x, this.y, 4, 4);
				}

				update() {
					this.ttl--;
				}
			};
		},
		frame() {
			let start = performance.now();

			let ctx = this.ctx;
			this.ctx.fillStyle = "#000f12";
			this.ctx.fillRect(0, 0, 800, 800);

			let maxNoise = 75

			// Real point
			let a = new this.Point(this.clientX - 3, this.clientY - 3, ctx);
			// Noisy input
			let n = new this.Point(this.clientX + this.getRandomInt(maxNoise) - maxNoise/2,
								   this.clientY + this.getRandomInt(maxNoise) - maxNoise/2, ctx)

			this.states.push(new this.State(a, n));

			this.states.forEach(
				function(state, i) {
					//console.log(i)
					state.display();
					state.update();
					if (state.dead) {
						this.states.splice(i, 1);
					}
				}.bind(this)
			);

			this.lastCalledTime = performance.now();

			// See ya in 1000/desiredFramerate milliseconds
			let ms = performance.now() - start;
			setTimeout(this.frame, 1000 / this.framerate - ms);
		}
	}
};
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
