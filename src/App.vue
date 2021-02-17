<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png" />
    <Timers />
  </div>
</template>

<script>
import Timers from "./components/Timers.vue";

export default {
  name: "App",
  components: {
    Timers
  }
};
</script>

<style lang="less">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
