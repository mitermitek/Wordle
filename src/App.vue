<script setup>
import { reactive, onMounted, computed } from "vue";
import SimpleKeyboard from "./components/SimpleKeyboard.vue"
import WordRow from "./components/WordRow.vue"
import Words from "an-array-of-french-words"

const words = Words.filter(w => w.length === 5 && /^[a-zA-Zéèâ]+$/.test(w))
const word = words[Math.floor(Math.random() * words.length)].replace(/[â]/g, "a").replace(/[éè]/g, "e")

const state = reactive({
  solution: word,
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
    if (currentGuess.length == 5) {
      state.currentGuessIndex++;

      for (var i = 0; i < currentGuess.length; i++) {
        let c = currentGuess.charAt(i)

        if (c == state.solution.charAt(i)) {
          state.guessedLetters.found.push(c)
        } else if (state.solution.indexOf(c) != -1) {
          state.guessedLetters.hint.push(c)
        } else {
          state.guessedLetters.miss.push(c)
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
  <p v-if="!wonGame && !lostGame" class="text-center">A toi de jouer !</p>
  <p v-if="wonGame" class="text-center">Tu as gagné !</p>
  <p v-if="lostGame" class="text-center">Tu as perdu !</p>
  <div class="flex flex-col justify-evenly h-screen max-w-md mx-auto">
    <div>
      <word-row
        v-for="(guess, i) in state.guesses"
        :key="i"
        :value="guess"
        :solution="state.solution"
        :submitted="i < state.currentGuessIndex"
      />
    </div>
    <simple-keyboard @onKeyPress="handleInput" :guessedLetters="state.guessedLetters" />
  </div>
</template>

<style></style>