<template>
  <div id="app">
    <div class="model__container">
      <div class="img__container">
        <img id="sample-img__img" src="./assets/sample_kite.jpg"/>
        <canvas id="sample-img__canvas"></canvas>
      </div>
      <button type="button" v-on:click="analyzeSample()" style="width: 90px">Run sample</button>
    </div>
    
    <div class="model__container">
      <div class="img__container">
        <img id="full-img__img" src="./assets/full_image.jpg"/>
        <canvas id="full-img__canvas"></canvas>
      </div>
      <button type="button" v-on:click="runModel('full-img__img', 'full-img__canvas', false)" style="width: 80px">Run full</button>
    </div>
  </div>
</template>

<!-- Load TensorFlow.js. This is required to use coco-ssd model. -->
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"> </script>
<!-- Load the coco-ssd model. -->
<script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd"> </script>

<script>
import P5 from 'p5';

// Notice there is no 'import' statement for tensorflow or coco-ssd. 'cocoSsd' and 'tf' is
// available on the index-page because of the script tag above.

export default {
  name: 'App',
  data() {
    return {
      sampleClass: ''
    }
  },
  mounted() {},
  methods: {
    async analyzeSample() {
      const vm = this;
      // this.runModel('sample-img__img', 'sample-img__canvas', true).then(cls => {
      vm.sampleClass = await this.runModel('sample-img__img', 'sample-img__canvas', true);
    },
    runModel(imgId, canvasId, isSample) {
      console.log('Running model...');
      
      return new Promise(resolve => {

        const img = document.getElementById(imgId);
        const c = document.getElementById(canvasId);
        const context = c.getContext('2d');
        c.width = img.clientWidth;
        c.height = img.clientHeight;
        // context.drawImage(image, 0, 0);
        context.font = '10px Arial';

        // Load the model.
        cocoSsd.load().then(model => {
          // detect objects in the image.
          model.detect(img).then(predictions => {
            // Returns an array of classes and probabilities that looks like:
            // [{
            //   bbox: [x, y, width, height],
            //   class: "person",
            //   score: 0.8380282521247864
            // }, {...}]
            console.log('Predictions: ', predictions);
            this.drawBoxes(context, predictions, isSample);
            if (isSample) resolve(predictions[0].class);
            // else { resolve(predictions); }
          });
        });
      })
    },
    drawBoxes(context, predictions, isSample) {
      for (let i = 0; i < predictions.length; i++) {
        if (isSample || predictions[i].class == this.sampleClass) {
          context.beginPath();
          context.rect(...predictions[i].bbox);
          context.lineWidth = 2;
          context.strokeStyle = 'green';
          context.fillStyle = 'green';
          context.stroke();
          context.fillText(
            predictions[i].score.toFixed(3) + ' ' + predictions[i].class, predictions[i].bbox[0],
            predictions[i].bbox[1] > 10 ? predictions[i].bbox[1] - 5 : 10);
        }
      }
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  display: flex;
  flex-direction: row;
  justify-content: space-evenly;
}

.model__container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.img__container {
  position: relative;
  display: inline-block;
}

#sample-img__canvas,
#full-img__canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>
