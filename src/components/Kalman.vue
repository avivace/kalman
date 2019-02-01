<template>
  <div>
    <p>Canvas</p>


    <canvas ref="cont" width="500" height="500"></canvas>
  </div>
</template>

<script>
export default {
  data: function () {
    return {
      a: null
    }
  },
  mounted () {
    var c = this.$refs.cont;
    var ctx = c.getContext("2d");
    ctx.fillStyle = 'black';
    ctx.fillRect(0,0,500, 500);
    ctx.fillStyle = 'white';
    ctx.moveTo(0, 0);
    ctx.lineTo(500, 100);
    ctx.stroke();

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
