<script setup lang="ts">
import { onBeforeMount, reactive, ref, watchEffect } from 'vue';
import { useHead } from '@vueuse/head'

type OptionState = 'blank' | 'red' | 'green'
interface Option {
  state: OptionState
}

const counter = ref(0)
const options = reactive<Option[]>([]);
const winner = ref('');
const aTie = ref(false);

const images = {
  red: new URL('./assets/x.svg', import.meta.url).href,
  green: new URL('./assets/circle.svg', import.meta.url).href,
  blank: ''
}

useHead({
  title: 'Jogo da Idosa'
})

watchEffect(() => { if (counter.value === 9 && winner.value === '') aTie.value = true })

const verifyWinner = () => {
  const [one, two, three, four, five, six, seven, eight, nine] = options

  const firstRow = [one.state, two.state, three.state]
  const secondRow = [four.state, five.state, six.state]
  const thirdRow = [seven.state, eight.state, nine.state]

  const firstColumn = [one.state, four.state, seven.state]
  const secondColumn = [two.state, five.state, eight.state]
  const thirdColumn = [three.state, six.state, nine.state]

  const firstDiagonal = [one.state, five.state, nine.state]
  const secondDiagonal = [three.state, five.state, seven.state]

  const rules = [firstRow, secondRow, thirdRow, firstColumn, secondColumn, thirdColumn, firstDiagonal, secondDiagonal]

  rules.forEach(rule => {
    if (rule.every(item => item === 'red')) {
      winner.value = 'Vermelho'
    } else if (rule.every(item => item === 'green')) {
      winner.value = 'Verde'
    }
  })
}

const getRandomIntInclusive = (min: number, max: number) => {
  min = Math.ceil(min);
  max = Math.floor(max);
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

const setSelected = (index: number) => {
  if (options[index].state === 'blank' && winner.value === '') {
    const isPair = counter.value % 2 === 0;
    options[index].state = isPair ? 'red' : 'green';
    counter.value++
    verifyWinner()
  }
}

const click = (index: number) => {
  if (options[index].state === 'blank' && winner.value === '') {
    setSelected(index)
    let mathRandom = getRandomIntInclusive(0, 8)
    while (options[mathRandom].state !== 'blank' && options.some(option => option.state === 'blank')) {
      mathRandom = getRandomIntInclusive(0, 8)
    }
    setSelected(mathRandom)
  }
}

const reset = () => {
  counter.value = 0;
  options.forEach(option => {
    option.state = 'blank';
    winner.value = ''
    aTie.value = false
  });
}

onBeforeMount(() => {
  for (let i = 0; i < 9; i++) {
    options.push({ state: 'blank' });
  }
})
</script>

<template>
  <main>
    <h2 v-show="winner">O vencedor é o {{ winner }}!</h2>
    <h2 v-show="aTie">Respeite a idosa!</h2>

    <div class="game-grid">
      <button v-for="(option, index) in options" :class="['game-option', option.state]" @click="click(index)"
        :key="index">
        <img v-if="option.state !== 'blank'" :src="(images[option.state])" />
      </button>
    </div>

    <div class="controls-box">
      <button class="reset-button" @click="reset">Resetar o jogo</button>
      <p>
        Quantidade de jogadas: {{ counter }}
      </p>
    </div>
  </main>
</template>

<style>
body {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}

main {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  background: #27272a;
}

h2 {
  color: white;
  font-family: sans-serif;
}

.game-grid {
  height: 25rem;
  width: 25rem;
  display: grid;
  border: 1px solid #fff;
  border-radius: 10%;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, 1fr);
  align-items: center;
  justify-items: center;
}

.game-option {
  border: 1px solid #fff;
  border-radius: 20%;
  width: 7rem;
  height: 7rem;
  cursor: pointer;
  background: #3f3f46;
}

.green {
  background: #134e4a;
  border: 1px solid #bef264;
}

.red {
  background: #831843;
  border: 1px solid #f9a8d4;
}

.green img,
.red img {
  filter: invert(1);
}

.controls-box {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.reset-button {
  background: #fff;
  border: 1px solid #fff;
  padding: 0.5rem;
  border-radius: 4px;
  margin-top: 3rem;
  cursor: pointer;
  color: #27272a;
}

p {
  color: white
}
</style>
