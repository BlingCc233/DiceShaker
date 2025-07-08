<template>

  <RandDice :key="randKey" :dice_num="diceCount" class="dices"/>

  <div class="dice-shake">

    <div class="mask" ref="mask" @mousedown="startDrag" @touchstart="startDragTouch"></div>


    <div class="buttion-area">
      <div id="setting-btn" @click="showDialog">
        <icon name="setting-1" style="color: #f9f9f9;" size="2rem"/>
      </div>
      <div id="shake-btn" @click="shakeDice">
        摇
      </div>
    </div>
  </div>


  <t-dialog
      v-model:visible="visible"
      header="骰子数量"
      width="80%"

      :confirm-on-enter="true"
      :footer="false"

      :on-close="close"
  >
    <t-space direction="vertical" style="width: 90%">

      <div>
        <p>选择骰子个数</p>
      </div>
      <div>
        <p>当前骰子数量：{{ diceCount }}</p>
      </div>
      <t-slider v-model="diceCount" :label="true" theme="capsule" :min="1" :max="8" :step="1">
      </t-slider>
    </t-space>
  </t-dialog>
</template>

<script setup>
import {onMounted, ref, watch} from 'vue';
import RandDice from './components/RandDice.vue';
import {Icon} from 'tdesign-icons-vue-next';


const diceCount = ref(5); // 默认骰子数量
const visible = ref(false);
const mask = ref(null);
let isDragging = false;
let initialY = 0;
let maskTop = 0;

const startPos = ref(null);
const stopPos = ref(null);

const randKey = ref("");


const shakeDice = () => {
  randKey.value = Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15);

  // 添加 shake-four-times 动画类
  mask.value.classList.add('shake-four-times');
  mask.value.style.top = `${startPos.value}px`;

  // 设置 interval 每隔 1ms 将 isDragging 设置为 false
  const intervalId = setInterval(() => {
    isDragging = false;
  }, 1);

  // 设置 timeout 在 1.5 秒后清除 interval
  setTimeout(() => {
    clearInterval(intervalId);
    mask.value.style.top = `${startPos.value}px`;

  }, 1500);

  // 移除 shake-four-times 动画类
  mask.value.addEventListener('animationend', () => {
    mask.value.classList.remove('shake-four-times');
  }, {once: true});


};


watch(diceCount, () => {
  randKey.value = Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15);
});


onMounted(() => {
  startPos.value = window.innerHeight / 7;
  stopPos.value = -window.innerHeight / 4;
  console.log(startPos.value);
  console.log(stopPos.value);
  randKey.value = Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15);
  mask.value.style.top = `${startPos.value}px`;
});

const startDrag = (e) => {
  isDragging = true;
  initialY = e.clientY;
  maskTop = mask.value.offsetTop;
};

const startDragTouch = (e) => {
  isDragging = true;
  initialY = e.touches[0].clientY;

  maskTop = mask.value.offsetTop;

};

const drag = (e) => {
  if (isDragging) {
    const deltaY = e.clientY - initialY;
    if (maskTop + deltaY > startPos.value) {
      mask.value.style.top = `${startPos.value}px`;
      return;
    }
    if (maskTop + deltaY < stopPos.value) {
      mask.value.style.top = `${stopPos.value}px`;
      return;
    }
    mask.value.style.top = `${maskTop + deltaY}px`;
  }
};

const dragTouch = (e) => {
  if (isDragging) {
    const deltaY = e.touches[0].clientY - initialY;
    if (maskTop + deltaY > startPos.value) {
      mask.value.style.top = `${startPos.value}px`;
      return;
    }
    if (maskTop + deltaY < stopPos.value) {
      mask.value.style.top = `${stopPos.value}px`;
      return;
    }
    mask.value.style.top = `${maskTop + deltaY}px`;
  }
};

const stopDrag = () => {
  isDragging = false;
};

const showDialog = () => {
  visible.value = true;
};

window.addEventListener('mousemove', drag);
window.addEventListener('mouseup', stopDrag);
window.addEventListener('touchmove', dragTouch);
window.addEventListener('touchend', stopDrag);
</script>

<style scoped>
* {
  user-select: none;
  outline: none;
}


.dice-shake {
  height: 100vh;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
}

.buttion-area {
  width: 100%;
  height: 20vh;
  display: flex;
  flex-direction: row;
  justify-content: space-evenly;
  align-items: center;
}

#shake-btn {
  width: 65px;
  height: 65px;
  font-size: 2rem;
  border-radius: 50%;
  border: solid 1px rgba(255, 255, 255, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  color: #fff;
  background-color: rgb(255, 100, 105);
  filter: drop-shadow(0 0 1.2rem rgba(255, 100, 105, 0.67));
  outline: none;
}

#setting-btn {
  width: 65px;
  height: 65px;
  border-radius: 50%;
  border: solid 1px rgba(255, 255, 255, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  filter: drop-shadow(0 0 3rem rgb(255, 255, 255));
  outline: none;


}

.mask {
  position: relative;
  min-width: 375px;
  max-width: 500px;
  min-height: 375px;
  max-height: 60vh;
  background-image: url("../public/assets/cup.png");
  background-size: contain;
  background-repeat: no-repeat;
  opacity: 1;
  cursor: grab;
  filter: drop-shadow(0 0 2rem rgba(105, 100, 255, 0.67));
}

.mask.shake-four-times {
  animation: shake-four-times 0.5s ease-in-out 3;
}


.mask:active {
  cursor: grabbing;
}

.dices {
  position: fixed;
  top: 35vh;
  left: 0;
  z-index: -1;
}

@keyframes shake-four-times {
  0%, 100% {
    transform: rotate(0deg);
  }
  25% {
    transform: rotate(-5deg);
  }
  50% {
    transform: rotate(10deg);
  }
  75% {
    transform: rotate(-5deg);
  }
}


@media (max-width: 768px) {
  .buttion-area {
    animation: slideUp 0.4s ease-out;
  }

  .dice-shake {
    animation: slideUp 0.4s ease-out;
  }

  @keyframes slideUp {
    from {
      opacity: 0;
      transform: translateY(-20px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }
}

@media (max-width: 1280px) {
  .mask {
    min-width: 500px;
    max-width: 500px;
    min-height: 500px;
    max-height: 500px;
  }

}

@media (max-width: 768px) {
  .mask {
    min-width: 375px;
    max-width: 375px;
    min-height: 80vh;
    max-height: 100vh;
  }

}
</style>
