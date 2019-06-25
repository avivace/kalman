<template>
	<div>
		<h2 class="title">Kalman Filter 2D demo</h2>
		<h3 class="subtitle">Antonio Vivace, June 2019</h3>

		<mu-row class="stats">
			<mu-col style="padding: 15px" span="3"
				><div class="grid-cell">
					Mode:
					<mu-select
						@change="
							drawPhase = 0;
							realPoint = null;
						"
						v-model="mode"
						>
						<mu-option
							v-for="(option, index) in options"
							:key="option"
							:label="option"
							:value="index"
						></mu-option>
					</mu-select>
					<br />
					Width: {{ width }}px
					<mu-slider
						type="range"
						:min="0"
						:max="1800"
						:step="1"
						value="75"
						class="slider"
						v-model="width"
					/>
					Height: {{ height }}px
					<mu-slider
						type="range"
						:min="0"
						:max="1000"
						:step="1"
						value="75"
						class="slider"
						v-model="height"
					/>

					Noise covariance: {{ sigma }}
					<mu-slider
						:min="0"
						:max="width / 3"
						:step="1"
						class="demo-slider"
						v-model="sigma"
					></mu-slider>
					<h4>Stats</h4>
					{{ status }}, <code>{{ states.length }}</code> states drawn.
					<code>{{ Math.round(ms) }}</code> ms per frame
				</div></mu-col
			>
			<mu-col span="9"
				><div class="grid-cell">
					<canvas
						class="kalmandemo"
						@mouseleave="mouseLeave"
						@pointermove="mouseOver"
						ref="ccont"
						:width="width"
						:height="height"
					></canvas></div
			></mu-col>
		</mu-row>

		<br /><br />

		<br /><br /><br />

		<span class="stats"> </span>
		<small>
			<p>
				<b
					><a href="https://github.com/avivace/kalman"
						>Source code</a
					></b
				>
			</p>
		</small>
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
		last: null,
		maxNoise: 75,
		ms: 0
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

			this.lastPoint = v([0, 0, 0, 0]);

			this.last = m([
				[0, 0, 0, 0],
				[0, 0, 0, 0],
				[0, 0, 0, 0],
				[0, 0, 0, 0]
			]);

			var c = this.$refs.ccont;
			this.ctx = c.getContext("2d", { alpha: false });
			this.ctx.fillStyle = "#37474f";
			this.ctx.fillRect(0, 0, this.width, this.height);

			this.State = class State {
				constructor(realInput, noisyInput, kalmanPoint) {
					this.realInput = realInput;
					this.noisyInput = noisyInput;
					this.kalmanPoint = kalmanPoint;
					this.dead = false;
				}

				display() {
					this.realInput.display("#0d47a1");
					this.kalmanPoint.display("#dd2c00");
					this.noisyInput.ttl;
					this.noisyInput.display("#388e3c");
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
					this.ttl = 200;
					this.ctx = ctx;
				}

				display(color) {
					// Map remaining TTL to 0-255, convert it to two hex digits
					//  and use it as Alpha channel (ttl -> 0, alpha -> 1)

					let alpha = Math.round((this.ttl * 255) / 200).toString(16);
					this.ctx.beginPath();
					this.ctx.arc(this.x, this.y, 2, 0, 2 * Math.PI);
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
			if (this.status == "running") {
				let v = window.$V;

				let ctx = this.ctx;
				this.ctx.fillStyle = "#263238";
				this.ctx.fillRect(0, 0, this.width, this.height);

				let maxNoise = this.maxNoise;

				// Real point
				let a = new this.Point(this.clientX - 3, this.clientY - 3, ctx);
				// Add noise to the clean input
				let noisyX = Math.round(
					this.clientX + this.getRandomInt(maxNoise) - maxNoise / 2
				);
				let noisyY = Math.round(
					this.clientY + this.getRandomInt(maxNoise) - maxNoise / 2
				);

				let n = new this.Point(noisyX, noisyY, ctx);

				/* 

				KALMAN FILTER implementation

				We ignore the control vector (c) and the Input Control Matrix (B)

				*/

				// m = [noisyX, noisyY, deltaX, deltaY]
				let deltaX = noisyX - this.lastPoint.elements[0];
				let deltaY = noisyY - this.lastPoint.elements[1];
				let measurement = v([noisyX, noisyY, deltaX, deltaY]);

				// PREDICTION step
				// x = (A * x) + (B * c)
				var x = this.A.multiply(this.lastPoint);
				// P = (A * P * AT) + Q
				var P = this.A.multiply(this.last)
					.multiply(this.A.transpose())
					.add(this.Q);

				// CORRECTION step
				// S = (H * P * HT) + R
				var S = this.H.multiply(P)
					.multiply(this.H.transpose())
					.add(this.R);
				// K = P * HT * S-1
				var K = P.multiply(this.H.transpose()).multiply(S.inverse());
				// y = m - (H * x)
				var y = measurement.subtract(this.H.multiply(x));

				// x = x + (K * y)
				//  this is the final filtered point for this iteration
				this.lastPoint = x.add(K.multiply(y));
				// P = (I - (K * H)) * P
				this.last = window.Matrix.I(4)
					.subtract(K.multiply(this.H))
					.multiply(P);

				// ---

				let k = new this.Point(
					this.lastPoint.elements[0],
					this.lastPoint.elements[1],
					ctx
				);

				// Push the final state (real, noisy, filtered)
				this.states.push(new this.State(a, n, k));

				// Draw every state in the stack, and kill the older ones
				for (let i = this.states.length - 1; i > 0; --i) {
					let state = this.states[i];
					state.display();
					state.update();
					if (state.dead) {
						this.states.splice(i, 1);
					}
				}
			}
			// See ya in 1000/desiredFramerate milliseconds
			this.lastCalledTime = performance.now();
			let ms = performance.now() - start;
			this.ms = ms;
			setTimeout(this.frame, 1000 / this.framerate - ms);
		}
	}
};
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
a {
	text-decoration: none;
}
.stats {
	font-size: 1.2rem;
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

.kalmandemo {
	box-shadow: 0 27.125px 50px -12.125px rgba(0, 0, 0, 0.4);
}
</style>
