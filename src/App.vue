<!--
 * @Author: caixin caixin185@163.com
 * @Date: 2023-04-08 10:52:11
 * @LastEditors: caixin
 * @LastEditTime: 2023-04-09 20:40:41
 * @Description: file content
-->
<script setup lang="ts">
import HelloWorld from './components/HelloWorld.vue'
import ThreeSample from './components/ThreeSample.vue'
import ThreeSampleShadow from './components/ThreeSampleShadow.vue'
import FishGame from './components/FishGame.vue';

</script>

<template>
  <div>
    <!-- <a href="https://vitejs.dev" target="_blank">
      <img src="/vite.svg" class="logo" alt="Vite logo" />
    </a>
    <a href="https://vuejs.org/" target="_blank">
      <img src="./assets/vue.svg" class="logo vue" alt="Vue logo" />
    </a>
    <HelloWorld msg="Vite + Vue" />
    <ThreeSample />
    <ThreeSampleShadow/>  -->
    <FishGame/> 
  </div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
