<script setup>
import { reactive, onMounted, computed } from "vue";
import SimpleKeyboard from "./components/SimpleKeyboard.vue"
import WordRow from "./components/WordRow.vue"
import Words from "an-array-of-french-words"

const words = Words.filter(w => w.length === 5 && /^[a-zA-Zéèâ]+$/.test(w))
const wordsWithoutAccents = words.map(w => w.replace(/[â]/g, "a").replace(/[éè]/g, "e"))
const word = words[Math.floor(Math.random() * words.length)]
const wordToGuess = word.replace(/[â]/g, "a").replace(/[éè]/g, "e")

const state = reactive({
  solution: wordToGuess,
  guesses: ["", "", "", "", "", ""],
  currentGuessIndex: 0,
  guessedLetters: {
    miss: [],
    found: [],
    hint: []
  }
})

const wonGame = computed(
  () => state.guesses[state.currentGuessIndex - 1] === state.solution
);

const lostGame = computed(
  () => !wonGame.value && state.currentGuessIndex >= 6
);

const handleInput = (key) => {
  if (state.currentGuessIndex >= 6 || wonGame.value) {
    return;
  }
  const currentGuess = state.guesses[state.currentGuessIndex]

  if (key == "{enter}") {
    if (currentGuess.length == 5 && wordsWithoutAccents.includes(currentGuess)) {
      state.currentGuessIndex++;

      for (var i = 0; i < currentGuess.length; i++) {
        let c = currentGuess.charAt(i)

        if (c == state.solution.charAt(i)) {
          state.guessedLetters.found.push(c.toUpperCase())
        } else if (state.solution.indexOf(c) != -1) {
          state.guessedLetters.hint.push(c.toUpperCase())
        } else {
          state.guessedLetters.miss.push(c.toUpperCase())
        }
      }
    }
  } else if (key == "{bksp}") {
    state.guesses[state.currentGuessIndex] = currentGuess.slice(0, -1)
  } else if (currentGuess.length < 5) {
    const alphaRegex = /[a-zA-Z]/
    if (alphaRegex.test(key)) {
      state.guesses[state.currentGuessIndex] += key
    }
  }
}

onMounted(() => {
  window.addEventListener("keyup", (e) => {
    e.preventDefault()
    let key = e.keyCode == 13 ? "{enter}" : e.keyCode == 8 ? "{bksp}" : String.fromCharCode(e.keyCode).toLowerCase()
    handleInput(key)
  })
})
</script>

<template>
  <div class="flex flex-col justify-between h-screen max-w-md mx-auto">
    <div v-if="!wonGame && !lostGame" class="bg-blue-100 border-t-4 border-blue-500 rounded-b text-blue-900 px-4 py-3"
      role="alert">
      <div class="flex">
        <div>
          <p class="font-semibold">Trouve le mot. Bonne chance ! 🤓</p>
        </div>
      </div>
    </div>
    <div v-if="wonGame" class="bg-green-100 border-t-4 border-green-500 rounded-b text-green-900 px-4 py-3"
      role="alert">
      <div class="flex">
        <div>
          <p class="font-semibold">Tu as gagné ! Hop + x points sur ton profil ! 🥳</p>
        </div>
      </div>
    </div>
    <div v-if="lostGame" class="bg-red-100 border-t-4 border-red-500 rounded-b text-red-900 px-4 py-3" role="alert">
      <div class="flex">
        <div>
          <p class="font-semibold">Perdu ! 🥶 Le mot à trouver était : {{ word }}.</p>
        </div>
      </div>
    </div>
    <div>
      <word-row v-for="(guess, i) in state.guesses" :key="i" :value="guess" :solution="state.solution"
        :submitted="i < state.currentGuessIndex" />
    </div>
    <simple-keyboard @onKeyPress="handleInput" :guessedLetters="state.guessedLetters" />
  </div>
</template>

<style>
</style>