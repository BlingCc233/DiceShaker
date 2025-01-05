<script setup>
import { ref, onMounted } from 'vue';

const props = defineProps({
  dice_num: {
    type: Number,
    required: true
  }
});

const diceImages = ref([]);

const diceResources = [];

onMounted(() => {
  for (let i = 1; i <= 6; i++) {
    diceResources.push(`assets/point${i}.png`);
  }
});

const generateDice = () => {
  diceImages.value = [];
  let currentX = 0; // 用于跟踪当前层的 translateX 位置
  for (let i = 0; i < props.dice_num; i++) {
    const randomDice = diceResources[Math.floor(Math.random() * diceResources.length)];
    // 计算 zIndex，每两到三个骰子共享同一个 zIndex
    const zIndex = Math.floor(i / 2.5);
    // 计算随机的 translateY 浮动
    const randomTranslateY = Math.random() * 10 - 5; // 在 -5px 到 5px 之间浮动
    // 计算随机的 translateX 浮动
    const randomTranslateX = Math.random() * 20 - 10; // 在 -10px 到 10px 之间浮动
    // 确保 translateX 不互相重叠
    const translateX = currentX;
    currentX += (35 * (Math.random() + 1.2)); // 每个骰子宽度为 40vw，这里假设 40vw 大约是 40% 的宽度
    if (i % 3 === 2) { // 每三个骰子后重置 currentX
      currentX = 0;
    }
    diceImages.value.push({ src: randomDice, zIndex, translateX, translateY: randomTranslateY, randomTranslateX });
  }
};

onMounted(() => {
  generateDice();
});
</script>

<template>
  <div class="dice-container">
    <img
      v-for="(dice, index) in diceImages"
      :key="index"
      :src="dice.src"
      :style="{
        zIndex: dice.zIndex,
        transform: `translateY(${dice.zIndex * 40 + dice.translateY}px) translateX(calc(${dice.translateX}% + ${dice.randomTranslateX}px))`,
      }"
      class="dice-image"
    />
  </div>
</template>

<style scoped>
.dice-container {
  width: 100%;
  position: relative; /* 确保子元素的绝对定位相对于容器 */
}

.dice-image {
  left: calc(50% - 150px);
  width: 140px;
  position: absolute;
}

img {
  filter: drop-shadow(0 0 0.6rem rgba(255, 170, 100, 0.67));

}
</style>
