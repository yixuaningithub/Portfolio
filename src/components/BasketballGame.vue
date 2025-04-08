<template>
  <div class="flex flex-col items-center justify-center h-screen bg-orange-100 p-4">
    <h1 class="text-3xl font-bold mb-4">🏀 投籃遊戲</h1>

    <div class="w-full max-w-md flex flex-col items-center">
      <div class="w-full h-4 bg-gray-300 rounded overflow-hidden mb-2">
        <div
          class="h-full bg-green-500 transition-all duration-75"
          :style="{ width: power + '%' }"
        ></div>
      </div>

      <div
        class="w-32 h-32 bg-orange-400 rounded-full shadow-lg flex items-center justify-center text-white text-xl cursor-pointer select-none mb-4"
        @mousedown="startCharging"
        @mouseup="stopCharging"
        @mouseleave="stopCharging"
      >
        投籃
      </div>

      <p class="text-lg">分數：{{ score }}</p>
      <p class="text-lg">剩餘時間：{{ timeLeft }}s</p>
      <p v-if="timeLeft === 0" class="mt-4 text-xl font-bold">時間到！🎉</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';

const power = ref(0);
const charging = ref(false);
const score = ref(0);
const timeLeft = ref(30);
let powerInterval = null;
let countdownInterval = null;

const startCharging = () => {
  if (timeLeft.value === 0) return;
  charging.value = true;
  powerInterval = setInterval(() => {
    power.value = power.value >= 100 ? 0 : power.value + 1;
  }, 10);
};

const stopCharging = () => {
  if (!charging.value || timeLeft.value === 0) return;
  charging.value = false;
  clearInterval(powerInterval);
  shootBall(power.value);
  power.value = 0;
};

const shootBall = (currentPower) => {
  const bestPower = 80;
  const errorMargin = Math.abs(currentPower - bestPower);
  const successChance = Math.max(0, 100 - errorMargin * 2);
  const isSuccess = Math.random() * 100 < successChance;
  if (isSuccess) score.value++;
};

onMounted(() => {
  countdownInterval = setInterval(() => {
    if (timeLeft.value > 0) {
      timeLeft.value--;
    } else {
      clearInterval(countdownInterval);
    }
  }, 1000);
});

onUnmounted(() => {
  clearInterval(powerInterval);
  clearInterval(countdownInterval);
});
</script>

<style scoped>
</style>
