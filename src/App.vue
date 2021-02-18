<template>
  <div id="app">
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
body {
  background: #8bb5b6;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  width: auto;
  height: auto;
}
</style>
