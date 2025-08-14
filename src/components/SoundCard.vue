<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import type { Ref } from 'vue';

defineProps({
  audioName: {
    type: String,
    required: true,
  }
});

const audio = ref();
const isClicked = ref(false);
const shouldIgnore = ref(false);
const btnRef: Ref<HTMLElement | null> = ref(null);
const height = ref('0%');

const getHeight = () => {
  if(!isClicked.value){
    return "0%";
  }

  return `${audio.value.volume * 100}%`;
}

const togglePlay = () => {
  if(shouldIgnore.value){
    return
  }
  isClicked.value = !isClicked.value;

  // start playing
  if(isClicked.value){
    height.value = getHeight();
    audio.value.play();
    return;
  }

  // pause with a fade out effect
  shouldIgnore.value = true;
  const lastVol = audio.value.volume;

  const interval = setInterval(function(){
    if(audio.value.volume < 0.1){
      audio.value.pause();
      audio.value.currentTime = 0;
      audio.value.volume = lastVol < 0.1 ? 1 : lastVol;
      clearInterval(interval);
      shouldIgnore.value = false;
      return;
    }

    if(audio.value.volume > 0.1){
      audio.value.volume -= 0.1;
      return
    }
  }, 200);
}

const controlVolume = (e: Event) => {
  const wheelEvent = e as WheelEvent;
  
  // down
  if(wheelEvent.deltaY > 0 && audio.value.volume > 0.1){
    audio.value.volume -= 0.1;
  }

  else if(wheelEvent.deltaY < 0 && audio.value.volume < 1){
    audio.value.volume += 0.1;
  }

  height.value = getHeight();
}

onMounted(()=>{
  if(btnRef.value){
    btnRef.value.addEventListener('wheel', controlVolume);
  }
})

onUnmounted(()=>{
  if(btnRef.value){
    btnRef.value.removeEventListener('wheel', controlVolume);
  }
})

</script>

<template>
  <div 
    class="btn rounded-lg flex justify-center items-center relative"
    @click="togglePlay"
    :class="{'spec': isClicked}"
    ref="btnRef"
    @scroll="controlVolume"
  >
    <div
      v-if="isClicked"
      class="volume w-full absolute bottom-0"
      :style="{
        height: height
      }"
    />
    <slot />
    <audio 
      ref="audio"
      loop
      :src="`/ambient/${audioName}`"
      type="audio/ogg"
    />
  </div>
</template>


<style scoped>
.btn{
  width: 26%;
  height: 40%;
  border: solid 2px var(--color-shadow);
}

.btn:hover{
  border-color: var(--color-hover);
  color: var(--color-text-primary);
}

.spec{
  border-color: var(--color-primary);
  border-width: 3px;
}

.volume{
  background-color: var(--color-shadow);
  opacity: 80%;
}
</style>
