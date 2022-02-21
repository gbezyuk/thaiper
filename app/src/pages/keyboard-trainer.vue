<template>
    <TypingTask :task="typingTask" :position="typingTaskPosition"/>
    <KeyboardLayout :map="kedmanee" :highlightedKey="typingTask[typingTaskPosition]" @keyPressed="onKeyPressed" ref="keyboardLayout"/>
    <LetterDetails :letter="typingTask[typingTaskPosition]" ref="letterDetails" :withAudio="true" :autoplay="false"/>
    <WordDetails :word="nextWord" :withAudio="true" :autoplay="true" ref="wordDetails"/>
    <div class="letters">
        <span v-for="l in letters.map(l => l.letter)" :class="{ enabled: enabledLetters.map(l => l.letter).includes(l)}">{{ l }}</span>
    </div>
</template>

<script>
import wlf from '../../../data/characters.json'
import words from '../../../data/most-frequent-4000-words.json'
import { defineComponent } from 'vue'
import KeyboardLayout from '../components/keyboard/KeyboardLayout.vue'
import TypingTask from '../components/keyboard/TypingTask.vue'
import LetterDetails from '../components/keyboard/LetterDetails.vue'
import WordDetails from '../components/keyboard/WordDetails.vue'

/** TODO:
 * graphs of letter frequency vs. word frequency
 * reproduce letter frequency from the top 4000 words list (find word weights)
 * letter details: autoplay as param; keyword translation; audio for non-consonant characters
 * word panel with audio and translation; autoplay as param, too
 * allow letter choice from the UX
 * publish Thaiper as a separate project; language-specific databanks as sub-projects
**/

// const sampleText = "เมื่อ วัน ก่อน เดิน ผ่าน ร้าน ขาย เสื้อผ้า แล้ว รู้สึก อยาก ได้ กาง เกง ยีน ส์ ตัว ใหม่"

// lipilina1
// const sampleText = "ยาย ตา สา ยาม ถาม ทา ขาย นอน สอน คอย ขอ ตี มี สี ยา ปู หวี หมา หมู หมอน หนู กาว คอ ตา"
// lipilina2
// const sampleText = "กา กุน หมอ ทำ กิน บิน หา ดู สายตา หมี ทีวี ซอย ขา ขาหมู นา ใน ดี ดำ พอดี ผอม ยาว ขาว หวาน นาน ดินสอ"
// lipilina3
const sampleText = "พี่ พี่สาว นาย พ่อ ปู่ ย่า ไก่ ข่าว ไข่ ที่ ที่นี่ ที่นั่น ป่า ไม่ ว่า บ่อ ที่ไหน ไหน ทาน มา ไป ตาม หั่น ใหม่ นุ่ม หนุ่ม หนาว ตาย หนี สาว ซู หลาน"

export default defineComponent({

    mounted () {
        setTimeout(() => {
            this.nextWord = this._getNextWord()
        }, 0)
    },
    //     const n = 100
    //     const ws = (words.slice(0, n).map(w => w.thai)).join('')
    //     const letters = ws.split('').reduce(
    //         (acc, wl) => {
    //             acc.add(wl)
    //             return acc
    //         },
    //         new Set()
    //     )
    //     console.log(n, letters.size, wlf.length)
    // },

    components: {
        KeyboardLayout,
        TypingTask,
        LetterDetails,
        WordDetails,
    },
    data () {
        return {
            typingTaskPosition: 0,
            nextWord: '',
        }
    },
    computed: {
        typingTask () {
            return sampleText.split(' ').reduce((acc, word) => {
                for (let i = 0; i < 5; i++) {
                    acc += word + ' '
                }
                return acc
            }, '')
            // const ws =  words.map(w => w.thai)
            //     .filter(this.allLettersAllowed)
            //     .slice(0, 100)
            // // console.log(ws.length)
            // return ws.join(' ')
            // return this._createRandomTask(100)
        },
        letters () {
            return wlf //.filter(l => l.frequency >= 0.03)
        },
        // taskLetters () {
        //     // return "านรอกเ"
        //     return "านรอกเง่มั"
        // },
        enabledLetters () {
            // return this.letters.filter(l => this.taskLetters.includes(l.letter))
            return this.letters // .slice(0, 7) //.filter(l => l.type === "sonorant")
        },
        kedmanee () {
            return this.enabledLetters.reduce(
                (acc, l) => {
                    acc[l.keyboard_locations.kedmanee] = l.letter
                    return acc
                },
                {}
            )
        },
        frequencies () {
            return this.enabledLetters.reduce(
                (acc, l) => {
                    acc[l.letter] = l.frequency
                    return acc
                },
                {}
            )
        }
    },
    methods: {
        allLettersAllowed (word) {
            return [...word].filter(l => this.enabledLetters.find(el => el.letter === l)).length === word.length
        },
        onKeyPressed (key) {
            const correct = this.typingTask[this.typingTaskPosition] === key
            if (correct) {
                // this.$refs.letterDetails.play()
                this.typingTaskPosition++
                while (this.typingTask[this.typingTaskPosition] === ' ' ||
                    this.typingTask[this.typingTaskPosition] === '\n'
                ) {
                    this.typingTaskPosition++
                    this.nextWord = this._getNextWord()
                    this.$refs.wordDetails.play()
                }
                if (this.typingTaskPosition >= this.typingTask.length) {
                    this.typingTaskPosition = 0
                }
            }
            setTimeout(() => this.$refs.keyboardLayout.resetPressedKey(), 0)
        },
        _createRandomTask (n) {
            const result = []
            for (let i = 0; i < n; i++) {
                result.push(this._getRandomEnabledLetter())
            }
            return result.join('')
        },
        _getRandomEnabledLetter (n) {
            const el = this.enabledLetters
            const i = Math.floor(Math.random() * el.length)
            return el[i].letter
        },
        _getNextWord () {
            let i
            for (
                i = this.typingTaskPosition + 1;
                this.typingTask[i] !== ' ' && this.typingTask[i] !== '\n';
                i++
            );
            return this.typingTask.slice(this.typingTaskPosition, i)
        }
    }
})
</script>

<style lang="stylus" scoped>
.letter-details
  position absolute
  right 1em
  top 1em

.word-details
  position absolute
  left 1em
  top 5em

.letters
    font-size 3em
    margin-top 3em
    text-align center
    > span
        opacity 0.1
        display inline-block
        padding 0.2em
        &.enabled
            opacity 1.0
</style>
