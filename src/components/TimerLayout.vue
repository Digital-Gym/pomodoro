<script setup>
import { ref, computed } from 'vue';
import PlayIcon from './icons/PlayIcon.vue'
import StopIcon from './icons/StopIcon.vue'
import PauseIcon from './icons/PauseIcon.vue'
import CheckIcon from './icons/CheckIcon.vue'
import CloseIcon from './icons/CloseIcon.vue'
import  AddIcon from './icons/AddIcon.vue';
import LikeIcon from './icons/LikeIcon.vue';
import { playBack, playClick, playAlarm, stopAlarm } from '../misc/audio';
import { notify } from '../misc/notify';


// set in seconds 1500 = 25 min
const timer = ref(0);
let myInterval;
const isPaused = ref(true);
const isDialog = ref(false);
const isAlarm = ref(false);
const isExtra = ref(false);

const pomodoro = [1500, 300, 1500, 300, 1500, 300, 1500, 1800];
// const pomodoro = [2, 1, 2, 1, 2, 1, 2, 5];

const index = ref(0);
const streak = ref(0);

const emit = defineEmits(['onStart', 'onPause', 'onStop']);

const strTime = computed(()=>{
  let min = Math.floor(timer.value / 60);
  let sec = Math.round(((timer.value / 60) - min) * 60);

  min = min.toFixed().padStart(2, "0").split('');
  sec = sec.toFixed().padStart(2, "0").split('');

  return [...min,':',...sec];
})

const decrTime = () => {
  if(timer.value > 0){
    timer.value --;
  } else{
    // Timer stopped
    let inventory = JSON.parse(localStorage.getItem('inventory'));
    const current = localStorage.getItem('selectedTheme');
    playAlarm();
    isAlarm.value = true;
    
    // work -> break
    if(pomodoro[index.value] == pomodoro[0]){
      notify("Stop working", "It's time to take a break!");
      streak.value = streak.value < 4 ? streak.value+1 : 1;
      inventory[current].count++;
      localStorage.setItem('inventory', JSON.stringify(inventory));
    } 
    // extra -> work/break
    else if(isExtra.value){
      notify("Hey!", "Your extra time is up");
      isExtra.value = false;
      isPaused.value = true;
      clearInterval(myInterval);
      timer.value = 0;
      emit('onStop');
      return;
    }
    // break -> work
    else{
      notify("Hey!", "It's time to work");
    }
    stopTimer();
  }
}

const runTimer = (time) => {
  timer.value = time;
  myInterval = setInterval(decrTime, 1000);
}

const getSeconds = () => {
  if(index.value > pomodoro.length - 1){
    index.value = 0;
  }
  return pomodoro[index.value];
}

const startTimer = (e, time) => {
  playClick();
  isPaused.value = false;
  clearInterval(myInterval);

  // 25 min
  const secs = time ?? getSeconds();

  if(timer.value === 0){
    runTimer(secs);
    emit('onStart', secs);
    return
  }
  // continue
  runTimer(timer.value);
  emit('onStart');
}

const pauseTimer = () => {
  playBack();
  isPaused.value = true;
  clearInterval(myInterval);
  emit('onPause');
}

const stopTimer = () => {
  isPaused.value = true;
  clearInterval(myInterval);
  timer.value = 0;
  index.value++;
  emit('onStop');
}

const confirmStop = (decision) => {
  isDialog.value = false;

  if(decision){
    playBack();
    stopTimer();
    return;
  }

  playClick();
}

const confirmAlarm = (decision) => {
  isAlarm.value = false;
  isDialog.value = false;
  stopAlarm();

  if(decision){
    isExtra.value = true;
    startTimer(null, 60); // extra time
    return;
  }

  startTimer();
  playClick();
}


defineExpose({
  pauseTimer,
  startTimer
})
</script>

<template>
  <div 
    class="flex justify-center items-center flex-col pt-3 w-2/4 h-full relative"
  > 
    <span 
      v-if="streak > 0" 
      class="
        text-sm text-center
        absolute bottom-3 right-3
        select-none sec
        "
    >
      {{ streak }}
    </span>

    <!-- what a shity code is that -->
    <div class="w-full flex justify-center">
      <h2 
        class="text-6xl w-1/6 select-none text-center overflow-hidden"
        v-for="digit in strTime"
      >
        {{ digit }}
      </h2>
    </div>
    
    <!-- Toolbox -->
    <div class="toolbox flex w-3/6 mt-2 flex-col items-center gap-3">
      <div v-if="!isDialog && !isAlarm" class="w-full flex justify-around items-center">
        <PlayIcon v-if="isPaused" @click="startTimer"/>
        <PauseIcon v-else @click="pauseTimer" />
        <StopIcon @click="isDialog = true; playClick();"/>
      </div>
      <!-- confirm to stop -->
      <div v-if="isDialog && !isAlarm" class="w-full flex justify-around items-center">
        <CheckIcon @click="confirmStop(true)" />
        <CloseIcon @click="confirmStop(false)" />
      </div>

      <div v-if="isAlarm" class="w-full flex justify-around items-center">
        <LikeIcon @click="confirmAlarm(false)"/>
        <AddIcon @click="confirmAlarm(true)"/>
      </div>
    </div>
  </div>
</template>

<style scoped>
.toolbox > * > *:hover{
  color: var(--color-hover);
}

.toolbox > * > *{
  width: 2rem;
  height: 2rem;
}

.sec{
  color: var(--color-text-secondary);
}
</style>