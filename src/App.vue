<script lang="ts">
import GameNotification from './components/GameNotification.vue'
import GamePopup from './components/GamePopup.vue'
import GameWord from './components/GameWord.vue'
import GameFigure from './components/GameFigure.vue'
import GameHeader from './components/GameHeader.vue'
import GameWrongLetters from './components/GameWrongLetters.vue'
import dictionary from './data/dictionary'
import { ref, computed, onMounted, watch } from 'vue'

export default {
  components: { GameHeader, GameFigure, GameWrongLetters, GameWord, GamePopup, GameNotification },
  setup() {
    const getRandomWord = () => {
      const index = Math.floor(Math.random() * dictionary.length)
      randomWord.value = dictionary[index]
    }

    const randomWord = ref('слово')
    const word = computed(() => randomWord.value)

    const letters = ref<string[]>([])
    const correctLetters = computed(() => letters.value.filter((x) => word.value.includes(x)))
    const wrongLetters = computed(() => letters.value.filter((x) => !word.value.includes(x)))
    const isLose = computed(() => wrongLetters.value.length === 6)
    const isWin = computed(() => [...word.value].every((x) => correctLetters.value.includes(x)))
    const notification = ref<InstanceType<typeof GameNotification> | null>(null)
    const popup = ref<InstanceType<typeof GamePopup> | null>(null)

    onMounted(() => {
      getRandomWord()
      console.log(randomWord.value)
      watch(wrongLetters, () => {
        if (isLose.value) {
          popup.value?.open('lose')
        }
      })
      watch(correctLetters, () => {
        if (isWin.value) {
          popup.value?.open('win')
        }
      })
      window.addEventListener('keydown', ({ key }) => {
        if (isLose.value || isWin.value) {
          return
        }
        if (letters.value.includes(key)) {
          notification.value?.open()
          setTimeout(() => notification.value?.close(), 2000)
          return
        }
        if (/[а-яА-яїїґҐ]/.test(key)) {
          letters.value.push(key.toLowerCase())
        }
      })
    })
    const restart = () => {
      letters.value = []
      getRandomWord()
      console.log(randomWord.value)
      popup.value?.close()
    }

    return {
      word,
      letters,
      correctLetters,
      wrongLetters,
      notification,
      popup,
      restart,
      randomWord,
      getRandomWord,
      dictionary
    }
  }
}
</script>

<template>
  <div>
    <GameHeader />
    <div class="game-container">
      <GameFigure :wrongLettersCount="wrongLetters.length" />
      <GameWrongLetters :wrong-letters="wrongLetters" />
      <GameWord :word="word" :correct-letters="correctLetters" />
    </div>
    <GamePopup :word="word" ref="popup" @restart="restart" />
    <GameNotification ref="notification" />
  </div>
</template>

<style scoped></style>
