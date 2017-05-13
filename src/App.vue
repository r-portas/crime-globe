<template>
  <div id="app">
    <globe></globe>
  </div>
</template>

<script>
import Globe from './components/Globe';

export default {
  name: 'app',
  components: {
    Globe,
  },
};
</script>

<style>

html, body, #app {
  height: 100%;
  width: 100%;
  margin: 0;
  padding: 0;
  border: 0;
}

</style>
