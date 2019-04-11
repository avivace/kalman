<template>
	<div>
		<h2 class="title">Kalman Filter 2D demo</h2>
		<h3 class="subtitle">Antonio Vivace, May 2019</h3>
		<canvas
			@mouseleave="mouseLeave"
			@pointermove="mouseOver"
			ref="ccont"
			:width="width"
			:height="height"
		></canvas>
		<br /><br /><br />
		
		<span class="stats">
			{{ status }}, <code>{{ states.length }}</code> states drawn
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
		states: [],
		framerate: 60,
		lastPoint: null,
		A: null,
		B: null,
		H: null,
		Q: null,
		R: null,
		last: null
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

			// State Transition
			this.A = m([
				[1, 0, 0.2, 0],
				[0, 1, 0, 0.2],
				[0, 0, 1, 0],
				[0, 0, 0, 1]
			]);

			// Input Control Matrix is ignored

			this.H = m([
				[1, 0, 0, 0],
				[0, 1, 0, 0],
				[0, 0, 1, 0],
				[0, 0, 0, 1]
			]);

			this.Q = m([
				[0.001, 0, 0, 0],
				[0, 0.001, 0, 0],
				[0, 0, 0, 0],
				[0, 0, 0, 0]
			]);

			this.R = m([
				[0.1, 0, 0, 0],
				[0, 0.1, 0, 0],
				[0, 0, 0.1, 0],
				[0, 0, 0, 0.1]
			]);

			this.lastPoint = v([0, 0, 0, 0])

			this.last = m([
				[0, 0, 0, 0],
				[0, 0, 0, 0],
				[0, 0, 0, 0],
				[0, 0, 0, 0]
			]);

			var c = this.$refs.ccont;
			this.ctx = c.getContext("2d", { alpha: false });
			this.ctx.fillStyle = "#37474f";
			this.ctx.fillRect(0, 0, 600, 600);

			this.State = class State {
				constructor(realInput, noisyInput, kalmanPoint) {
					this.realInput = realInput;
					this.noisyInput = noisyInput;
					this.kalmanPoint = kalmanPoint;
					this.dead = false;
				}

				display() {
					this.realInput.display("#0000ff");
					this.kalmanPoint.display("#ff0000");
					this.noisyInput.ttl;
					this.noisyInput.display("#558b2f");
				}

				update() {
					this.realInput.update();
					this.noisyInput.update();
					this.kalmanPoint.update();
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

				display(color) {
					// Map remaining TTL to 0-255, convert it to two hex digits
					//  and use it as Alpha channel (ttl -> 0, alpha -> 1)

					let alpha = Math.round(this.ttl * 255/520).toString(16)
					this.ctx.beginPath();
					this.ctx.arc(this.x, this.y, 2, 0, 2*Math.PI)
					this.ctx.fillStyle = color + alpha;
					this.ctx.fill();
					//this.ctx.fillRect(this.x, this.y, 4, 4);
				}

				update() {
					this.ttl--;
				}
			};
		},
		frame() {
			let start = performance.now();
			if (this.status == "running"){
				let v = window.$V;
				
				

				let ctx = this.ctx;
				this.ctx.fillStyle = "#000f12";
				this.ctx.fillRect(0, 0, 600, 600);

				let maxNoise = 75

				// Real point
				let a = new this.Point(this.clientX - 3, this.clientY - 3, ctx);
				// Noisy input
				let noisyX = Math.round(this.clientX + this.getRandomInt(maxNoise) - maxNoise/2)
				let noisyY = Math.round(this.clientY + this.getRandomInt(maxNoise) - maxNoise/2)
				let n = new this.Point(noisyX, noisyY, ctx)

				let deltaX = noisyX - this.lastPoint.elements[0];
				let deltaY = noisyY - this.lastPoint.elements[1];

				/*
				
				Kalman things

				m = [noisyX, noisyY, deltaX, deltaY]

				Prediction Step:
				x = (A * x) + (B * c)
				P = (A * P * AT) + Q

				Correction Step:
				S = (H * P * HT) + R 
				K = P * HT * S-1
				y = m - (H * x)
				x = x + (K * y)
				P = (I - (K * H)) * P
				
				We ignore the control vector (c) and the Input Control Matrix (B)

				*/

				let measurement = v([noisyX, noisyY, deltaX, deltaY]);
				

				// PREDICTION step
				var x = this.A.multiply(this.lastPoint)
				var P = ((this.A.multiply(this.last)).multiply(this.A.transpose())).add(this.Q); 

				// CORRECTION step
				var S = ((this.H.multiply(P)).multiply(this.H.transpose())).add(this.R);
				var K = (P.multiply(this.H.transpose())).multiply(S.inverse());
				var y = measurement.subtract(this.H.multiply(x));

				this.lastPoint = x.add(K.multiply(y));
				this.last = ((window.Matrix.I(4)).subtract(K.multiply(this.H))).multiply(P);

				// ---


				let k = new this.Point(this.lastPoint.elements[0], this.lastPoint.elements[1], ctx)

				this.states.push(new this.State(a, n, k));

				this.states.forEach(
					function(state, i) {
						state.display();
						state.update();
						if (state.dead) {
							this.states.splice(i, 1);
						}
					}.bind(this)
				);

				

				// See ya in 1000/desiredFramerate milliseconds

			}
			this.lastCalledTime = performance.now();
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
