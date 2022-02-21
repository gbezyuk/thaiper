<template>
  <div
    class="word-details"
  >
    <h1 @click="play">{{ word }}</h1>
    <h2>{{ translation }}</h2>
    <audio v-if="withAudio" ref="audio" :src="`/audio-samples/words/${word}.mp3`" :autoplay="autoplay"/>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import words from '../../../../data/most-frequent-4000-words.json'

export default defineComponent({
  props: ["word", "withAudio", "autoplay"],
  // mounted () {
  //   if (this.withAudio && this.autoplay) {
  //     setTimeout(this.play, 0)
  //   }
  // },
  computed: {
    translation () {
      return words.find(w => w.thai === this.word)?.english || ''
    }
  },
  methods: {
    play() {
      this.$refs.audio.play();
    },
  },
});
</script>

<style lang="stylus" scoped>
.word-details
  max-width 200px
  h1
    cursor pointer
    margin 0
    padding 0 .1em
    font-size 10em
    color black
    min-width 1em
    text-align center
  h2
    cursor pointer
    margin 0
    padding 0 .1em
    font-size 4em
    color darkgray
    min-width 1em
    text-align center
</style>
