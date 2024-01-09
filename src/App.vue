<script setup>
import { reactive, ref } from "vue";
import Hint from "./components/Hint.vue";

const palette = [
  "#9ADCFF",
  "#FFF89A",
  "#6867AC",
  "#FFB2A6",
  "#FF8AAE",
  "#B5EAEA",
  "#1572A1",
  "#D9D7F1",
];
const sounds = {
  tap: new Audio("tap.mp3"),
  choose: new Audio("choose.mp3"),
};
let timeout = null;
let touches = reactive([]);
let choosed = ref(-1);
let showDialog = ref(showIosInstallModal());
let showHint = ref(Boolean(localStorage.getItem("hide_hint")) !== true);

function onTouchstart(event) {
  clearTimeout(timeout);
  Object.assign(touches, event.touches);
  sounds["tap"].play();
  showHint.value = false;
  localStorage.setItem("hide_hint", true);
  if (event.touches.length >= 2) {
    timeout = setTimeout(() => {
      const index = Math.floor(Math.random() * touches.length);
      const { identifier } = touches[index];
      choosed.value = identifier;
      sounds["choose"].play();
    }, 3000);
  }
}

function onTouchmove(event) {
  Object.assign(touches, event.touches);
}

function onTouchend(event) {
  clearTimeout(timeout);
  for (let i = 0; i < event.changedTouches.length; i++) {
    const { identifier } = event.changedTouches[i];
    const index = touches.findIndex((touch) => touch.identifier === identifier);
    touches.splice(index, 1);
    if (choosed.value >= 0) {
      choosed.value = -1;
    }
  }
}

function showIosInstallModal() {
  const shouldShowModalResponse =
    (/iphone|ipad|ipod/.test(window.navigator.userAgent.toLowerCase()) ||
      (/Macintosh/i.test(window.navigator.userAgent) &&
        window.navigator.maxTouchPoints &&
        window.navigator.maxTouchPoints > 1)) &&
    !window.navigator?.standalone &&
    !localStorage.getItem("pwa_install_prompt_shown");
  localStorage.setItem("pwa_install_prompt_shown", true);
  return shouldShowModalResponse;
}

document.addEventListener(
  "touchmove",
  function (event) {
    if (event.scale !== 1) {
      event.preventDefault();
    }
  },
  { passive: false }
);
</script>

<template>
  <div
    class="bg"
    @touchstart="onTouchstart"
    @touchmove="onTouchmove"
    @touchend="onTouchend"
  >
    <hint :show="showHint" />
    <div
      v-for="({ identifier, clientX, clientY }, index) in touches"
      class="finger"
      :class="{ active: identifier === choosed }"
      :key="identifier"
      :style="{
        '--left': `${clientX}px`,
        '--top': `${clientY}px`,
        '--color': palette[index],
      }"
    />
  </div>
  <dialog :open="showDialog">
    <p>
      Install this application on your home screen for quick, easy and offline
      access when you’re on the go. Close this modal, tap the “share” icon, and
      then tap on “Add to home screen”.
    </p>
    <button @click="showDialog = false">Got it</button>
  </dialog>
</template>

<style lang="scss" scoped>
dialog {
  border: none;
  border-radius: 0.8rem;
  padding: 1rem;
  width: 80vw;
  position: fixed;
  top: 1rem;
  background-color: rgba(255, 255, 255, 0.9);
  left: 0;
  right: 0;
  p {
    margin: 0 0 0.5rem;
    font-size: 0.85rem;
  }
  button {
    border: 1px solid;
    border-radius: 3px;
    padding: 4px 12px;
    font-weight: 500;
    font-family: "Open Sans", sans-serif;
  }
}
.bg {
  width: 100dvw;
  height: 100dvh;
  position: fixed;
  inset: 0;
  background-color: #000;
  -webkit-user-select: none; /* Safari */
  -ms-user-select: none; /* IE 10 and IE 11 */
  user-select: none; /* Standard syntax */
}

.finger {
  border-radius: 50%;
  width: 80px;
  height: 80px;
  display: block;
  position: absolute;
  left: var(--left);
  top: var(--top);
  background-color: var(--color);
  animation: ripple 600ms, scale 800ms ease-in-out infinite alternate;
  margin-top: -40px;
  margin-left: -40px;

  &::before {
    content: " ";
    position: absolute;
    inset: -12px;
    border-radius: 50%;
    display: block;
    border: 8px solid var(--color);
    animation: rotate 800ms ease-in-out infinite alternate;
    opacity: 0.65;
  }

  &.active {
    z-index: 1;
    &::after {
      content: " ";
      position: absolute;
      inset: 50%;
      transform: translate(-50%, -50%);
      border-radius: 50%;
      border: calc(var(--max-size) * 3.5) solid var(--color);
      animation: backdrop 400ms ease-in-out;
      width: 128px;
      height: 128px;
    }
  }
}

@keyframes backdrop {
  from {
    width: calc(var(--max-size) * 3.5);
    height: calc(var(--max-size) * 3.5);
  }
  to {
    width: 128px;
    height: 128px;
  }
}

@keyframes rotate {
  from {
    border-width: 8px;
  }
  to {
    border-width: 4px;
  }
}

@keyframes scale {
  from {
    scale: 0.9;
  }
  to {
    scale: 1;
  }
}

@keyframes ripple {
  from {
    width: 0;
    height: 0;
    margin-top: 0;
    margin-left: 0;
  }
  to {
    width: 80px;
    height: 80px;
    margin-top: -40px;
    margin-left: -40px;
  }
}
</style>
