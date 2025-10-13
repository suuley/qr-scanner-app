<template>
    <div class="qr-scanner">
      <h1>QR Code Scanner</h1>
      
      <!-- Camera Stream -->
      <div class="camera-container">
        <qrcode-stream 
          @detect="onDetect"
          @error="onError"
          :camera="camera"
          v-if="hasCamera"
        ></qrcode-stream>
        
        <div v-else class="no-camera">
          <p>Camera not available or access denied</p>
        </div>
      </div>
  
      <!-- Controls -->
      <div class="controls">
        <button @click="toggleCamera" class="btn">
          {{ camera === 'auto' ? 'Stop Camera' : 'Start Camera' }}
        </button>
        <button @click="clearResult" class="btn" v-if="result">
          Clear Result
        </button>
      </div>
  
      <!-- Results -->
      <div v-if="result" class="result">
        <h3>Scanned Content:</h3>
        <div class="result-content">
          {{ result }}
        </div>
      </div>
  
      <!-- Error Message -->
      <div v-if="error" class="error">
        <p>{{ error }}</p>
      </div>
    </div>
  </template>
  
  <script>
  import { QrcodeStream } from 'vue-qrcode-reader'
  
  export default {
    name: 'QrScanner',
    components: {
      QrcodeStream
    },
    data() {
      return {
        result: null,
        error: null,
        camera: 'auto',
        hasCamera: true
      }
    },
    methods: {
      onDetect(detectedCodes) {
        if (detectedCodes.length > 0) {
          const content = detectedCodes[0].rawValue
          this.result = content
          console.log('QR Code detected:', content)
          
          // Optional: Stop camera after successful scan
          // this.camera = 'off'
        }
      },
      
      onError(error) {
        console.error('QR Scanner error:', error)
        
        if (error.name === 'NotAllowedError') {
          this.error = 'Camera access denied. Please allow camera permissions.'
          this.hasCamera = false
        } else if (error.name === 'NotFoundError') {
          this.error = 'No camera found on this device.'
          this.hasCamera = false
        } else if (error.name === 'NotSupportedError') {
          this.error = 'This page is not served over HTTPS. Camera access requires HTTPS.'
          this.hasCamera = false
        } else if (error.name === 'NotReadableError') {
          this.error = 'Camera is already in use by another application.'
          this.hasCamera = false
        } else {
          this.error = 'An unexpected error occurred: ' + error.message
        }
      },
      
      toggleCamera() {
        if (this.camera === 'auto') {
          this.camera = 'off'
        } else {
          this.camera = 'auto'
          this.error = null
          this.result = null
        }
      },
      
      clearResult() {
        this.result = null
      }
    },
    
    mounted() {
      // Check if the browser supports the required APIs
      if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
        this.error = 'Your browser does not support camera access.'
        this.hasCamera = false
      }
    }
  }
  </script>
  
  <style scoped>
  .qr-scanner {
    max-width: 600px;
    margin: 0 auto;
    padding: 20px;
    text-align: center;
    font-family: Arial, sans-serif;
  }
  
  .camera-container {
    width: 100%;
    max-width: 500px;
    height: 300px;
    margin: 20px auto;
    border: 2px solid #ddd;
    border-radius: 8px;
    overflow: hidden;
    background-color: #f5f5f5;
  }
  
  .no-camera {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100%;
    color: #666;
  }
  
  .controls {
    margin: 20px 0;
  }
  
  .btn {
    background-color: #4CAF50;
    color: white;
    border: none;
    padding: 10px 20px;
    margin: 0 10px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
  }
  
  .btn:hover {
    background-color: #45a049;
  }
  
  .result {
    margin-top: 20px;
    padding: 15px;
    background-color: #e8f5e8;
    border: 1px solid #4CAF50;
    border-radius: 4px;
  }
  
  .result-content {
    word-break: break-all;
    font-family: monospace;
    background-color: white;
    padding: 10px;
    border-radius: 4px;
    margin-top: 10px;
  }
  
  .error {
    margin-top: 20px;
    padding: 15px;
    background-color: #ffebee;
    border: 1px solid #f44336;
    border-radius: 4px;
    color: #c62828;
  }
  
  h1 {
    color: #333;
  }
  
  h3 {
    color: #555;
    margin-bottom: 10px;
  }
  </style>