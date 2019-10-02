<template>
  <div class="player">
    <audio v-bind:src="streamSource" id="stream" ref="stream">
    </audio>
    <canvas ref="oscilloscope"></canvas>
    <div class="playerControls" ref="playerControls">
      <div class="playPauseControl">
        <button v-on:click="playPause()" ref="playPause">
          <i ref="playPauseIcon" class="material-icons" style="font-size: 10vh; color: white;">play_arrow</i>
        </button>
      </div>
      <div class="songInfoControl">
        <div class="volume-container">
          <div class="volume-icon">
            <i class="material-icons" style="color: white;">volume_mute</i>
          </div>
          <div class="volume">
            <vue-slider ref="volume" v-model="value" v-on:change="setVolume(value)" v-bind:min="volumeMin" v-bind:max="volumeMax"/>
          </div>
          <div class="volume-icon">
            <i class="material-icons" style="color: white;">volume_up</i>
          </div>
        </div>
        <div ref="songTitle" class="songTitle">
        </div>
        <div ref="artistName" class="artistName">
        </div>
      </div>
      <div class="skipControl">
        <button v-on:click="skip()" ref="playPause">
          <i class="material-icons" style="font-size: 10vh; color: white;">fast_forward</i>
        </button>
      </div>
    </div>
  </div>
</template>

<script>

import VueSlider from 'vue-slider-component';
import 'vue-slider-component/theme/material.css';

export default {
  name: 'Player',
  components: {
    VueSlider
  },
  props: {
    streamSource: String,
    streamMimeType: String,
    volumeMax: 100,
    volumeMin: 0
  },
  data: function() {
    return {
      audioCtx: new AudioContext(),
      value: 50,
      radioState: {
        title: null,
        artist: null
      },
      checkRadioState: null
    }
  },
  mounted() {
    this.getRadioState();

    this.audioElement = this.$refs["stream"];
    this.audioElement = this.$refs["stream"];
    this.playButton = this.$refs["playPause"];
    this.volume = this.$refs["volume"];
    this.canvas = this.$refs["oscilloscope"];
    this.canvasCtx = this.canvas.getContext("2d");
    this.playButtonIcon = this.$refs["playPauseIcon"];

    this.song = this.$refs["songTitle"];
    this.artist = this.$refs["artistName"];

    this.audioElement.crossOrigin = "anonymous";

    this.source = this.audioCtx.createMediaElementSource(this.audioElement);
    this.gainNode = this.audioCtx.createGain();

    this.analyser = this.audioCtx.createAnalyser();
    this.source.connect(this.gainNode);
    this.gainNode.connect(this.analyser);
    this.gainNode.connect(this.audioCtx.destination);

    this.draw();
  },
  methods: {
    getRadioState() {
        let response = fetch("http://34.70.62.142/stream/", {method: "GET"})
            .then(response => response.json())
            .then(data => {
                this.radioState.title = data.title;
                this.radioState.artist = data.artist;

                this.setRadioLabels();
            });
    },
    setRadioLabels() {
      if (this.radioState.title && this.radioState.artist)  {
        this.song.innerHTML = this.radioState.title;
        this.artist.innerHTML = this.radioState.artist;
      }
    },
    skip() {
        fetch("http://34.70.62.142/stream/skip", {method: "GET"})
    },
    draw: function() {
      this.analyser.fftSize = 1024;
      let bufferLength = this.analyser.frequencyBinCount;
      let dataArray = new Uint8Array(bufferLength);
      let canvasCtx = this.canvasCtx;
      let canvas = this.canvas;
      let drawVisual = requestAnimationFrame(this.draw);

      // Adapt width and height to fit the entire window
      canvasCtx.canvas.width = window.innerWidth;
      canvasCtx.canvas.height = window.innerHeight;

      this.analyser.getByteTimeDomainData(dataArray);
      
      let WIDTH = canvasCtx.canvas.width;
      let HEIGHT = canvasCtx.canvas.height;

      canvasCtx.fillStyle = 'rgb(0, 0, 0)';
      canvasCtx.fillRect(0, 0, WIDTH, HEIGHT);

      canvasCtx.lineWidth = 2;
      canvasCtx.strokeStyle = 'rgb(0, 255, 0)';

      canvasCtx.beginPath();

      var sliceWidth = WIDTH * 1.0 / bufferLength;
      var x = 0;

      for(var i = 0; i < bufferLength; i++) {
  
        var v = (dataArray[i]) / (128.0);
        var y = ((v) * (HEIGHT/2));

        if(i === 0) {
          canvasCtx.moveTo(x, y);
        } else {
          canvasCtx.lineTo(x, y);
        }

        x += sliceWidth;
      }

      canvasCtx.lineTo(canvas.width, canvas.height/2);
      canvasCtx.stroke();
    },
    setVolume(value) {
      this.gainNode.gain.value = value / 100;
    },
    playPause() {
      if(this.audioCtx.state === "suspended") {
        this.audioCtx.resume();
      }

      if(!this.audioElement) {
        this.audioElement = this.$refs["stream"];
      }

      if(!this.playButton) {
        this.playButton = this.$refs["playPause"];
      }

      if(!this.volume) {
        this.volume = this.$refs["colume"];
      }

      if(this.playButton.dataset.playing === "true") {
        this.audioElement.pause();
        clearInterval(this.checkRadioState);

        this.playButton.dataset.playing = 'false';     
        this.playButtonIcon.innerHTML = "play_arrow";
      } else {
        this.audioElement.play();
        this.checkRadioState = setInterval(this.getRadioState, 3000);

        this.playButton.dataset.playing = 'true';
        this.playButtonIcon.innerHTML = "pause";
      }
    }
  }
}

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
audio {
    display: none;
}

canvas {
    width: 100%;
    height: 100%;
    left: 0;
    top: 0;
    overflow: hidden;
    position: fixed;
    z-index: 1;
}

button {
    background-color: Transparent;
    background-repeat: no-repeat;
    border: none;
    cursor:pointer;
    overflow: hidden;
    outline:none;
}

.playPauseControl {
    display: flex;
    backdrop-filter: blur(10px);
    text-align: center;
    margin-right: 5%;
}

.skipControl {
    display: flex;
    backdrop-filter: blur(10px);
    text-align: center;
    margin-left: 5%;
}

.songInfoControl {
    display: inline-block;
    backdrop-filter: blur(10px);
    text-align: left;
    white-space: nowrap;
    min-width: 80%;
    padding: 2%;
}

.volume {
    width: 80%;
}

.volume-container {
    display: flex; 
    height: 20%;
}

.volume-icon {
    width: 20%;
    text-align: center;
}

.songTitle {
    font-size: 6vh;
    height: 60%;
}

.artistName {
    height: 20%;
    font-size: 2vh;
}

.playerControls {
    display: flex;
    flex-flow: row nowrap;
    position: absolute;
    background-color: Transparent;
    color: white;
    z-index: 2;
    margin: auto;
    padding: 5%;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}
</style>
