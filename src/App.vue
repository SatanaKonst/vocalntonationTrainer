<template>
  <div class="container p-2">

    <div class="row mb-4">
      <div class="col-12 text-center">
        <h2>Тюнер</h2>
      </div>
      <div class="col-12">
        <guitar-tuner @note="tunerNote=$event"></guitar-tuner>
      </div>
    </div>

    <div class="row mb-2">
      <div class="col-12 text-center">
        <div class="alert alert-info">
          После запуска вы услышите звуковой сигнал начала тренировки. <br>
          Далее будет воспроизведена нота которую нужно интонировать! <br>
          Для прохождения задания продолжительность попадания в ноту должна составить 3 секунды!
        </div>
      </div>
      <div class="col-12 text-center mb-3">
        <button type="button" class="btn btn-success" @click="noteWin=true">
          Начать тренировку
        </button>
      </div>
    </div>

    <div class="row mb-5">
      <div class="col-4">
        Нота которую надо интонировать: {{ randomNote }}
      </div>
      <div class="col-8">
        <div class="progress">
          <div class="progress-bar" role="progressbar"
               :style="'width: '+intonationPause/100*intonationProgress+'%;'"
               :aria-valuenow="intonationProgress"
               aria-valuemin="0"
               :aria-valuemax="intonationPause">
            {{ intonationProgress }}
          </div>
        </div>
      </div>
    </div>


    <div class="row">
      <play-notes @randomNote="randomNote=$event" :play-random-note="noteWin"></play-notes>
    </div>

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
