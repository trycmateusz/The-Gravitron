<template>
  <div class="absolute w-full h-full bg-white">
    <div class="background h-full"></div>
  </div>
</template>

<script setup lang="ts">

</script>

<style lang="scss" scoped>
.background {
  background-image: url('./../../zigzag.svg');
  background-repeat: repeat;
  animation: move 4s linear infinite;
}
@keyframes move {
  from {
    background-position: 0
  }
  to {
    background-position: -779px
  }
}
</style>