<template>
  <div class="qr-scanner">
    <h1>Vehicle QR Code Scanner</h1>
    
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

    <!-- Vehicle ID Form -->
    <div class="form-container">
      <form @submit.prevent="lookupVehicle" class="vehicle-form">
        <div class="input-group">
          <label for="vehicleId">Vehicle ID:</label>
          <input 
            type="text" 
            id="vehicleId"
            v-model="vehicleId"
            placeholder="Scan QR code or enter manually"
            required
          />
        </div>
        <button type="submit" class="btn btn-primary" :disabled="loading">
          {{ loading ? 'Searching...' : 'Lookup Vehicle' }}
        </button>
        <button type="button" @click="clearAll" class="btn btn-secondary">
          Clear All
        </button>
      </form>
    </div>

    <!-- Loading State -->
    <div v-if="loading" class="loading">
      <p>Searching for vehicle information...</p>
    </div>

    <!-- Vehicle Information -->
    <div v-if="vehicleData" class="vehicle-card">
      <div class="vehicle-header">
        <h2>Vehicle Details</h2>
        <span class="status-badge">Found</span>
      </div>
      
      <div class="vehicle-info">
        <div class="info-section">
          <h3>Basic Information</h3>
          <div class="info-grid">
            <div class="info-item">
              <label>Vehicle ID:</label>
              <span class="info-value">{{ vehicleData.id }}</span>
            </div>
            <div class="info-item">
              <label>Make:</label>
              <span class="info-value">{{ vehicleData.make }}</span>
            </div>
            <div class="info-item">
              <label>Model:</label>
              <span class="info-value">{{ vehicleData.model }}</span>
            </div>
            <div class="info-item">
              <label>Year:</label>
              <span class="info-value">{{ vehicleData.year }}</span>
            </div>
            <div class="info-item">
              <label>Color:</label>
              <span class="info-value">{{ vehicleData.color }}</span>
            </div>
          </div>
        </div>

        <div class="info-section">
          <h3>Ownership & Insurance</h3>
          <div class="info-grid">
            <div class="info-item">
              <label>Owner:</label>
              <span class="info-value">{{ vehicleData.owner }}</span>
            </div>
            <div class="info-item">
              <label>Insurance:</label>
              <span class="info-value">{{ vehicleData.insurance }}</span>
            </div>
            <div class="info-item">
              <label>VIN:</label>
              <span class="info-value vin">{{ vehicleData.vin }}</span>
            </div>
            <div class="info-item">
              <label>Last Service:</label>
              <span class="info-value">{{ vehicleData.lastService }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Not Found Message -->
    <div v-if="notFound" class="not-found">
      <h3>Vehicle Not Found</h3>
      <p>No vehicle found with ID: "{{ vehicleId }}"</p>
      <p>Please check the ID and try again.</p>
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
      vehicleId: '',
      vehicleData: null,
      loading: false,
      notFound: false,
      error: null,
      camera: 'auto',
      hasCamera: true
    }
  },
  methods: {
    onDetect(detectedCodes) {
      if (detectedCodes.length > 0) {
        const content = detectedCodes[0].rawValue
        this.vehicleId = content
        console.log('QR Code detected:', content)
        
        // Auto-search after a short delay
        setTimeout(() => {
          this.lookupVehicle()
        }, 500)
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
    
    async lookupVehicle() {
      if (!this.vehicleId.trim()) {
        this.error = 'Please enter a Vehicle ID'
        return
      }

      this.loading = true
      this.error = null
      this.notFound = false
      this.vehicleData = null

      try {
        // Fetch the cars data from JSON file
        const response = await fetch('/cars.json')
        if (!response.ok) {
          throw new Error('Failed to load vehicle database')
        }

        const data = await response.json()
        
        // Find the vehicle by ID (case insensitive)
        const vehicle = data.cars.find(car => 
          car.id.toLowerCase() === this.vehicleId.trim().toLowerCase()
        )

        if (vehicle) {
          this.vehicleData = vehicle
          // Stop camera after successful scan to save resources
          this.camera = 'off'
        } else {
          this.notFound = true
        }
      } catch (err) {
        this.error = 'Error searching for vehicle: ' + err.message
        console.error('Lookup error:', err)
      } finally {
        this.loading = false
      }
    },
    
    clearAll() {
      this.vehicleId = ''
      this.vehicleData = null
      this.notFound = false
      this.error = null
      this.camera = 'auto'
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
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.camera-container {
  width: 100%;
  max-width: 500px;
  height: 300px;
  margin: 20px auto;
  border: 2px solid #ddd;
  border-radius: 12px;
  overflow: hidden;
  background-color: #f5f5f5;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.no-camera {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: #666;
  background-color: #f9f9f9;
}

.form-container {
  background: white;
  padding: 25px;
  border-radius: 12px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  margin: 25px 0;
}

.vehicle-form {
  display: flex;
  gap: 15px;
  align-items: end;
  flex-wrap: wrap;
}

.input-group {
  flex: 1;
  min-width: 250px;
}

.input-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #333;
}

.input-group input {
  width: 100%;
  padding: 12px;
  border: 2px solid #e1e5e9;
  border-radius: 8px;
  font-size: 16px;
  transition: border-color 0.3s;
}

.input-group input:focus {
  outline: none;
  border-color: #4CAF50;
}

.btn {
  padding: 12px 24px;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
}

.btn-primary {
  background-color: #4CAF50;
  color: white;
}

.btn-primary:hover:not(:disabled) {
  background-color: #45a049;
  transform: translateY(-1px);
}

.btn-secondary {
  background-color: #6c757d;
  color: white;
}

.btn-secondary:hover {
  background-color: #5a6268;
}

.btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
}

.loading {
  text-align: center;
  padding: 30px;
  color: #666;
}

.vehicle-card {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 16px;
  padding: 2px;
  margin: 25px 0;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}

.vehicle-info {
  background: white;
  border-radius: 14px;
  padding: 30px;
}

.vehicle-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 25px;
  padding-bottom: 15px;
  border-bottom: 2px solid #f0f0f0;
}

.vehicle-header h2 {
  color: #333;
  margin: 0;
}

.status-badge {
  background: #4CAF50;
  color: white;
  padding: 6px 12px;
  border-radius: 20px;
  font-size: 14px;
  font-weight: 600;
}

.info-section {
  margin-bottom: 30px;
}

.info-section h3 {
  color: #555;
  margin-bottom: 15px;
  font-size: 18px;
  border-left: 4px solid #4CAF50;
  padding-left: 12px;
}

.info-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 15px;
}

