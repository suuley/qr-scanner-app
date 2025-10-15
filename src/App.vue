<!-- src/App.vue -->
<template>
  <div id="app">
    <!-- Show scanner only after user clicks "Start Scan" -->
    <QrScanner v-if="showScanner" @close="showScanner = false" />
    
    <!-- Start Screen -->
    <div v-else class="start-screen">
      <h1>Vehicle QR Code Scanner</h1>
      <p>Scan a vehicle QR code to view details instantly</p>
      <button @click="startScanning" class="btn-start">
        Start Scanning
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import QrScanner from './components/QrScanner.vue'

const showScanner = ref(false)

const startScanning = () => {
  showScanner.value = true
}
</script>

<style scoped>
.start-screen {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  padding: 20px;
  background: #f8f9fa;
  text-align: center;
}

.start-screen h1 {
  font-size: 2.2em;
  color: #333;
  margin-bottom: 16px;
}

.start-screen p {
  color: #666;
  margin-bottom: 32px;
  font-size: 1.1em;
  max-width: 500px;
}

.btn-start {
  padding: 14px 36px;
  font-size: 18px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  transition: background 0.3s;
}

.btn-start:hover {
  background: #45a049;
}
</style>