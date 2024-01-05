<template>

  <div class="col-12 col-md-3 mb-3">
    Формат нот:
    <ul class="list-group mb-3">
      <li class="list-group-item">
        <input type="radio" v-model="notePianoFormat" value="null">
        Обе
      </li>
      <li class="list-group-item">
        <input type="radio" v-model="notePianoFormat" value="helmholtz">
        Нотация Гельмгольца
      </li>
      <li class="list-group-item">
        <input type="radio" v-model="notePianoFormat" value="scientific">
        Научная нотация
      </li>
    </ul>

    Показать октавы
    <ul class="list-group">
      <li class="list-group-item" v-for="(octave, octaveIndex) in octaves" :key="'showOctave'+octaveIndex">
        <input type="checkbox" v-model="showOctaves" :value="octaveIndex">
        {{ octavesTranslate[octaveIndex] }}
      </li>
    </ul>
  </div>
  <div class="col-12 col-md-9">
    <div class="row">
      <template v-for="(octave, octaveIndex) in getRenderOctaves()" :key="'octave'+octaveIndex">
        <div class="col-12 col-md-6 mb-2 ">
          <h6>{{ octavesTranslate[octaveIndex] }}</h6>
          <ul class="board pano-background">
            <li v-for="(note,indexNote) in notes.helmholtz" :key="'note'+indexNote"
                v-bind:class="getPianoBtnClass(note,indexNote)"
                :data-note="note+octave"
                @click="play(notes.scientific[indexNote]+octave)"
            >
              <span v-html="getNoteName(indexNote, octave)"></span>
            </li>
          </ul>
        </div>
      </template>
    </div>

  </div>


</template>

<script>
import * as Tone from 'tone'

export default {
  name: 'playNotes',
  props: {
    "playRandomNote": Boolean
  },
  data() {
    return {
      notePianoFormat: null,
      notes: {
        "helmholtz": ['Do', 'Do#', 'Re', 'Re#', 'Mi', 'Fa', 'Fa#', 'Sol', 'Sol#', 'La', 'La#', 'Si'],
        "scientific": ['C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#', 'A', 'A#', 'B'],
      },
      octaves: {
        "sub_contractave": 0,
        "contractave": 1,
        "major_octave": 2,
        "small_octave": 3,
        "first_octave": 4,
        "second_octave": 5,
        "third_octave": 6,
        "fourth_octave": 7,
        "fifth_octave": 8
      },
      showOctaves: [
        'small_octave',
        'first_octave',
      ],
      octavesTranslate: {
        "sub_contractave": "Субконтроктава",
        "contractave": "Контроктава",
        "major_octave": "Большая октава",
        "small_octave": "Малая октава",
        "first_octave": "Первая октава",
        "second_octave": "Вторая октава",
        "third_octave": "Третья октава",
        "fourth_octave": "Четвёртая октава",
        "fifth_octave": "Пятая октава"
      }
    }
  },
  watch: {
    playRandomNote() {
      if (this.$props.playRandomNote === true) {
        if (this.playStartMelody() === true) {
          setTimeout(() => {
            this.play(this.getRandomNote())
          }, 2000);
        }
      }
    }
  },
  methods: {
    getRenderOctaves() {
      let tmp = {};
      for (let octave in this.octaves) {
        if (this.showOctaves.indexOf(octave) !== -1) {
          tmp[octave] = this.octaves[octave];
        }
      }
      return tmp;
    },
    play(note) {
      const synth = new Tone.Synth().toDestination();
      synth.triggerAttackRelease(note, 1);
    },
    getRandomNote() {
      let note = this.notes.scientific[this.getRandomInt(0, this.notes.scientific.length - 1)];
      let renderOctaves = Object.values(this.getRenderOctaves());
      let octave = renderOctaves[this.getRandomInt(0, renderOctaves.length - 1)]
      this.$emit('randomNote', note + octave);
      return note + octave;
    },
    getRandomInt(min, max) {
      min = Math.ceil(min);
      max = Math.floor(max);
      return Math.floor(Math.random() * (max - min) + min); // The maximum is exclusive and the minimum is inclusive
    },
    playStartMelody() {
      const synth = new Tone.Synth().toDestination();
      const now = Tone.Time()
      let currentTime = 0
      let myChords = ["D4", "E4", "F4"];
      myChords.forEach(note => {
        if (note !== '') {
          synth.triggerAttackRelease(note, '8n', now + currentTime)
        }
        currentTime += Tone.Time('8n').toSeconds()
      })
      return true;
    },
    getPianoBtnClass(note, noteIndex) {
      let classes = [];
      if (noteIndex === 0) {
        classes.push('white');
      }

      if (note.indexOf('#') !== -1) {
        classes.push('black');
      } else {
        classes.push('white');
        if (this.notes.scientific[noteIndex - 1] && this.notes.scientific[noteIndex - 1].indexOf('#') !== -1) {
          classes.push('sq');
        }
      }

      return classes.join(' ');
    },
    getNoteName(indexNote, octava) {
      let science = this.notes.scientific[indexNote];
      let helmholtz = this.notes.helmholtz[indexNote];
      if (this.notePianoFormat == 'helmholtz') {
        return `${helmholtz + octava}`
      }
      if (this.notePianoFormat == 'scientific') {
        return `${science + octava}`
      }

      return helmholtz + octava + "<br>" + science + octava;
    }

  }
}
</script>


