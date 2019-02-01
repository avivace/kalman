<template>
	<div >
	<h2>kalman</h2>
	<canvas @pointermove="mouseOver" ref="ccont" width="600" height="600"></canvas>
	</div>
</template>

<script>
export default {
	data: () =>  ({
		a: null,
		ctx: null
	}),
	mounted () {

		this.init();
		this.frame();

	},
	methods: {
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
			let ctx = this.ctx
			ctx.fillStyle = 'blue';
			ctx.fillRect(0, 0, 600, 600);
			

			ctx.fillStyle = 'red';
			ctx.fillRect(this.clientX, this.clientY,50,50);


			setTimeout(this.frame, 1000/60);
		}
	}
}


</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
