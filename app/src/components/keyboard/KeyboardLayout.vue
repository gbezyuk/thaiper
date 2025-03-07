<template>
  <div class="keyboard">
    <div class="row" v-for="r in rows" :key="r.id">
      <div
        class="button"
        v-for="b in r.buttons"
        :key="b"
        :class="{
          [b]: true,
          highlighted: map[b] && map[b] === highlightedKey,
          pressed: map[b] && pressedKey && map[b] === pressedKey || b === pressedKey,
        }"
      >
        <span class="mapped-key" v-if="map[!shiftOn ? b : getShifted(b)]">{{ map[!shiftOn ? b : getShifted(b)] }}</span>
        <span class="en-key">{{ !shiftOn ? b : getShifted(b) }}</span>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  props: [
    'map',
    'highlightedKey',
  ],
  emits: [
    'keyPressed',
  ],
  data () {
    return {
      pressedKey: '',
      shiftOn: false,
    }
  },
  computed: {
    universalKeys () {
      return [
        'tab', 'caps', 'lshift', 'rshift', 'del', 'return',
      ].reduce(
        (acc, key) => {
          acc[key] = key;
          return acc
        },
        {}
      )
    },
    extendedMap () {
      return {
        ...this.map,
        ...this.universalKeys,
      }
    },
    rows () {
      return [
        {
          id: 'digit-row',
          buttons: ["`", "1", "2", "3", "4", "5", "6", "7", "8", "9", "0", "-", "=", "del"],
        },
        {
          id: 'tab-row',
          buttons: ["tab", "q", "w", "e", "r", "t", "y", "u", "i", "o", "p", "[", "]", "\\"],
        },
        {
          id: 'caps-row',
          buttons: ["caps", "a", "s", "d", "f", "g", "h", "j", "k", "l", ";", "'", "return"],
        },
        {
          id: 'shift-row',
          buttons: ["lshift", "z", "x", "c", "v", "b", "n", "m", ",", ".", "/", "rshift"],
        },
      ]
    }
  },
  methods: {
    onKeyDown (e: KeyboardEvent) {
      if (e.key === 'Shift') {
        this.shiftOn = true
      }
    },
    onKeyUp (e: KeyboardEvent) {
      if (e.key === 'Shift') {
        this.shiftOn = false
      }
    },
    onKeyPress (e: KeyboardEvent) {
      if (Object.values(this.map).includes(e.key)) {
        this.pressedKey = e.key
        this.$emit('keyPressed', this.pressedKey)
      } if (Object.keys(this.map).includes(e.key)) {
        this.pressedKey = this.map[e.key]
        this.$emit('keyPressed', this.pressedKey)
      } else {
        this.pressedKey = e.key
        setTimeout(() => this.resetPressedKey(), 0)
      }
    },
    resetPressedKey () {
      this.pressedKey = ''
    },
    getShifted (b: string) {
      if (b.match(/[a-z]/i)) {
        return b.toUpperCase()
      }

      const l2u = {
        "0": "!",
        "1": "@",
        "2": "#",
        "3": "$",
        "4": "%",
        "5": "^",
        "6": "&",
        "7": "*",
        "8": "(",
        "9": ")",
        "-": "_",
        "=": "+",
        "[": "{",
        "]": "}",
        "\\": "|",
        ";": ":",
        "'": "\"",
        ",": "<",
        ".": ">",
        "/": "?"
      }

      if (b in l2u) {
        return l2u[b]
      }

      return b
    }
  },
  mounted () {
    window.addEventListener('keypress', this.onKeyPress)
    window.addEventListener('keydown', this.onKeyDown)
    window.addEventListener('keyup', this.onKeyUp)
  },
  beforeUnmount () {
    window.removeEventListener('keypress', this.onKeyPress)
    window.addEventListener('keyup', this.onKeyUp)
  }
})
</script>

<style lang="stylus" scoped>
.keyboard
  font-size 2em
  .row
    display flex
    flex-direction row
    justify-content center
    .button
      display inline-block
      box-sizing border-box
      min-width 2em
      height 2em
      border 1px solid lightgray
      border-bottom-width 3px
      border-radius .5em
      padding .2em
      margin .2em
      text-align center
      position relative
      &.tab
        margin-left 2em
      &.caps
        margin-left 2em
      &.lshift
        margin-left 1.5em
      &.highlighted
        border-bottom-width 5px
      &.pressed
        border-color blue
      .en-key
        opacity .2
      .mapped-key
        font-size 2em
        position relative
        top -.5em
      .mapped-key + .en-key
        position absolute
        bottom 0
        right 0
</style>
