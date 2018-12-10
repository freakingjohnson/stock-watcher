<template>
  <div id="app">
    <StockWatcher/>
  </div>
</template>

<script>
import StockWatcher from "./components/StockWatcher.vue";

export default {
  name: "app",
  components: {
    StockWatcher
  }
};
</script>

<style>
#app {
  font-family: "Droid Serif";
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
body {
  background-color: lightgray;
  overflow-x: hidden;
}
</style>
