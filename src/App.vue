<template>
  <div id="app">
    <ImageSubtitle />
  </div>
</template>

<script>
import ImageSubtitle from './components/ImageSubtitle.vue';

export default {
  name: 'App',
  components: {
    ImageSubtitle,
  },
};
</script>

<style>
body {
  margin: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  background-image: url('@/assets/background.png');
  background-size: cover;
  background-repeat: no-repeat;
  background-position: center;
}
</style>
