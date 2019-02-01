<template>
	<div >
	<h2>kalman</h2>
	<canvas @pointermove="mouseOver" ref="ccont" width="600" height="600"></canvas>
	<br><br><br>
	{{ Number((fps).toFixed(0)) }} FPS
	</div>
</template>

<script>
export default {
	data: () =>  ({
		a: null,
		ctx: null,
		lastCalledTime: null,
		fps: 0,
		framecount: 0,
	}),
	mounted () {

		this.init();
		this.frame();


	},
	methods: {
		getRandomInt(max) {
			return Math.floor(Math.random() * Math.floor(max));
		},
		mouseOver(event){
			let canvas = this.$refs.ccont;
			var rect = canvas.getBoundingClientRect();

			this.clientX = event.clientX - rect.left;
			this.clientY = event.clientY - rect.top;
		},
		init() {
			var c = this.$refs.ccont;
			this.ctx = c.getContext("2d");
		},
		frame() {
			this.framecount++;
			let ctx = this.ctx
			//ctx.fillStyle = 'blue';
			//ctx.fillRect(0, 0, 600, 600);
			

			ctx.fillStyle = 'red';
			ctx.fillRect(this.clientX, this.clientY,50,50);

			ctx.fillStyle = 'green';
			ctx.fillRect(this.getRandomInt(600), this.getRandomInt(600),50,50);

			

			// Compute the real framerate
			let delta = (performance.now() - this.lastCalledTime)/1000;
			this.lastCalledTime = performance.now();

			// Update the fps counter every 5 rendered frames
			// i.e. every 1000/fps * 5 milliseconds 
			if (this.framecount % 5 == 0)
				this.fps = 1/delta;

			// Desired framerate
			setTimeout(this.frame, 1000/1000);

		}
	}
}


</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
