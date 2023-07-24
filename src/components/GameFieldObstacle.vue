<template>
  <div>
    <div
      v-if="obstacle.spawned && visible"
      ref="container"
      class="obstacle-box absolute w-[4.8vh] h-[4.8vh]"
      :class="[initialVerticalPosition, initialHorizontalPosition]"
    >
      <div ref="visibleObstacle" class="obstacle w-full h-full border-[1.2vh] border-slate-300" :class="rotateClass"></div>
    </div>
    <div 
      class="absolute indicator-box w-[4.8vh] h-[4.8vh]"
      :class="[initialVerticalPosition, initialIndicatorHorizontalPosition]"
    >
      <GameFieldObstacleIndicator
        v-if="obstacle.spawned && !visible"
        class="w-full h-full"
        :obstacle="obstacle" 
      />
    </div>
  </div>
</template>
<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import type { Obstacle } from '../types/Obstacle.js'
import type { Interval } from '../types/Interval.js'
import GameFieldObstacleIndicator from './GameFieldObstacleIndicator.vue'
import { useGameTime } from '../composables/useGameTime'
const props = defineProps<{
  obstacle: Obstacle
  gameFieldRECT: DOMRect | undefined
}>()
const emit = defineEmits<{
  (e: 'finish', value: number): void
}>()
const { pauseTime, timeInterval } = useGameTime()
const distanceX = ref(0)
const horizontalMovement = ref(0)
const visible = ref(false)
const moveIntervalId = ref<Interval>(undefined)
const container = ref<HTMLDivElement | null>(null)
const visibleObstacle = ref<HTMLDivElement | null>(null)
const initialVerticalPosition = computed(() => {
  if(props.obstacle.row === 0){
    return 'row-0'
  }
  else if(props.obstacle.row === 1){
    return 'row-1'
  }
  else if(props.obstacle.row === 2){
    return 'row-2'
  }
  else if(props.obstacle.row === 3){
    return 'row-3'
  }
  else if(props.obstacle.row === 4){
    return 'row-4'
  }
  else {
    return 'row-5'
  }
})
const initialHorizontalPosition = computed(() => {
  if(props.obstacle.direction === 'left'){
    return 'right'
  }
  else {
    return 'left'
  }
})
const initialIndicatorHorizontalPosition = computed(() => {
  if(props.obstacle.direction === 'left'){
    return 'right-indicator'
  }
  else {
    return 'left-indicator'
  }
})
const rotateDirection = computed(() => {
  if(props.obstacle.direction === 'left'){
    return 'rotate-left'
  }
  else {
    return 'rotate-right'
  }
})
const rotateClass = computed(() => {
  return props.obstacle.spawned && visible.value ? rotateDirection.value : ''
})
const move = () => {
  if(props.obstacle.direction === 'left'){
    distanceX.value -= horizontalMovement.value
  }
  else {
    distanceX.value += horizontalMovement.value
  }
}
watch(distanceX, () => {
  if(container.value && props.gameFieldRECT){
    const maxDistance = props.gameFieldRECT.width + container.value.clientWidth * 2
    if(distanceX.value > maxDistance || distanceX.value < -(maxDistance)){
      visible.value = false
      distanceX.value = 0
      clearInterval(moveIntervalId.value)
      emit('finish', props.obstacle.id)
    }
    else {
      if(container.value.classList.contains('left')){
        container.value.style.transform = `translateX(Calc(-100% + ${distanceX.value}px))`
      }
      else if(container.value.classList.contains('right')){
        container.value.style.transform = `translteX(Calc(100% + ${distanceX.value}px))`
      }
    }
  }
})
watch(props, () => {
  if(props.obstacle.spawned){
    setTimeout(() => {
      visible.value = true
      moveIntervalId.value = setInterval(move, timeInterval)
    }, pauseTime / 4)
  }
  if(props.gameFieldRECT){
    horizontalMovement.value = props.gameFieldRECT.width / 100
  }
})
</script>

<style lang="scss" scoped>
.left {
  left: 0;
  transform: translateX(-100%)
}
.right {
  right: 0;
  transform: translateX(100%);
}
.left-indicator {
  left: 0;
}
.right-indicator {
  right: 0;
  transform: rotate(180deg)
}
.row {
  &-0 {
    top: 2rem;
  }
  &-1 {
    top: Calc(2rem + 16.67%)
  }
  &-2 {
    top: Calc(2rem + 33.33%)
  }
  &-3 {
    bottom: Calc(2rem + 33.33%)
  }
  &-4 {
    bottom: Calc(2rem + 16.67%)
  }
  &-5 {
    bottom: 2rem;
  }
}
.rotate-right {
  animation: rotateRight 3s infinite;
}
.rotate-left {
  animation: rotateLeft 3s infinite;
}
@keyframes rotateRight {
  0% {
    transform: rotate(0)
  }
  11% {
    transform: rotate(0)
  }
  12% {
    transform: rotate(22.5deg)
  }
  24% {
    transform: rotate(22.5deg)
  }
  25% {
    transform: rotate(45deg)
  }
  36% {
    transform: rotate(45deg)
  }
  37% {
    transform: rotate(67.5deg)
  }
  49% {
    transform: rotate(67.5deg)
  }
  50% {
    transform: rotate(90deg);
  }
  61% {
    transform: rotate(90deg);
  }
  62% {
    transform: rotate(112.5deg);
  }
  74% {
    transform: rotate(112.5deg);
  }
  75% {
    transform: rotate(135deg);
  }
  86% {
    transform: rotate(135deg);
  }
  87% {
    transform: rotate(157.5deg);
  }
  99% {
    transform: rotate(157.5deg);
  }
  100% {
    transform: rotate(180deg)
  }
}
@keyframes rotateLeft {
  0% {
    transform: rotate(0)
  }
  11% {
    transform: rotate(0)
  }
  12% {
    transform: rotate(-22.5deg)
  }
  24% {
    transform: rotate(-22.5deg)
  }
  25% {
    transform: rotate(-45deg)
  }
  36% {
    transform: rotate(-45deg)
  }
  37% {
    transform: rotate(-67.5deg)
  }
  49% {
    transform: rotate(-67.5deg)
  }
  50% {
    transform: rotate(-90deg);
  }
  61% {
    transform: rotate(-90deg);
  }
  62% {
    transform: rotate(-112.5deg);
  }
  74% {
    transform: rotate(-112.5deg);
  }
  75% {
    transform: rotate(-135deg);
  }
  86% {
    transform: rotate(-135deg);
  }
  87% {
    transform: rotate(-157.5deg);
  }
  99% {
    transform: rotate(-157.5deg);
  }
  100% {
    transform: rotate(-180deg)
  }
}
</style>