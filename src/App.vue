<template>
  <div id="app">
    <Demo />
    <Demo2 />
  </div>
</template>

<script>
import Demo from "./Demo.vue";
import Demo2 from "./Demo2.vue";

export default {
  name: "App",
  components: {
    Demo,
    Demo2,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /* text-align: center; */
  color: #2c3e50;
  margin-top: 60px;
}
</style>
