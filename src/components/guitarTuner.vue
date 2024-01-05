<template>

  <div class="row">
    <div class="col-3">Частота: </div>
    <div class="col-9">
      <div class="progress">
        <div class="progress-bar" role="progressbar" :style="'width: '+tuneFrequency/10+'%;'" :aria-valuenow="tuneFrequency" aria-valuemin="0" aria-valuemax="100">{{tuneFrequency}} Hz</div>
      </div>
    </div>
  </div>
  <div class="row">
    <div class="col-3">Нота: </div>
    <div class="col-9">
      <h3>{{ getNoteFromFrequency() }}</h3>
    </div>
  </div>


</template>

<script>
import * as Tone from 'tone'

export default {
  name: 'guitarTuner',
  components: {},
  data() {
    return {
      tuneFrequency: null,
    };
  },
  mounted() {
    this.tune()
  },
  watch: {
    tuneFrequency() {
      this.$emit('note', this.getNoteFromFrequency());
    }
  },
  methods: {
    tune() {
      navigator.mediaDevices.getUserMedia(
          {
            "audio": {
              "mandatory": {
                "googEchoCancellation": "false",
                "googAutoGainControl": "false",
                "googNoiseSuppression": "false",
                "googHighpassFilter": "false"
              },
              "optional": []
            },
          }).then((stream) => {

        let audioContext = new AudioContext();
        const gainNode = audioContext.createGain();
        // gainNode.connect(audioContext.destination);
        let buf = new Float32Array(2048);
        // Create an AudioNode from the stream.
        let mediaStreamSource = audioContext.createMediaStreamSource(stream);

        mediaStreamSource.connect(gainNode);


        //увеличиваем гейн
        gainNode.gain.value = 200;

        // Connect it to the destination.
        let analyser = audioContext.createAnalyser();
        analyser.fftSize = 2048;
        mediaStreamSource.connect(analyser);
        setInterval(() => {
          analyser.getFloatTimeDomainData(buf);
          this.tuneFrequency = Math.round(this.autoCorrelate(buf, audioContext.sampleRate));
        }, 10);
      }).catch((err) => {
        // always check for errors at the end.
        console.error(`${err.name}: ${err.message}`);
        alert('Stream generation failed.');
      });
    },
    autoCorrelate(buf, sampleRate) {
      // Implements the ACF2+ algorithm
      let SIZE = buf.length;
      let rms = 0;

      for (let i = 0; i < SIZE; i++) {
        let val = buf[i];
        rms += val * val;
      }
      rms = Math.sqrt(rms / SIZE);
      if (rms < 0.01) // not enough signal
        return -1;

      let r1 = 0, r2 = SIZE - 1, thres = 0.2;
      for (let i = 0; i < SIZE / 2; i++)
        if (Math.abs(buf[i]) < thres) {
          r1 = i;
          break;
        }
      for (let i = 1; i < SIZE / 2; i++)
        if (Math.abs(buf[SIZE - i]) < thres) {
          r2 = SIZE - i;
          break;
        }

      buf = buf.slice(r1, r2);
      SIZE = buf.length;

      let c = new Array(SIZE).fill(0);
      for (let i = 0; i < SIZE; i++)
        for (let j = 0; j < SIZE - i; j++)
          c[i] = c[i] + buf[j] * buf[j + i];

      let d = 0;
      while (c[d] > c[d + 1]) d++;
      let maxval = -1, maxpos = -1;
      for (let i = d; i < SIZE; i++) {
        if (c[i] > maxval) {
          maxval = c[i];
          maxpos = i;
        }
      }
      let T0 = maxpos;

      let x1 = c[T0 - 1], x2 = c[T0], x3 = c[T0 + 1];
      let a = (x1 + x3 - 2 * x2) / 2;
      let b = (x3 - x1) / 2;
      if (a) T0 = T0 - b / (2 * a);

      return sampleRate / T0;
    },
    getNoteFromFrequency() {
      return Tone.Frequency(this.tuneFrequency).toNote()
    }
  },
}

</script>


<style scoped>

</style>