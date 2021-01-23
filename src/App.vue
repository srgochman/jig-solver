<template>
  <div id="app">
    <div class="model__container">
      <h1>1. Upload your full puzzle:</h1>
      <input id="select-full" type="file" accept="image/png, image/jpeg, image/tiff" onchange="fullImgSrc = window.URL.createObjectURL(this.files[0])">
      <img id="full-img__img"/>
      <h1>and your puzzle piece:</h1>
      <input id="select-sample" type="file" accept="image/png, image/jpeg, image/tiff" onchange="document.getElementById('sample-img__img').src = window.URL.createObjectURL(this.files[0])">
      <div class="img__container">
        <img id="sample-img__img"/>
        <canvas id="sample-img__canvas"></canvas>
      </div>
    </div>

    <div class="model__container">
      <h1>2. Run the analysis:</h1>
      <button type="button" v-on:click="analyzeSample()">Find it!</button>
    </div>
    
    <div class="model__container model__container--full-img">
      <h1>3. Here's where the piece might belong:</h1>
      <div class="img__container">
        <img class="full-img__img"/>
        <canvas id="full-img__canvas"></canvas>
      </div>
    </div>
    <button @click="checkFull"></button>
  </div>
</template>

<!-- Load TensorFlow.js. This is required to use coco-ssd model. -->
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"> </script>
<!-- Load the coco-ssd model. -->
<script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd"> </script>

<script>
// Notice there is no 'import' statement for tensorflow or coco-ssd. 'cocoSsd' and 'tf' is
// available on the index-page because of the script tag above.

export default {
  name: 'App',
  data() {
    return {
      sampleImgSrc: '/assets/sample_kite.jpg',
      sampleClass: '',
      fullImgSrc: '/assets/full_image.jpg'
    }
  },
  mounted() {},
  // computed: {
  //   sampleImgScr() {
  //     return '/assets/sample_kite.jpg';
  //   }
  // },
  methods: {
    checkFull() {
      console.log(this.fullImgSrc);
    },
    selectInput(input) {
      // document.getElementById('full-img__img').src = window.URL.createObjectURL(input.files[0])
      console.log(input);
    },
    readURL(input) {
      console.log(input);
      const vm = this;
      if (input.files && input.files[0]) {
        var reader = new FileReader();
        reader.onload = function () {
          vm.sampleImgSrc = reader.result;
          document.getElementById('sample-img__img')
          // $('#blah')
          //   .attr('src', e.target.result)
          //   .width(150)
          //   .height(200);
        };

        reader.readAsDataURL(input.files[0]);
      }
    },
    async analyzeSample() {
      const vm = this;
      vm.sampleClass = await this.runModel('sample-img__img', 'sample-img__canvas', true);  // run model on sample to retrieve object class
      await this.runModel('full-img__img', 'full-img__canvas', false);  // run model on full image to identify locations of same object class
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
html {
  /* background-color: rgb(192, 217, 241); */
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  display: flex;
  /* flex-direction: column; */
  /* align-items: center; */
  flex-direction: row;
  justify-content: space-around;
}

.model__container {
  display: flex;
  /* flex-direction: row;
  justify-content: flex-start; */
  align-items: center;
  /* width: 1024px; */
  width: 700px;
  /* margin-bottom: 50px; */
  flex-direction: column;
  border: 1px solid #2c3e50;
  border-radius: 25px;
}

/* .model__container.model__container--full-img {
  flex-direction: column;
  align-items: flex-start;
} */

.img__container {
  position: relative;
  display: inline-block;
  width: min-content;
}

#sample-img__canvas,
#full-img__canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

button {
  width: min-content;
}
</style>
