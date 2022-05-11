<template>
  <div id="app">
    <ThreeboxExample />
  </div>
</template>

<script>
import ThreeboxExample from "./components/Threebox-project";

export default {
  name: "App",
  components: {
    ThreeboxExample,
  },
};
</script>

<style>
body,
html {
  width: 100%;
  height: 100%;
  margin: 0;
}

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 0;
  padding: 0;
  width: 100vw;
  height: 100vh;
}
</style>