<style scoped>
.pano-background {
  background-color: palevioletred;
}

.board li {
  margin: 0;
  padding: 0;
  list-style: none;
  position: relative;
  float: left;
  cursor: pointer;
  user-select: none;
}

ul.board {
  height: 30vh;
  width: fit-content;
  padding: 10px;
  position: relative;
}

.board .white {
  height: 270px;
  width: 38px;
  z-index: 1;
  border-left: 1px solid #bbb;
  border-bottom: 1px solid #bbb;
  border-radius: 0 0 5px 5px;
  box-shadow: -1px 0 0 rgba(255, 255, 255, 0.8) inset, 0 0 5px #ccc inset,
  0 0 3px rgba(0, 0, 0, 0.2);
  background: linear-gradient(to bottom, #eee 0%, #fff 100%);
  color: black;
}

.board .white-dark {
  height: 270px;
  width: 38px;
  z-index: 1;
  border-radius: 0 0 5px 5px;
  border-top: 1px solid #777;
  border-left: 1px solid #999;
  border-bottom: 1px solid #999;
  box-shadow: 2px 0 3px rgba(0, 0, 0, 0.1) inset,
  -5px 5px 20px rgba(0, 0, 0, 0.2) inset, 0 0 3px rgba(0, 0, 0, 0.2);
  background: linear-gradient(to bottom, #fff 0%, #e9e9e9 100%);
}

.board .black {
  height: 150px;
  width: 28px;
  margin: 0 0 0 -14px;
  z-index: 2;
  border: 1px solid #000;
  border-radius: 0 0 3px 3px;
  box-shadow: -1px -1px 2px rgba(255, 255, 255, 0.2) inset,
  0 -5px 2px 3px rgba(0, 0, 0, 0.6) inset, 0 2px 4px rgba(0, 0, 0, 0.5);
  background: linear-gradient(45deg, #222 0%, #555 100%);
  color: white;
}

.board .black-light {
  height: 150px;
  width: 28px;
  margin: 0 0 0 -14px;
  z-index: 2;
  border: 1px solid #000;
  border-radius: 0 0 3px 3px;
  box-shadow: -1px -1px 2px rgba(255, 255, 255, 0.2) inset,
  0 -2px 2px 3px rgba(0, 0, 0, 0.6) inset, 0 1px 2px rgba(0, 0, 0, 0.5);
  background: linear-gradient(to right, #444 0%, #222 100%);
}

.board .sq {
  margin: 0 0 0 -14px;
}

ul.board li:first-child {
  border-radius: 5px 0 5px 5px;
}

ul.board li:last-child {
  border-radius: 0 5px 5px 5px;
}

.board li .note {
  position: relative;
  z-index: 3;
  width: 30px;
  height: 30px;
  top: 236px;
  left: 4px;
  color: white;
  font-family: "Clip";
  font-size: 25px;
  border-radius: 10px;
  opacity: 0.3;
  text-align: center;
  background: linear-gradient(
      to right,
      rgb(132, 204, 128) 0%,
      rgb(1, 174, 18) 100%
  );
}

.board li span {
  display: flex;
  justify-content: flex-end;
  width: 100%;
  height: 100%;
  flex-direction: column;
  font-size: x-small;
  align-items: center;
}

</style>