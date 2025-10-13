<template>
  <div id="app">
    <QrScanner />
  </div>
</template>

<script>
import QrScanner from './components/QrScanner.vue'

export default {
  name: 'App',
  components: {
    QrScanner
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f8f9fa;
  line-height: 1.6;
}

#app {
  min-height: 100vh;
  padding: 20px;
}
</style>