.info-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 0;
  border-bottom: 1px solid #f0f0f0;
}

.info-item label {
  font-weight: 600;
  color: #666;
}

.info-value {
  font-weight: 500;
  color: #333;
}

.info-value.vin {
  font-family: monospace;
  background: #f5f5f5;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 14px;
}

.not-found {
  background: #fff3cd;
  border: 1px solid #ffeaa7;
  border-radius: 12px;
  padding: 25px;
  text-align: center;
  margin: 25px 0;
}

.not-found h3 {
  color: #856404;
  margin-bottom: 10px;
}

.not-found p {
  color: #856404;
  margin: 5px 0;
}

.error {
  background: #f8d7da;
  border: 1px solid #f5c6cb;
  border-radius: 12px;
  padding: 20px;
  color: #721c24;
  margin: 20px 0;
}

h1 {
  text-align: center;
  color: #333;
  margin-bottom: 10px;
  font-size: 2.5em;
}

@media (max-width: 768px) {
  .vehicle-form {
    flex-direction: column;
  }
  
  .input-group {
    min-width: 100%;
  }
  
  .btn {
    width: 100%;
  }
  
  .info-grid {
    grid-template-columns: 1fr;
  }
  
  .vehicle-header {
    flex-direction: column;
    gap: 10px;
    align-items: start;
  }
}
</style>