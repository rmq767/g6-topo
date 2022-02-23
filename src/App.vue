<template>
  <div id="app">
    <topo />
  </div>
</template>

<script>
import topo from "./components/topo.vue";

export default {
  name: "App",
  components: {
    topo,
  },
};
</script>

<style>
html,
body,
#app {
  margin: 0;
  width: 100%;
  height: 100%;
}
</style>
