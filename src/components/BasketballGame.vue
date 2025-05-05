<template>
  <div class="flex flex-col items-center justify-end h-screen bg-white relative overflow-hidden">
    <h1 class="text-3xl font-bold absolute top-4 left-1/2 transform -translate-x-1/2">🏀 Basketball Game</h1>

    <div id="hoop" class="absolute top-[8vh] left-1/2 transform -translate-x-1/2 w-[28vw] max-w-[140px] h-auto z-10">
      <svg viewBox="0 0 200 150" xmlns="http://www.w3.org/2000/svg">
        <rect x="20" y="10" width="160" height="100" rx="8" stroke="#888" fill="none" stroke-width="6"/>
        <rect x="70" y="30" width="60" height="40" stroke="#888" fill="none" stroke-width="4"/>
        <line x1="60" y1="70" x2="140" y2="70" stroke="#e53e3e" stroke-width="6"/>
        <rect x="85" y="72" width="30" height="8" fill="#555" rx="2"/>
      </svg>
    </div>

    <img
      v-if="!showBall"
      ref="staticBall"
      src="https://upload.wikimedia.org/wikipedia/commons/7/7a/Basketball.png"
      alt="ball"
      class="w-10 h-10 absolute z-10"
      :style="{
        left: staticBallX + 'px',
        bottom: staticBallY + 'px',
        transform: 'translate(-50%, 0)'
      }"
    />

    <img
      v-if="showBall"
      src="https://upload.wikimedia.org/wikipedia/commons/7/7a/Basketball.png"
      alt="ball"
      class="w-10 h-10 absolute z-10"
      :style="{
        left: ballX + 'px',
        bottom: ballY + 'px',
        transform: 'translate(-50%, 0)'
      }"
    />

    <div class="w-full max-w-md flex flex-col items-center mb-8 z-20">
      <div class="w-full h-4 bg-gray-300 rounded overflow-hidden mb-2">
        <div
          class="h-full transition-all duration-75"
          :style="{
            width: power + '%',
            backgroundColor: powerSuccessHighlight ? '#f87171' : '#22c55e'
          }"
        ></div>
      </div>

      <div
        class="w-32 h-32 bg-orange-400 rounded-full shadow-lg flex items-center justify-center text-white text-xl cursor-pointer select-none"
        @mousedown="startCharging"
        @mouseup="stopCharging"
        @mouseleave="stopCharging"
      >
        Button
      </div>

      <p class="text-lg mt-4">Score: {{ score }}</p>
      <p class="text-lg">Remaining: {{ timeLeft }}s</p>
      <p v-if="timeLeft === 0" class="mt-4 text-xl font-bold">Time's up!🎉</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, nextTick } from 'vue';

const power = ref(0);
const charging = ref(false);
const chargingDirection = ref(1);
const powerSuccessHighlight = ref(false);
const score = ref(0);
const timeLeft = ref(30);
const showBall = ref(false);
const isAnimating = ref(false);
const isSuccess = ref(false);

const staticBallX = ref(window.innerWidth / 2);
const staticBallY = ref(200);
const ballX = ref(window.innerWidth / 2);
const ballY = ref(0);

let powerInterval = null;
let countdownInterval = null;

const startCharging = () => {
  if (timeLeft.value === 0 || isAnimating.value) return;
  charging.value = true;
  chargingDirection.value = 1;

  powerInterval = setInterval(() => {
    power.value += chargingDirection.value;

    if (power.value >= 100) {
      power.value = 100;
      chargingDirection.value = -1;
    } else if (power.value <= 0) {
      power.value = 0;
      chargingDirection.value = 1;
    }
  }, 10);
};

const stopCharging = () => {
  if (!charging.value || timeLeft.value === 0) return;
  charging.value = false;
  clearInterval(powerInterval);
  chargingDirection.value = 1;
  powerSuccessHighlight.value = power.value >= 90;
  triggerBallAnimation();
};

const triggerBallAnimation = () => {
  isAnimating.value = true;
  showBall.value = true;

  const currentPower = power.value;
  isSuccess.value = currentPower >= 90;

  const startX = staticBallX.value;
  const startY = staticBallY.value;

  ballX.value = startX;
  ballY.value = startY;

  const hoop = document.getElementById('hoop');
  const hoopRect = hoop.getBoundingClientRect();

  const totalFrames = 60;
  const peakFrame = 30;
  const g = 0.5;

  let targetX, targetY;

  if (isSuccess.value) {
    targetX = hoopRect.left + (100 / 200) * hoopRect.width;
    const targetY_px = hoopRect.top + (50 / 150) * hoopRect.height;
    targetY = window.innerHeight - targetY_px;
  } else {
    const isLeft = Math.random() < 0.5;
    const missX = isLeft ? 85 : 115;
    targetX = hoopRect.left + (missX / 200) * hoopRect.width;
    const targetY_px = hoopRect.top + (50 / 150) * hoopRect.height;
    targetY = window.innerHeight - targetY_px;
  }

  const vx = (targetX - startX) / peakFrame;
  const vy = (targetY - startY) / peakFrame + 0.5 * g * peakFrame;

  let frame = 0;

  const animate = () => {
    const t = frame;
    ballX.value = startX + vx * t;
    ballY.value = startY + vy * t - 0.5 * g * t ** 2;

    if (frame < totalFrames) {
      frame++;
      requestAnimationFrame(animate);
    } else {
      if (isSuccess.value) score.value++;
      showBall.value = false;
      power.value = 0;
      isAnimating.value = false;
      powerSuccessHighlight.value = false;
    }
  };

  requestAnimationFrame(animate);
};

onMounted(() => {
  countdownInterval = setInterval(() => {
    if (timeLeft.value > 0) {
      timeLeft.value--;
    } else {
      clearInterval(countdownInterval);
    }
  }, 1000);

  nextTick(() => {
    const btn = document.querySelector(".bg-orange-400");
    if (btn) {
      const rect = btn.getBoundingClientRect();
      staticBallX.value = rect.left + rect.width / 2;
      staticBallY.value = window.innerHeight - rect.top + 30;
    }
  });
});

onUnmounted(() => {
  clearInterval(powerInterval);
  clearInterval(countdownInterval);
});
</script>

<style scoped>
img {
  transition: none !important;
}
</style>
