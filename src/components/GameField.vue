<template>
  <div ref="gameField" class="game-field relative h-full w-full max-w-[130vh] bg-zinc-800 text-gray-50 overflow-hidden">
    <GameFieldBackground />
    <GameFieldTimer :checkpoint="checkpoint" />
    <GameFieldGravityChangers @change-gravity="changePlayerGravity" :positionedPlayer="currentPlayerPosition" />
    <GameFieldPlayer 
      v-for="(player, index) in players" 
      :key="player.id"  
      @position-change="changePlayerPosition"
      @collision="player.collides = true"
      @freedom="player.collides = false"
      @changeGravityToCheckpoint="setPlayerGravityToCheckpoint"
      :index="index" 
      :gameFieldRECT="gameFieldRECT" 
      :player="player"
    />
    <div class="obstacles relative top-[Calc(20%_+_2rem)] h-[Calc(60%_-_4rem)] w-full">
      <div v-for="obstacle in obstacles" :key="obstacle.id" >
        <GameFieldObstacle 
          v-if="obstacle.spawned"
          @finish="unspawnObstacle"
          @fail-game="failGame"
          :fail="fail"
          :obstacle="obstacle"
          :gameFieldRECT="gameFieldRECT"
          :currentPlayerPosition="currentPlayerPosition"
        >
        </GameFieldObstacle>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch, reactive } from 'vue'
