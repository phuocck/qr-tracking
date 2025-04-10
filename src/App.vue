<script>
import { Html5QrcodeScanner } from 'html5-qrcode'

export default {
  name: 'App',
  data() {
    return {
      trackedCodes: [],
      scannedCode: '',
      newCode: '',
      scanHistory: [],
      audio: new Audio('/beep.mp3'),
      scanTimeout: null,
      lastScanResult: null,
      scanner: null,
      isScanning: false
    }
  },
  mounted() {
    // Tự động focus vào input khi component được mount
    this.$refs.scannerInput.focus()
    
    // Thêm event listener cho input để xử lý máy quét mã vạch
    this.$refs.scannerInput.addEventListener('keydown', this.handleKeyDown)
  },
  beforeUnmount() {
    // Cleanup event listener
    this.$refs.scannerInput.removeEventListener('keydown', this.handleKeyDown)
    if (this.scanner) {
      this.scanner.clear()
    }
  },
  methods: {
    handleKeyDown(event) {
      // Nếu là phím Enter
      if (event.key === 'Enter') {
        event.preventDefault();
        this.checkCode();
      }
    },
    startScanner() {
      if (this.scanner) {
        this.scanner.clear();
      }

      this.scanner = new Html5QrcodeScanner(
        "qr-reader",
        { 
          fps: 10,
          qrbox: { width: 250, height: 250 },
          aspectRatio: 1.0
        }
      );

      this.scanner.render((decodedText) => {
        this.scannedCode = decodedText;
        this.checkCode();
      }, (error) => {
        console.warn(`QR Code scanning failed: ${error}`);
      });

      this.isScanning = true;
    },
    stopScanner() {
      if (this.scanner) {
        this.scanner.clear();
        this.scanner = null;
      }
      this.isScanning = false;
    },
    checkCode() {
      if (!this.scannedCode) return;

      const matched = this.trackedCodes.includes(this.scannedCode);

      // Thêm vào lịch sử
      this.scanHistory.unshift({
        code: this.scannedCode,
        timestamp: new Date().toLocaleTimeString(),
        matched
      });

      // Cập nhật kết quả quét cuối cùng
      this.lastScanResult = {
        matched,
        message: matched ? 'Mã trùng khớp!' : 'Mã không trùng khớp!'
      };

      const url = matched ? 'correct.wav' : 'wrong.mp3';
      // Phát âm thanh dựa trên kết quả
      const audio = new Audio(url);
      audio.play();

      // Reset input và focus lại
      this.scannedCode = '';
      if (!this.isScanning) {
        this.$refs.scannerInput.focus();
      }

      // Xóa thông báo sau 3 giây
      setTimeout(() => {
        this.lastScanResult = null;
      }, 3000);
    },
    addCode() {
      if (this.newCode && !this.trackedCodes.includes(this.newCode)) {
        this.trackedCodes.push(this.newCode);
        this.newCode = '';
      }
    }
  }
}
</script>

<template>
  <div class="container">
    <header class="app-header">
      <h1>QR Code Scanner</h1>
    </header>
    
    <main class="main-content">
      <div class="scanner-section">
        <div id="qr-reader"></div>
        <div class="scanner-controls">
          <button v-if="!isScanning" @click="startScanner" class="scanner-btn">
            <i class="fas fa-camera"></i> Bật Camera
          </button>
          <button v-else @click="stopScanner" class="scanner-btn stop">
            <i class="fas fa-stop"></i> Tắt Camera
          </button>
        </div>
      </div>

      <div class="input-section">
        <input 
          type="text" 
          v-model="scannedCode" 
          @keyup.enter="checkCode"
          placeholder="Quét mã vạch hoặc nhập mã..."
          ref="scannerInput"
        />
        <div v-if="lastScanResult" :class="['scan-result', lastScanResult.matched ? 'matched' : 'not-matched']">
          <i :class="lastScanResult.matched ? 'fas fa-check-circle' : 'fas fa-times-circle'"></i>
          {{ lastScanResult.message }}
        </div>
      </div>

      <div class="lists-section">
        <div class="tracked-codes card">
          <div class="card-header">
            <h2><i class="fas fa-list"></i> Danh sách mã cần theo dõi</h2>
            <div class="add-code">
              <input 
                type="text" 
                v-model="newCode" 
                @keyup.enter="addCode"
                placeholder="Thêm mã mới..."
              />
              <button @click="addCode" class="add-btn">
                <i class="fas fa-plus"></i>
              </button>
            </div>
          </div>
          <div class="code-list">
            <div v-for="(code, index) in trackedCodes" :key="index" class="code-item">
              <i class="fas fa-barcode"></i>
              {{ code }}
            </div>
          </div>
        </div>

        <div class="scanned-history card">
          <div class="card-header">
            <h2><i class="fas fa-history"></i> Lịch sử quét</h2>
          </div>
          <div class="history-list">
            <div v-for="(item, index) in scanHistory" :key="index" 
                 :class="['history-item', item.matched ? 'matched' : 'not-matched']">
              <i :class="item.matched ? 'fas fa-check' : 'fas fa-times'"></i>
              <span class="code">{{ item.code }}</span>
              <span class="timestamp">{{ item.timestamp }}</span>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500;700&display=swap');
