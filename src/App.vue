<template>
  <div>
    <button type="button" @click="noteWin=true">Start train</button>
    <p>
      Play note: {{ randomNote }} <br>
      <progress :max="intonationPause" :value="intonationProgress"></progress>
    </p>
    <guitar-tuner @note="tunerNote=$event"></guitar-tuner>
    <play-notes @randomNote="randomNote=$event" :play-random-note="noteWin"></play-notes>
  </div>
</template>

<script>

import PlayNotes from "@/components/playNotes.vue";
import GuitarTuner from "@/components/guitarTuner.vue";


export default {
  name: 'App',
  components: {
    PlayNotes,
    GuitarTuner
  },
  data() {
    return {
      tunerNote: null,
      randomNote: null,
      noteWin: false,
      intonationPause: 3000,
      intonationTimer: null,
      intonationLastTimer: null,
      intonationProgress: 0
    }
  },
  watch: {
    randomNote() {
      this.noteWin = false;
    },
    tunerNote() {
      if (this.tunerNote === this.randomNote) {
        this.intonationTimer = setTimeout(() => {
          this.noteWin = true;
        }, this.intonationPause);

        this.intonationLastTimer = setInterval(() => {
          this.intonationProgress += 200;
        }, 200);
      } else {
        clearTimeout(this.intonationTimer);
        clearInterval(this.intonationLastTimer);
        this.intonationProgress = 0;
      }
    }
  }
}
</script>

<style>
#app {

}
</style>
