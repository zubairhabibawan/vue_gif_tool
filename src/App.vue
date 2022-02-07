<template>
  <div id="app">
    <Loafz />
  </div>
</template>

<script>
import Loafz from "./components/Loafz.vue";
export default {
  name: "App",
  components: {
    Loafz,
  },
  mounted() {},
};
</script>

<style>
@font-face {
  font-family: myCustomBodyFont;
  src: url("./assets/mvboli.ttf");
}
#app {
  font-family: myCustomBodyFont;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  font-size: 12px;
}
</style>
