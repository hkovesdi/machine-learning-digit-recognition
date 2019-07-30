<template>
  <div class="container">
    <div class="row">
      <div class="d-flex justify-content-center">
        <transition name="fade"><h2 v-if="predictedNumber !=='-'">I think you drew {{predictedNumber==8 ? "an":"a"}} {{predictedNumber}}!</h2></transition>
      </div>
    </div>
    <div class="row">
      <div>
        <VueSignaturePad
          id="signature"
          width="286px"
          height="286px"
          ref="signaturePad"
          :options="{ minWidth: 5, maxWidth:6, onEnd}"
        />
      </div>
    </div>
    <div class="row" style="margin-top:5px;">
        <button
          class="btn btn-outline-secondary"
          @click="undo"
        >
          Undo
        </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'DrawingPadComponent',
  data: function () {
    return {
      predictedNumber: '-'
    }
  },
  methods: {
    // MIT http://rem.mit-license.org
    trimCanvas(c) {
      var ctx = c.getContext('2d'),
          copy = document.createElement('canvas').getContext('2d'),
          pixels = ctx.getImageData(0, 0, c.width, c.height),
          l = pixels.data.length,
          i,
          bound = {
              top: null,
              left: null,
              right: null,
              bottom: null
          },
          x, y;
      
      // Iterate over every pixel to find the highest
      // and where it ends on every axis ()
      for (i = 0; i < l; i += 4) {
          if (pixels.data[i + 3] !== 0) {
              x = (i / 4) % c.width;
              y = ~~((i / 4) / c.width);

              if (bound.top === null) {
                  bound.top = y;
              }

              if (bound.left === null) {
                  bound.left = x;
              } else if (x < bound.left) {
                  bound.left = x;
              }

              if (bound.right === null) {
                  bound.right = x;
              } else if (bound.right < x) {
                  bound.right = x;
              }

              if (bound.bottom === null) {
                  bound.bottom = y;
              } else if (bound.bottom < y) {
                  bound.bottom = y;
              }
          }
      }
      
      // Calculate the height and width of the content
      var trimHeight = bound.bottom - bound.top,
          trimWidth = bound.right - bound.left,
          trimmed = ctx.getImageData(bound.left, bound.top, trimWidth, trimHeight);

      copy.canvas.width = trimWidth;
      copy.canvas.height = trimHeight;
      copy.putImageData(trimmed, 0, 0);

      // Return trimmed canvas
      return copy.canvas;
    },
    cloneCanvas(oldCanvas, w = oldCanvas.width, h = oldCanvas.height, startX = 0, startY = 0) {

      //create a new canvas
      var newCanvas = document.createElement('canvas');
      var context = newCanvas.getContext('2d');

      //set dimensions
      context.canvas.width = w;
      context.canvas.height = h;

      //apply the old canvas to the new one
      context.drawImage(oldCanvas, startX, startY, w, h);

      //return the new canvas
      return newCanvas;
    },
    downScaleCanvasTo20x20(canvas) {
      const context = canvas.getContext('2d');

      let canvasWidth = context.canvas.width;
      let canvasHeight = context.canvas.height;

      const downScaleRate = 20/(canvasWidth > canvasHeight ? canvasWidth : canvasHeight);
      canvasWidth *= downScaleRate;
      canvasHeight *= downScaleRate;

      const newCanvas = document.createElement('canvas');
      const newContext = newCanvas.getContext('2d');
      newContext.canvas.width = 20;
      newContext.canvas.height = 20;

      const clonedCanvas = this.cloneCanvas(canvas,canvasWidth,canvasHeight);
      //document.body.appendChild(clonedCanvas);
      newContext.drawImage(clonedCanvas,20 / 2 - clonedCanvas.width / 2, 20 / 2 - clonedCanvas.height / 2,clonedCanvas.width,clonedCanvas.height);
      //document.body.appendChild(newCanvas);
      return newCanvas;

    },
    pad20x20with8x8(oldCanvas) {
      let newCanvas = document.createElement('canvas');
      const context = newCanvas.getContext('2d');

      const oldCW = oldCanvas.getContext('2d').width;
      const oldCH = oldCanvas.getContext('2d').height;
      context.canvas.width = 28;
      context.canvas.height = 28;
      
      context.drawImage(oldCanvas,4,4,20,20);
      //document.body.appendChild(newCanvas);
      return newCanvas;
    },
    convert28x28toPixelArray(canvas) { 
      return canvas.getContext("2d").getImageData(0,0,28,28).data.filter((element,index) => { return ((index+1) % 4 == 0)});
    },
    canvasToMnist(canvas) {
      return this.convert28x28toPixelArray(this.pad20x20with8x8(this.downScaleCanvasTo20x20(this.trimCanvas(canvas))));
    },
    undo() {
      this.$refs.signaturePad.undoSignature();
    },
    onEnd() {
      const { isEmpty } = this.$refs.signaturePad.saveSignature();
      if(!isEmpty) {
        let canvas = document.getElementById('signature').querySelector('canvas');
        const imgData = this.canvasToMnist(canvas); 
        axios.post('/flask/predict', {
          number: JSON.stringify(imgData)
        })
        .then( (response) => {
          this.predictedNumber = response.data.prediction;
        })
        .catch((error) => {
          console.log(error);
        });
      }
    }
  }
};
</script>

<style>
#signature {
  border: double 3px transparent;
  border-radius: 5px;
  background-image: linear-gradient(white, white),
    radial-gradient(circle at top left, #4bc5e8, #9f6274);
  background-origin: border-box;
  background-clip: content-box, border-box;
}
.row {
  justify-content: center;
}
.fade-enter-active, .fade-leave-active {
  transition: opacity .7s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
</style>
