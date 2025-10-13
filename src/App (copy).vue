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
  background-color: #f0f2f5;
}

#app {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
}
</style>