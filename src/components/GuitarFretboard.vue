<template>
  <div class="fretboard-container">
    <table class="info-table">
      <tr>
        <td>Root: {{ rootNote }}, Scale: {{ scaleType }}</td>
        <td>Score: {{ score }}</td>
      </tr>
    </table>
    <button @click="startGame">Start Game</button>
    <div v-if="gameOver" class="game-over">
      <p>Game Over! Your score: {{ score }}</p>
      <button @click="startGame">Restart</button>
    </div>

    <div class="fretboard" >
      <table>
        <thead>
          <tr>
            <th v-for="(fret, index) in frets" :key="'fret-num-' + fret" :class="{ 'first-column': index === 0 }">
              {{ fret }}
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="string in strings" :key="string">
            <td v-for="(fret, index) in frets" :key="string + fret" :class="{ 'first-column': index === 0, 'clicked': (clickedCells[string + fret] ||  isScale(string,fret) && gameOver) }" @click="checkNote(string, fret)">
              <span v-if="clickedCells[string + fret]">{{ getSoundName(string, fret) }}</span>
              <span v-else-if="gameOver">{{ getSoundName(string, fret) }}</span>
            </td>
          </tr>
        </tbody>
      </table>

    </div>
  </div>
</template>

<script>
import { defineComponent } from 'vue';

export default defineComponent({
  name: 'GuitarFretboard',
  data() {
    return {
      strings: ['E', 'A', 'D', 'G', 'B', 'e'],
      frets: Array.from({ length: 13 }, (_, i) => i),
      notes: ['C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#', 'A', 'A#', 'B'],
      rootNote: '',
      scaleType: '',
      majorScaleIntervals: [0, 2, 4, 5, 7, 9, 11],
      minorScaleIntervals: [0, 2, 3, 5, 7, 8, 10],
      clickedCells: {},
      score: 0,
      showFailMessage: false,
      gameOver: false,
    };
  },
  methods: {
    startGame() {
      this.rootNote = this.notes[Math.floor(Math.random() * this.notes.length)];
      this.scaleType = ['major', 'minor'][Math.floor(Math.random() * 2)];
      this.clickedCells = {};
      this.score = this.getScore();
      this.showFailMessage = false;
      this.gameOver = false;
    },
    getSoundName(string, fret) {
      const openStringNoteIndex = this.notes.indexOf(string);
      const noteIndex = (openStringNoteIndex + fret) % 12;
      return this.notes[noteIndex];
    },
    isScale(string, fret) {
      const note = this.getSoundName(string, fret);
      const rootNoteIndex = this.notes.indexOf(this.rootNote);
      const scaleIntervals = this.scaleType === 'major' ? this.majorScaleIntervals : this.minorScaleIntervals;
      const scaleNotes = scaleIntervals.map(interval => this.notes[(rootNoteIndex + interval) % 12]);
        return scaleNotes.includes(note);
    },
    checkNote(string, fret) {
      if (this.gameOver) return;
      if (this.isScale(string, fret)) {
        this.score++;
        this.clickedCells[string + fret] = true;
        this.showFailMessage = false;
      } else {
        this.showFailMessage = true;
        this.gameOver = true;
        this.saveScore(this.score)
      }
    },
  // Save score to cookies
  saveScore(score) {
    const d = new Date();
    d.setTime(d.getTime() + (7*24*60*60*1000)); // Expires in 7 days
    let expires = "expires="+ d.toUTCString();
    document.cookie = "score=" + score + ";" + expires + ";path=/";
  },

  // Get score from cookies
  getScore() {
    let name = "score=";
    let decodedCookie = decodeURIComponent(document.cookie);
    let ca = decodedCookie.split(';');
    for(let i = 0; i <ca.length; i++) {
      let c = ca[i];
      while (c.charAt(0) == ' ') {
        c = c.substring(1);
      }
      if (c.indexOf(name) == 0) {
        return c.substring(name.length, c.length);
      }
    }
    return "";
  }
  },
});
</script>

<style scoped>
.fretboard-container {
  margin-bottom: 20px;
}
.info-table,
.fretboard table {
  width: 100%;
  border-collapse: collapse;
}
.info-table td,
.fretboard th,
.fretboard td {
  border: 1px solid #000;
  text-align: center;
  padding: 10px;
}
.clicked {
  background-color: red;
}
.game-over {
  margin-top: 20px;
}
</style>
