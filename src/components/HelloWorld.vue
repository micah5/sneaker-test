<template>
  <div class="hello">
    <canvas ref="myCanvas" width="200" height="200"></canvas>
    <button @click="predict">ok</button>
  </div>
</template>

<script>
import * as tf from '@tensorflow/tfjs'
import nj from 'numjs'
import zeros from 'zeros'
import savePixels from 'save-pixels'
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  methods: {
    randn_bm: function(min, max, skew) {
        var u = 0, v = 0;
        while(u === 0) u = Math.random(); //Converting [0,1) to (0,1)
        while(v === 0) v = Math.random();
        let num = Math.sqrt( -2.0 * Math.log( u ) ) * Math.cos( 2.0 * Math.PI * v );

        num = num / 10.0 + 0.5; // Translate to 0 -> 1
        if (num > 1 || num < 0) num = randn_bm(min, max, skew); // resample between 0 and 1 if out of range
        num = Math.pow(num, skew); // Skew
        num *= max - min; // Stretch to fill range
        num += min; // offset to min
        return num;
    },
    predict: async function() {
      const model = await tf.loadModel('file://generator/model.json');

      const batch_size = 64

      // Generate noise
      //let noise = nj.random([batch_size, 1, 1, 100])
      let noise = nj.zeros([batch_size, 1, 1, 100])
      for (var i = 0; i < batch_size; i++) {
        for (var j = 0; j < 100; j++) {
          noise.set(i, 0, 0, j, randn_bm(-5, 5, 1))
        }
      }

      let noise_tensor = tf.tensor4d(noise.tolist())
      noise_tensor.print(true)

      // Generate images
      console.log('create images')
      let generated_images = model.predict(noise_tensor) //error here
      generated_images.print(true)

      let output_data = await generated_images.dataSync()
      let preds = Array.prototype.slice.call(output_data);

      // Save image
      let count = 0
      for (var pic_count = 0; pic_count < 1; pic_count++) {
        let x = zeros([256, 256, 3]);
        for (var i = 0; i < 256; i++) {
          for (var j = 0; j < 256; j++) {
            r = (preds[count] + 1)*127.5
            g = (preds[count+1] + 1)*127.5
            b = (preds[count+2] + 1)*127.5
            x.set(i, j, 0, r)
            x.set(i, j, 1, g)
            x.set(i, j, 2, b)
            count += 3
          }
        }
        console.log('here')
        savePixels(x, "canvas").pipe(this.$refs.myCanvas)
      }
    }
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
