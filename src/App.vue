<template>
  <div id="app">
    <guitar-fretboard></guitar-fretboard>
  </div>
</template>

<script>
import { defineComponent } from 'vue';
import GuitarFretboard from './components/GuitarFretboard.vue';

export default defineComponent({
  name: 'App',
  components: {
    GuitarFretboard
  }
});
</script>
