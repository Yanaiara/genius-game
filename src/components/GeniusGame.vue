<template>
  <div id="app">
    <div class="circle-container mb-4">
      <div v-for="color in colors" :key="color.name" :class="[color.class, 'button']"
        @click="handlePlayerClick(color.name)" :disabled="!playerTurn"></div>

      <button class="center-button" @click="startGame" :disabled="gameStarted">
        {{ gameStarted ? 'Jogando...' : 'Iniciar' }}
      </button>
    </div>

    <p v-if="status" class="text-lg font-semibold">{{ status }}</p>
    <p class="score-display">Pontos: {{ score }}</p>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const colors = ref([
  { name: 'red', class: 'position-top-left' },
  { name: 'blue', class: 'position-top-right' },
  { name: 'green', class: 'position-bottom-left' },
  { name: 'yellow', class: 'position-bottom-right' }
]);

const sequence = ref([]);
const playerSequence = ref([]);
const gameStarted = ref(false);
const playerTurn = ref(false);
const status = ref('');
const sequenceIndex = ref(0);
const score = ref(0);

const startGame = async () => {
  sequence.value = [];
  playerSequence.value = [];
  sequenceIndex.value = 0;
  score.value = 0;
  gameStarted.value = true;
  status.value = 'Observe a sequência...';
  await nextRound();
};

const nextRound = async () => {
  playerTurn.value = false;
  playerSequence.value = [];

  const randomColor = colors.value[Math.floor(Math.random() * colors.value.length)];
  sequence.value.push(randomColor);

  await showSequence();
  playerTurn.value = true;
  status.value = 'Sua vez!';
};

const showSequence = async () => {
  for (const color of sequence.value) {
    await highlightColor(color);
    await sleep(500);
  }
};

const highlightColor = async (color) => {
  const button = document.querySelector(`.${color.class}`);
  if (button) {
    button.classList.add('active');
    await sleep(400);
    button.classList.remove('active');
  }
};

const handlePlayerClick = (colorName) => {
  if (!playerTurn.value) return;

  playerSequence.value.push(colorName);

  const color = colors.value.find(c => c.name === colorName);
  highlightColor(color);

  const currentIndex = playerSequence.value.length - 1;
  if (playerSequence.value[currentIndex] !== sequence.value[currentIndex].name) {
    status.value = 'Você errou! Tente novamente.';
    resetGame();
    return;
  }

  if (playerSequence.value.length === sequence.value.length) {
    score.value += 1;
    status.value = 'Você acertou! Próxima rodada...';
    playerTurn.value = false;
    setTimeout(() => nextRound(), 1000);
  }
};

const resetGame = () => {
  gameStarted.value = false;
  playerTurn.value = false;
  sequence.value = [];
  playerSequence.value = [];
};

const sleep = (ms) => new Promise(resolve => setTimeout(resolve, ms));
</script>

<style scoped>
#app {
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.circle-container {
  position: relative;
  width: 320px;
  height: 320px;
  border-radius: 50%;
  overflow: hidden;
  border: 8px solid #333;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
}

.button {
  width: 50%;
  height: 50%;
  border: 4px solid #000;
  cursor: pointer;
  position: absolute;
  box-sizing: border-box;
  transition: transform 0.1s ease, opacity 0.1s ease;
}

.button.active {
  transform: scale(1.1);
  opacity: 0.6;
}

.position-top-left {
  top: 0;
  left: 0;
  background-color: #ff4c4c;
  border-top-left-radius: 50%;
}

.position-top-right {
  top: 0;
  right: 0;
  background-color: #4c4cff;
  border-top-right-radius: 50%;
}

.position-bottom-left {
  bottom: 0;
  left: 0;
  background-color: #4cff4c;
  border-bottom-left-radius: 50%;
}

.position-bottom-right {
  bottom: 0;
  right: 0;
  background-color: #ffff4c;
  border-bottom-right-radius: 50%;
}

.center-button {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 100px;
  height: 100px;
  background-color: #333;
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1rem;
  font-weight: bold;
  cursor: pointer;
}

.score-display {
  margin-top: 1rem;
  font-size: 1.2rem;
  font-weight: bold;
  color: #ffcc00;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
}
</style>