import GameFieldGravityChangers from './GameFieldGravityChangers.vue'
import GameFieldPlayer from './GameFieldPlayer.vue'
import GameFieldTimer from './GameFieldTimer.vue'
import GameFieldBackground from './GameFieldBackground.vue'
import GameFieldObstacle from './GameFieldObstacle.vue'
import type { Player, PlayerPosition } from '../types/Player.js'
import type { Controls } from '../types/Controls.js'
import type { Obstacle, ObstacleDirection, ObstacleRow } from '../types/Obstacle.js'
import type { Interval } from '../types/Interval.js'
import { getRandomFrom } from '../helpers'
import { useFail } from './../composables/useFail.js'
import { useGameTime } from './../composables/useGameTime.js'
const { fail, failGame } = useFail()
const { gameTimeInMiliseconds, start, pauseTime, timeInterval, halfSecond, fiveSeconds, second} = useGameTime()
const props = defineProps<{
  playerCount: number
}>()
const emit = defineEmits<{
  (e: 'end'): void
}>()
const obstacles = reactive<Obstacle[]>([
  {
    id: Math.random(),
    row: 1,
    spawnTime: 57000,
    direction: 'right',
    spawned: false
  }
])
const players = ref<Player[]>([])
const timeIntervalId = ref<Interval>(undefined)
const checkpoint = ref(gameTimeInMiliseconds.value)
const gameField = ref<HTMLDivElement | undefined>()
const gameFieldRECT = ref<DOMRect | undefined>()
const currentPlayerPosition = ref<PlayerPosition | undefined>()
const changePlayerPosition = (newPosition: PlayerPosition): void => {
  currentPlayerPosition.value = newPosition
  const player = players.value.find(player => player.id === newPosition.id)
  if(player){
    player.position = newPosition.position
    if(!player.checkpointPosition){
      player.checkpointPosition = newPosition.position
    }
  }
}
const changePlayerGravity = (id: number): void => {
  const collidedPlayer: Player | undefined = players.value.find(player => player.id === id)
  if(collidedPlayer){
    if(collidedPlayer.gravity === 'down'){
      collidedPlayer.gravity = 'up'
    }
    else {
      collidedPlayer.gravity = 'down'
    }
  }
}
const setTimer = () => {
  timeIntervalId.value = setInterval(() => {
    gameTimeInMiliseconds.value -= timeInterval
  }, timeInterval)
}
const revertToCheckpoint = () => {
  clearInterval(timeIntervalId.value)
  const showFullRevertTime = halfSecond //just for UI
  const revert: number = checkpoint.value - gameTimeInMiliseconds.value
  const revertIntervalValue: number = Math.ceil(revert / ((pauseTime - showFullRevertTime) / timeInterval))
  //1200 ms equally distributed in 2 seconds
  let revertIntervalId = setInterval(() => {
    gameTimeInMiliseconds.value += revertIntervalValue
  }, timeInterval)
  setTimeout(() => {
    clearInterval(revertIntervalId)
    gameTimeInMiliseconds.value = checkpoint.value
  }, pauseTime - showFullRevertTime)
  setTimeout(() => {
    setTimer()
  }, pauseTime)
}
const setPlayersCheckpoint = () => {
  players.value.forEach(player => {
    player.checkpointPosition = player.position
    player.checkpointGravity = player.gravity
  })
}
const setPlayerGravityToCheckpoint = (player: Player) => {
  if(player.checkpointGravity){
    player.gravity = player.checkpointGravity
  }
}
const createObstacle = (previousObstacles?: Obstacle[]): Obstacle => {
  const possibleRows: ObstacleRow[] = [0, 1, 2, 3, 4, 5]
  const possibleDirections: ObstacleDirection[] = ['left', 'right']
  let row = getRandomFrom(possibleRows)
  const direction: ObstacleDirection = getRandomFrom(possibleDirections)
  if(!previousObstacles){
    row = getRandomFrom(possibleRows)
  }
  else {
    const usedRows = previousObstacles.map(obstacle => obstacle.row)
    const unusedRows = possibleRows.filter(row => !usedRows.includes(row))
    row = getRandomFrom(unusedRows)
  }
  return {
    id: Math.random(),
    spawnTime: gameTimeInMiliseconds.value,
    row,
    direction,
    spawned: false
  }
}
const spawnObstacles = () => {
  const obstaclesToSpawn = obstacles.filter(obstacle => obstacle.spawnTime === gameTimeInMiliseconds.value)
  if(obstaclesToSpawn){
    obstaclesToSpawn.forEach(obstacle => obstacle.spawned = true)
  }
}
const unspawnAllObstacles = () => {
  obstacles.forEach(obstacle => {
    obstacle.spawned = false
  })
}
const unspawnObstacle = (id: number) => { 
  const obstacleToUnspawn = obstacles.find(obstacle => obstacle.id === id)
  if(obstacleToUnspawn){
    obstacleToUnspawn.spawned = false
  }
}
watch(props, () => {
  const playerControls: Controls[] = [
    //this might not be the best and passing down players directly seems better
    //but refactoring most of the code here would be neccessary and I dont see many issues with this approach
    {
      left: 'a',
      right: 'd'
    },
    {
      left: 'ArrowLeft',
      right: 'ArrowRight'
    },
    {
      left: 'j',
      right: 'l'
    }
  ]
  players.value = []
  for(let i = 0; i < props.playerCount; i++){
    const player: Player = {
      id: Math.random(),
      controls: playerControls[i],
      gravity: 'down',
      collides: false,
      position: null,
      checkpointGravity: null,
      checkpointPosition: null
    }
    players.value.push(player)
  }
}, { immediate: true })
watch(gameTimeInMiliseconds, () => {
  if(gameTimeInMiliseconds.value % fiveSeconds === 0 && gameTimeInMiliseconds.value > 0){
    if(!fail.value){
      checkpoint.value = gameTimeInMiliseconds.value
      setPlayersCheckpoint()
    }
    const isSet = obstacles.find(obstacle => obstacle.spawnTime === gameTimeInMiliseconds.value)
    if(!isSet){
      const obstacle = createObstacle()
      const secondObstacle = createObstacle([obstacle])
      const thirdObstacle = createObstacle([obstacle, secondObstacle])
      obstacles.push(obstacle, secondObstacle, thirdObstacle)
    }
  }
  else if(gameTimeInMiliseconds.value % second === 0 && gameTimeInMiliseconds.value > second){
    const isSet = obstacles.find(obstacle => obstacle.spawnTime === gameTimeInMiliseconds.value)
    if(!isSet){
      const obstacle = createObstacle()
      obstacles.push(obstacle)
    }
  }
  if(start.value){
    spawnObstacles()
  }
  if(gameTimeInMiliseconds.value <= 0){
    setTimeout(() => {
      emit('end')
    }, second)
    clearInterval(timeIntervalId.value)
  }
},{ immediate: true })
watch(start, () => {
  if(start.value){
    setTimer()
    setPlayersCheckpoint()
    spawnObstacles()
  }
})
watch(fail, () => {
  if(fail.value){
    revertToCheckpoint()
    setTimeout(() => {
      unspawnAllObstacles()
    }, halfSecond)
  }
  else {
    spawnObstacles()
  }
})
onMounted(() => {
  gameFieldRECT.value = gameField.value?.getBoundingClientRect()
  window.addEventListener('resize', () => {
    gameFieldRECT.value = gameField.value?.getBoundingClientRect()
  })
  setTimeout(() => {
    start.value = true
  }, 3500)
})
</script>

<style scoped>
</style>