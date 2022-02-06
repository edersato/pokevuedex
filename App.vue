<template>
  <div id="app">
    <pokemon/>
  </div>
</template>

<script>
import Pokemon from './components/Pokemon.vue'


export default {
  name: "app",
  components: { 
    Pokemon 
  },
}
</script>

<style>
@import url("https://use.fontawesome.com/releases/v5.15.0/css/all.css");

body {
  margin: 0;
  padding: 0;
}

#app {
  background: linear-gradient( #fff, #3351fd);
}

</style>
