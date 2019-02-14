<template>
  <div id="app">
    <video 
      :width="width"
      :height="height"
      autoplay />
    <v-snackbar
      v-model="snackbar"
      :timeout=3000
      color="cyan"
    >
      {{ qr_code }}
      <v-btn
        dark
        flat
        large
        @click="snackbar = false"
      >
        ×
      </v-btn>
    </v-snackbar>
  </div>
</template>

<script>
import jsQR from 'jsqr'

export default {
  name: 'app',
  data() {
    return {
      srcObject: "",
      width: 640,
      height: 480,
      qr_code: "",
      snackbar: false,
      error: "",
    }
  },
  created: function() {
    // カメラの起動
    const p = navigator.mediaDevices.getUserMedia({
      audio: false,
      video: {
        width: this.width,
        height: this.height,
        frameRate: { ideal: 5, max: 15 },
        facingMode: {
          exact: "environment"
        }
      }
    });
    p.then(function(mediaStream) {
      document.querySelector("video").srcObject = mediaStream;
    }).catch(function(err) {
      alert(err.name + ": " + err.message);
    });
  },
  mounted: function() {
    // ビデオをキャンパスにコピーしキャンパスのデータをjsQRで処理
    const video = document.querySelector("video");
    const canv = document.createElement("canvas");
    canv.height = video.height;
    canv.width = video.width;

    const context = canv.getContext("2d");

    setInterval(() => {
      console.log("search .....");
      context.drawImage(video, 0, 0, this.width, this.height);
      const imageData = context.getImageData(0, 0, this.width, this.height);
      const code = jsQR(imageData.data, imageData.width, imageData.height);
      if (code) {
        console.log("Found QR code", code, code.data);
        this.qr_code = code.data;
        if (!this.snackbar) {
          this.snackbar = true;
        }
      }
    }, 500);
  },
}
</script>

<style>
html {
  margin: 0;
  padding: 0;
}
video {
  width: 100%;
  height: 100%;
  position: absolute;
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>
