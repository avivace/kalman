<template>
	<div >
	<h2>kalman</h2>
	<canvas @mouseleave="mouseLeave" @pointermove="mouseOver" ref="ccont" :width="width" :height="height"></canvas>
	<br><br><br>
	{{ Number((fps).toFixed(0)) }} FPS
  {{ status }}
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
    status: "paused"
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
		},
		frame() {
			let ctx = this.ctx

			ctx.fillStyle = 'red';
			ctx.fillRect(this.clientX, this.clientY,50,50);

			ctx.fillStyle = 'green';
			ctx.fillRect(this.getRandomInt(600), this.getRandomInt(600),50,50);

			

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