@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.4/css/all.min.css');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Roboto', sans-serif;
  background-color: #f5f5f5;
  color: #333;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.app-header {
  text-align: center;
  margin-bottom: 30px;
  padding: 20px;
  background: linear-gradient(135deg, #4CAF50, #45a049);
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.app-header h1 {
  color: white;
  font-size: 2.5em;
  font-weight: 500;
}

.main-content {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.scanner-section {
  background: white;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

#qr-reader {
  width: 100%;
  max-width: 500px;
  margin: 0 auto;
  border-radius: 10px;
  overflow: hidden;
}

.scanner-controls {
  margin-top: 15px;
  text-align: center;
}

.scanner-btn {
  padding: 12px 24px;
  font-size: 16px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  transition: all 0.3s ease;
  display: inline-flex;
  align-items: center;
  gap: 8px;
}

.scanner-btn.stop {
  background-color: #f44336;
}

.scanner-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
}

.input-section {
  background: white;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.input-section input {
  width: 100%;
  padding: 12px;
  font-size: 16px;
  border: 2px solid #ddd;
  border-radius: 25px;
  transition: all 0.3s ease;
}

.input-section input:focus {
  border-color: #4CAF50;
  outline: none;
  box-shadow: 0 0 0 3px rgba(76,175,80,0.1);
}

.scan-result {
  margin-top: 10px;
  padding: 12px;
  border-radius: 25px;
  text-align: center;
  font-weight: 500;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.scan-result.matched {
  background-color: #e8f5e9;
  color: #2e7d32;
}

.scan-result.not-matched {
  background-color: #ffebee;
  color: #c62828;
}

.lists-section {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

.card {
  background: white;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  overflow: hidden;
}

.card-header {
  padding: 15px 20px;
  background: #f8f9fa;
  border-bottom: 1px solid #eee;
}

.card-header h2 {
  font-size: 1.2em;
  font-weight: 500;
  color: #333;
  display: flex;
  align-items: center;
  gap: 8px;
}

.code-list, .history-list {
  max-height: 300px;
  overflow-y: auto;
  padding: 10px;
}

.code-item, .history-item {
  padding: 12px;
  border-bottom: 1px solid #eee;
  display: flex;
  align-items: center;
  gap: 8px;
  transition: background-color 0.3s ease;
}

.code-item:hover, .history-item:hover {
  background-color: #f8f9fa;
}

.history-item.matched {
  background-color: #e8f5e9;
  color: #2e7d32;
}

.history-item.not-matched {
  background-color: #ffebee;
  color: #c62828;
}

.history-item .code {
  flex: 1;
}

.history-item .timestamp {
  font-size: 0.9em;
  color: #666;
}

.add-code {
  margin-top: 10px;
  display: flex;
  gap: 10px;
}

.add-code input {
  flex: 1;
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 20px;
  transition: all 0.3s ease;
}

.add-code input:focus {
  border-color: #4CAF50;
  outline: none;
  box-shadow: 0 0 0 3px rgba(76,175,80,0.1);
}

.add-btn {
  padding: 8px 16px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.add-btn:hover {
  background-color: #45a049;
  transform: translateY(-2px);
}

@media (max-width: 768px) {
  .lists-section {
    grid-template-columns: 1fr;
  }
  
  .app-header h1 {
    font-size: 2em;
  }
  
  .container {
    padding: 10px;
  }
}
</style>
