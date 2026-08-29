<script setup>
import { ref, onMounted, onBeforeUnmount, watch } from 'vue';

const props = defineProps({
  events: {
    type: Array,
    required: true
  },
  selectedEvent: {
    type: Object,
    default: null
  }
});

const emit = defineEmits(['update-event', 'close-scanner']);

const currentEvent = ref(props.selectedEvent || (props.events.length > 0 ? props.events[0] : null));

watch(() => props.selectedEvent, (newEv) => {
  if (newEv) {
    currentEvent.value = newEv;
  }
});

// Setup Lottie Player script if needed
onMounted(() => {
  if (!window.LottiePlayer && !document.getElementById('lottie-player-script')) {
    const script = document.createElement('script');
    script.id = 'lottie-player-script';
    script.src = 'https://unpkg.com/@lottiefiles/lottie-player@latest/dist/lottie-player.js';
    document.head.appendChild(script);
  }
  startCamera();
});

// Scan Results Popup Notification state
const scanPopupVisible = ref(false);
const isConfirmed = ref(false);
const scanResultStatus = ref('success'); // 'success', 'failed', 'already'
const scanData = ref({
  invoice: 'INV-88419',
  name: 'Ahmad Fauzi',
  ticketType: 'E-Ticket',
  time: ''
});

let popupTimer = null;

const triggerScan = (status, rawData = null) => {
  scanResultStatus.value = status;
  isConfirmed.value = false;
  scanPopupVisible.value = true;
  
  const tType = ticketCategory.value === 'invitation' ? 'Invitation' : 'E-Ticket';
  const tCode = rawData ? rawData.substring(0, 10).toUpperCase() : `INV-${Math.floor(10000 + Math.random() * 90000)}`;

  const names = ['Ahmad Fauzi', 'Budi Santoso', 'Citra Kirana', 'Dewi Lestari', 'Eko Prasetyo', 'Fina Melati'];
  let sum = 0;
  for(let i=0; i<tCode.length; i++) sum += tCode.charCodeAt(i);
  const buyerName = names[sum % names.length];

  scanData.value = {
    invoice: tCode,
    name: status === 'failed' ? 'Tidak Terdaftar' : buyerName,
    ticketType: tType,
    time: new Date().toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' }) + ' WIB'
  };
  
  if (popupTimer) clearTimeout(popupTimer);
  popupTimer = setTimeout(() => {
    scanPopupVisible.value = false;
  }, 15000);
};

const closePopup = () => {
  scanPopupVisible.value = false;
  if (popupTimer) clearTimeout(popupTimer);
  if (!isManualMode()) {
    if (animationFrameId) clearTimeout(animationFrameId);
    animationFrameId = setTimeout(scanQRCode, 100);
  }
};

const confirmCheckin = () => {
  if (scanResultStatus.value === 'success' && !isConfirmed.value) {
    isConfirmed.value = true;
    const activeEv = props.selectedEvent || currentEvent.value;
    if (activeEv && activeEv.sold < activeEv.total) {
      activeEv.sold++;
    }
    if (popupTimer) clearTimeout(popupTimer);
    popupTimer = setTimeout(() => {
      closePopup();
    }, 15000);
  } else {
    closePopup();
  }
};

// Camera Scanner View State
const videoElement = ref(null);
let videoStream = null;
const checkinMode = ref('qr'); // 'qr' | 'manual'
const ticketCategory = ref('e-ticket'); // 'e-ticket' | 'invitation'
const manualTicketCode = ref('');

// Derive ticket category from mode
const isManualMode = () => checkinMode.value === 'manual';
const isQRMode = () => checkinMode.value !== 'manual';

// jsQR dynamic script loader
let jsScriptLoaded = false;
const loadJsQR = () => {
  return new Promise((resolve, reject) => {
    if (window.jsQR || jsScriptLoaded) {
      resolve();
      return;
    }
    const script = document.createElement('script');
    script.src = 'https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.min.js';
    script.onload = () => {
      jsScriptLoaded = true;
      resolve();
    };
    script.onerror = reject;
    document.head.appendChild(script);
  });
};

const cameraErrorPopupVisible = ref(false);
let canvasElement = null;
let canvasCtx = null;
let animationFrameId = null;

const scanQRCode = () => {
  if (videoElement.value && videoElement.value.readyState === videoElement.value.HAVE_ENOUGH_DATA) {
    if (!canvasElement) {
      canvasElement = document.createElement('canvas');
      canvasCtx = canvasElement.getContext('2d', { willReadFrequently: true });
    }
    canvasElement.width = videoElement.value.videoWidth;
    canvasElement.height = videoElement.value.videoHeight;
    canvasCtx.drawImage(videoElement.value, 0, 0, canvasElement.width, canvasElement.height);
    const imageData = canvasCtx.getImageData(0, 0, canvasElement.width, canvasElement.height);
    const code = window.jsQR ? window.jsQR(imageData.data, imageData.width, imageData.height, {
      inversionAttempts: 'attemptBoth'
    }) : null;
    
    if (code) {
      const qrData = code.data;
      
      const statuses = ['success', 'failed', 'already'];
      let hash = 0;
      for (let i = 0; i < qrData.length; i++) {
        hash = qrData.charCodeAt(i) + ((hash << 5) - hash);
      }
      const statusIndex = Math.abs(hash) % statuses.length;
      const status = statuses[statusIndex];
      
      triggerScan(status, qrData);
      
      if (navigator.vibrate) {
        navigator.vibrate(200);
      }
      return;
    }
  }
  
  if (checkinMode.value === 'qr' && !scanPopupVisible.value) {
    animationFrameId = setTimeout(scanQRCode, 100);
  }
};

const startCamera = async () => {
  try {
    cameraErrorPopupVisible.value = false;
    await loadJsQR();
    const stream = await navigator.mediaDevices.getUserMedia({
      video: { facingMode: 'environment' }
    });
    if (videoElement.value) {
      videoElement.value.srcObject = stream;
      videoStream = stream;
      if (animationFrameId) clearTimeout(animationFrameId);
      animationFrameId = setTimeout(scanQRCode, 100);
    }
  } catch (err) {
    console.error("Camera access blocked or error:", err);
    cameraErrorPopupVisible.value = true;
  }
};

const retryCamera = () => {
  cameraErrorPopupVisible.value = false;
  startCamera();
};

const stopCamera = () => {
  cameraErrorPopupVisible.value = false;
  if (animationFrameId) {
    clearTimeout(animationFrameId);
    animationFrameId = null;
  }
  if (videoStream) {
    videoStream.getTracks().forEach(track => track.stop());
    videoStream = null;
  }
  if (videoElement.value) {
    videoElement.value.srcObject = null;
  }
};

// Flashlight toggle control
const isFlashlightOn = ref(false);
watch(isFlashlightOn, async (on) => {
  if (videoStream && isQRMode()) {
    const track = videoStream.getVideoTracks()[0];
    if (track) {
      try {
        await track.applyConstraints({
          advanced: [{ torch: on }]
        });
      } catch (err) {
        console.warn("Torch / Flashlight not supported on this device/browser constraints:", err);
      }
    }
  }
});

// Watch mode switch to start/stop camera stream dynamically
watch(checkinMode, (newMode) => {
  if (newMode !== 'manual') {
    startCamera();
  } else {
    stopCamera();
    isFlashlightOn.value = false;
  }
});

// Viewfinder tap simulation helper
const handleViewfinderTap = () => {
  const statuses = ['success', 'failed', 'already'];
  const randomStatus = statuses[Math.floor(Math.random() * statuses.length)];
  triggerScan(randomStatus);
  if (navigator.vibrate) {
    navigator.vibrate(200);
  }
};

// Manual Input Checker trigger
const handleManualCheckin = () => {
  const code = manualTicketCode.value.trim();
  if (!code) return;
  
  const statuses = ['success', 'failed', 'already'];
  let hash = 0;
  for (let i = 0; i < code.length; i++) {
    hash = code.charCodeAt(i) + ((hash << 5) - hash);
  }
  const statusIndex = Math.abs(hash) % statuses.length;
  const status = statuses[statusIndex];
  
  triggerScan(status, code);
  manualTicketCode.value = '';
  if (navigator.vibrate) {
    navigator.vibrate(200);
  }
};

onBeforeUnmount(() => {
  stopCamera();
});
</script>

<template>
  <div class="scanner-page">
    <div class="scanner-fullscreen-container" :class="{ 'manual-bg': checkinMode === 'manual' }">
      <!-- Real Camera Feed Video Element -->
      <video v-show="isQRMode()" ref="videoElement" autoplay playsinline class="scanner-camera-feed"></video>
      
      <!-- Scanner Top Bar -->
      <div class="scanner-top-bar">
        <button
          class="scanner-icon-btn close-btn"
          :class="{ 'manual-mode': checkinMode === 'manual' }"
          @click="emit('close-scanner')"
        >
          <!-- Close X Icon -->
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" class="scanner-svg-icon">
            <line x1="18" y1="6" x2="6" y2="18"></line>
            <line x1="6" y1="6" x2="18" y2="18"></line>
          </svg>
        </button>

        <div class="scanner-header-title">
          <div class="marquee-wrap">
            <span class="marquee-track" :class="{ 'marquee-animate': ((selectedEvent ? selectedEvent.title : 'All Event') + ' - ' + (ticketCategory === 'invitation' ? 'Invitation' : 'E-Ticket')).length > 18 }">
              {{ selectedEvent ? selectedEvent.title : 'All Event' }} - {{ ticketCategory === 'invitation' ? 'Invitation' : 'E-Ticket' }}
              <template v-if="((selectedEvent ? selectedEvent.title : 'All Event') + ' - ' + (ticketCategory === 'invitation' ? 'Invitation' : 'E-Ticket')).length > 18">
                &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                {{ selectedEvent ? selectedEvent.title : 'All Event' }} - {{ ticketCategory === 'invitation' ? 'Invitation' : 'E-Ticket' }}
              </template>
            </span>
          </div>
          <p>{{ selectedEvent ? selectedEvent.location : 'Semua Lokasi' }}</p>
        </div>
        
        <!-- Flash button on right -->
        <button v-show="isQRMode()" class="scanner-icon-btn flash-btn" :class="{ active: isFlashlightOn }" @click="isFlashlightOn = !isFlashlightOn">
          <!-- Flash Bolt Icon -->
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" class="scanner-svg-icon" :style="{ fill: isFlashlightOn ? '#F5C453' : 'none', stroke: isFlashlightOn ? '#F5C453' : 'currentColor' }">
            <polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2" />
          </svg>
        </button>
        <!-- Spacer when flash hidden (manual mode) to keep title centered -->
        <div v-show="!isQRMode()" style="width:40px;"></div>
      </div>
      
      <!-- Check-in Mode Selector Tab Toggle -->
      <div class="checkin-mode-toggle">
        <button class="toggle-btn" :class="{ active: checkinMode === 'qr' }" @click="checkinMode = 'qr'">
          Scan-QR
        </button>
        <button class="toggle-btn" :class="{ active: checkinMode === 'manual' }" @click="checkinMode = 'manual'">
          Scanner
        </button>
      </div>
      
      <!-- Scanner Middle Body Content Area -->
      <div class="scanner-body">
        <!-- Scanning Window Area (QR Mode: Reguler & Invitation) -->
        <div v-if="isQRMode()" class="scanner-viewfinder" @click="handleViewfinderTap">
          <div class="scanner-laser-trail"></div>
          <div class="scanner-laser-line"></div>
        </div>
        
        <!-- Manual Input Card (Manual Mode) -->
        <div v-if="checkinMode === 'manual'" class="manual-checkin-container">
          <div class="manual-form-card">
            <div class="manual-field-group">
              <label class="manual-label">Kode Tiket / Invoice</label>
              <input 
                v-model="manualTicketCode" 
                type="text" 
                placeholder="Masukkan kode tiket" 
                class="manual-input-field"
                @keyup.enter="handleManualCheckin"
              />
              <p class="manual-desc-mockup">Masukkan nomor invoice atau kode tiket untuk diproses scanner</p>
            </div>
            
            <button class="manual-submit-btn" @click="handleManualCheckin">
              Check-in Tiket
            </button>
            
            <div class="manual-tips">
              <span>💡 <strong>Tips Simulasi:</strong> Ketik kata "gagal" untuk simulasi gagal, atau "sudah" untuk simulasi tiket terpakai.</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Ticket Category Bottom Bar -->
      <div class="ticket-category-bar">
        <div class="category-buttons-row">
          <button class="sheet-category-btn" :class="{ active: ticketCategory === 'e-ticket' }" @click="ticketCategory = 'e-ticket'">E-Ticket</button>
          <button class="sheet-category-btn" :class="{ active: ticketCategory === 'invitation' }" @click="ticketCategory = 'invitation'">Invitation</button>
        </div>
      </div>

      <!-- Bottom Total Audience Progress Bar (Only visible when specific event is selected) -->
      <div class="scanner-bottom-stats" v-if="selectedEvent">
        <div class="stats-row">
          <span class="stats-label">TOTAL MASUK</span>
          <span class="stats-val">{{ selectedEvent.sold }} / {{ selectedEvent.total }} Penonton</span>
        </div>
        <div class="stats-progress-container">
          <div class="stats-progress-track">
            <div class="stats-progress-fill" :style="{ width: `${(selectedEvent.sold / selectedEvent.total) * 100}%` }"></div>
          </div>
        </div>
      </div>

      <!-- Sliding Notification Popups -->
      <transition name="popup-slide">
        <div v-if="scanPopupVisible" class="notification-popup-card">
          <div class="notification-header" :class="scanResultStatus === 'success' ? (isConfirmed ? 'success' : 'pending') : scanResultStatus">
            <div class="header-icon">
              <svg v-if="scanResultStatus === 'success' && !isConfirmed" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="status-svg">
                <path fill-rule="evenodd" d="M12 2.25c-5.385 0-9.75 4.365-9.75 9.75s4.365 9.75 9.75 9.75 9.75-4.365 9.75-9.75S17.385 2.25 12 2.25zM12.75 6a.75.75 0 00-1.5 0v6c0 .414.336.75.75.75h4.5a.75.75 0 000-1.5h-3.75V6z" clip-rule="evenodd" />
              </svg>
              <svg v-else-if="scanResultStatus === 'success' && isConfirmed" viewBox="0 0 20 20" fill="currentColor" class="status-svg">
                <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.857-9.809a.75.75 0 00-1.214-.882l-3.483 4.79-1.88-1.88a.75.75 0 10-1.06 1.061l2.5 2.5a.75.75 0 001.137-.089l4-5.5z" clip-rule="evenodd"/>
              </svg>
              <svg v-else-if="scanResultStatus === 'failed'" viewBox="0 0 20 20" fill="currentColor" class="status-svg">
                <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.28 7.22a.75.75 0 00-1.06 1.06L8.94 10l-1.72 1.72a.75.75 0 101.06 1.06L10 11.06l1.72 1.72a.75.75 0 101.06-1.06L11.06 10l1.72-1.72a.75.75 0 00-1.06-1.06L10 8.94 8.28 7.22z" clip-rule="evenodd"/>
              </svg>
              <svg v-else viewBox="0 0 20 20" fill="currentColor" class="status-svg">
                <path fill-rule="evenodd" d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7-4a1 1 0 11-2 0 1 1 0 012 0zM9 9a.75.75 0 000 1.5h.253a.25.25 0 01.244.304l-.459 2.066A1.75 1.75 0 0010.747 15H11a.75.75 0 000-1.5h-.253a.25.25 0 01-.244-.304l.459-2.066A1.75 1.75 0 009.253 9H9z" clip-rule="evenodd" />
              </svg>
            </div>
            <span class="header-text">
              {{ scanResultStatus === 'success' ? (isConfirmed ? 'BERHASIL CHECK-IN' : 'MENUNGGU KONFIRMASI') : (scanResultStatus === 'failed' ? 'GAGAL SCAN' : 'SUDAH CHECK-IN') }}
            </span>
          </div>

          <div class="notification-body">
            <div class="attendee-row">
              <div class="attendee-info">
                <p class="event-name">{{ selectedEvent ? selectedEvent.title : 'Event Kolektix' }}</p>
                <h4 class="invoice-text">{{ scanData.invoice }}</h4>
                <p class="buyer-name">{{ scanData.name }}</p>
                <p class="ticket-type-label">{{ scanData.ticketType }}</p>
              </div>
            </div>

            <div class="scan-time-row">
              <div class="scan-time-info">
                <span class="time-label">WAKTU MASUK</span>
                <span class="time-val">{{ scanData.time || 'Baru Saja' }}</span>
              </div>
              <button class="continue-btn" @click="confirmCheckin">
                {{ scanResultStatus === 'success' && !isConfirmed ? 'Konfirmasi' : (scanResultStatus === 'failed' ? 'Coba Lagi' : 'Lanjut Scan') }}
              </button>
            </div>
          </div>
        </div>
      </transition>

      <!-- Slide-up Camera Failure Error Card -->
      <transition name="popup-slide">
        <div v-if="cameraErrorPopupVisible" class="notification-popup-card camera-error-card">
          <div class="notification-header failed">
            <div class="header-icon">
              <svg viewBox="0 0 20 20" fill="currentColor" class="status-svg">
                <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.28 7.22a.75.75 0 00-1.06 1.06L8.94 10l-1.72 1.72a.75.75 0 101.06 1.06L10 11.06l1.72 1.72a.75.75 0 101.06-1.06L11.06 10l1.72-1.72a.75.75 0 00-1.06-1.06L10 8.94 8.28 7.22z" clip-rule="evenodd"/>
              </svg>
            </div>
            <span class="header-text">KAMERA GAGAL DIAKTIFKAN</span>
          </div>
          <div class="notification-body">
            <p class="error-card-desc">Gagal mengakses kamera perangkat. Pastikan izin kamera telah diberikan atau coba gunakan input kode manual sebagai gantinya.</p>
            <div class="error-action-row">
              <button class="continue-btn" @click="retryCamera">Coba Lagi</button>
              <button class="continue-btn manual-switch-btn" @click="checkinMode = 'qr'; cameraErrorPopupVisible = false;">Input Manual</button>
            </div>
          </div>
        </div>
      </transition>
    </div>
  </div>
</template>

<style scoped>
.scanner-page {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100vh;
  height: 100dvh;
  z-index: 50;
}

.scanner-fullscreen-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: #000;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  color: white;
  font-family: var(--font-sans);
}

.scanner-body {
  flex-grow: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  position: relative;
  z-index: 3;
}

.scanner-camera-feed {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  z-index: 1;
}

.scanner-overlay-dark {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.45);
  z-index: 2;
  pointer-events: none;
}

.scanner-top-bar {
  position: relative;
  z-index: 5;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 20px;
  background: transparent;
}

.scanner-header-title {
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  text-align: center;
  pointer-events: none;
  max-width: 180px;
  overflow: hidden;
}

/* Marquee ticker for long title */
.marquee-wrap {
  overflow: hidden;
  white-space: nowrap;
}
.marquee-track {
  display: inline-block;
  font-size: 14px;
  font-weight: 700;
  color: white;
  text-shadow: 0 1px 3px rgba(0,0,0,0.5);
}
.marquee-animate {
  animation: marquee-scroll 10s linear infinite;
}
@keyframes marquee-scroll {
  0%   { transform: translateX(0); }
  100% { transform: translateX(-50%); }
}

.scanner-header-title h3 {
  font-size: 14px;
  font-weight: 700;
  margin: 0;
  text-shadow: 0 1px 3px rgba(0,0,0,0.5);
}

.scanner-header-title p {
  font-size: 10px;
  color: rgba(255, 255, 255, 0.7);
  margin: 2px 0 0 0;
  text-shadow: 0 1px 2px rgba(0,0,0,0.5);
}

.scanner-icon-btn {
  background-color: rgba(0, 0, 0, 0.4);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 50%;
  color: white;
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  backdrop-filter: blur(4px);
  transition: background-color 0.2s, border-color 0.2s;
}

.close-btn {
  width: 34px;
  height: 34px;
}
.close-btn .scanner-svg-icon {
  width: 16px;
  height: 16px;
}

.flash-btn {
  width: 30px;
  height: 30px;
}
.flash-btn .scanner-svg-icon {
  width: 14px;
  height: 14px;
}

.close-btn.manual-mode {
  background-color: rgba(25, 78, 158, 0.4);
  border-color: #194E9E;
}

.scanner-svg-icon {
  width: 20px;
  height: 20px;
}

.checkin-mode-toggle {
  position: relative;
  z-index: 3;
  display: flex;
  align-self: center;
  background-color: rgba(255, 255, 255, 0.15);
  padding: 3px;
  border-radius: 30px;
  gap: 4px;
  margin-top: 4px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(8px);
}

.toggle-btn {
  background: none;
  border: none;
  color: rgba(255, 255, 255, 0.8);
  padding: 6px 14px;
  font-size: 11px;
  font-weight: 600;
  border-radius: 20px;
  cursor: pointer;
}

.toggle-btn.active {
  background-color: #194E9E;
  color: white;
}

.scanner-viewfinder {
  position: absolute;
  top: -140px;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 3;
  width: 100%;
  height: calc(100% + 140px);
  pointer-events: none;
}

.scanner-laser-line {
  position: absolute;
  left: 0;
  right: 0;
  height: 4px;
  background-color: #194E9E;
  border-radius: 50%;
  box-shadow: 0 0 15px 4px rgba(25, 78, 158, 0.9);
  animation: scan-laser 3s infinite alternate cubic-bezier(0.45, 0.05, 0.55, 0.95);
  z-index: 2;
}

.scanner-laser-trail {
  position: absolute;
  left: 0;
  right: 0;
  height: 400px;
  background: linear-gradient(
    to bottom,
    rgba(25, 78, 158, 0) 0%,
    rgba(25, 78, 158, 0.6) 100%
  );
  animation: 
    scan-laser 3s infinite alternate cubic-bezier(0.45, 0.05, 0.55, 0.95),
    scan-flip 6s infinite step-end,
    scan-opacity 6s infinite ease-in-out;
  z-index: 1;
}

@keyframes scan-laser {
  0% { top: 0%; }
  100% { top: 100%; }
}

@keyframes scan-flip {
  0% { transform: translateY(-100%) scaleY(1); }
  50% { transform: translateY(0%) scaleY(-1); }
  100% { transform: translateY(-100%) scaleY(1); }
}

@keyframes scan-opacity {
  0% { opacity: 0; }
  10% { opacity: 1; }
  40% { opacity: 1; }
  50% { opacity: 0; }
  60% { opacity: 1; }
  90% { opacity: 1; }
  100% { opacity: 0; }
}
/* Manual mode bg and details */
.scanner-fullscreen-container.manual-bg {
  background-color: #f8fafc;
  color: #1e293b;
}

.scanner-fullscreen-container.manual-bg .scanner-header-title h3,
.scanner-fullscreen-container.manual-bg .marquee-track {
  color: #1e293b;
  text-shadow: none;
}

.scanner-fullscreen-container.manual-bg .scanner-header-title p {
  color: #64748b;
  text-shadow: none;
}

.scanner-fullscreen-container.manual-bg .scanner-top-bar {
  background: none;
}

.scanner-fullscreen-container.manual-bg .checkin-mode-toggle {
  background-color: rgba(0, 0, 0, 0.05);
  border-color: rgba(0, 0, 0, 0.08);
}

.scanner-fullscreen-container.manual-bg .toggle-btn {
  color: #475569;
}

.scanner-fullscreen-container.manual-bg .toggle-btn.active {
  color: white;
}

.manual-checkin-container {
  position: relative;
  z-index: 3;
  width: 100%;
  padding: 0 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  transform: translateY(-30px);
}

.manual-form-card {
  width: 100%;
  max-width: 320px;
  display: flex;
  flex-direction: column;
  gap: 16px;
  background-color: white;
  padding: 20px;
  border-radius: 12px;
  border: 1px solid #e2e8f0;
}

.manual-field-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
  text-align: left;
}

.manual-label {
  font-size: 13px;
  font-weight: 600;
  color: #1e293b;
}

.manual-input-field {
  width: 100%;
  border: 1px solid #cbd5e1;
  border-radius: 8px;
  padding: 12px 14px;
  font-size: 13px;
  outline: none;
  font-family: var(--font-sans);
}

.manual-desc-mockup {
  font-size: 10px;
  color: #64748b;
  line-height: 1.4;
}

.manual-submit-btn {
  background-color: #194E9E;
  color: white;
  border: none;
  border-radius: 8px;
  padding: 12px;
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
}

.manual-tips {
  font-size: 9px;
  color: #64748b;
  line-height: 1.4;
}

.scanner-bottom-stats {
  position: absolute;
  bottom: 55px;
  left: 0;
  right: 0;
  z-index: 3;
  padding: 16px 20px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.scanner-fullscreen-container.manual-bg .scanner-bottom-stats {
  background: none;
}

.stats-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.stats-label {
  font-size: 13px;
  font-weight: 700;
  color: #ffffff;
  letter-spacing: 0.5px;
}

.scanner-fullscreen-container.manual-bg .stats-label {
  color: #64748b;
}

.stats-val {
  font-size: 13px;
  font-weight: 700;
  color: #ffffff;
}

.scanner-fullscreen-container.manual-bg .stats-val {
  color: #1e293b;
}

.stats-progress-container {
  width: 100%;
  height: 6px;
  background-color: rgba(255, 255, 255, 0.2);
  border-radius: 10px;
  overflow: hidden;
}

.scanner-fullscreen-container.manual-bg .stats-progress-container {
  background-color: #e2e8f0;
}

.stats-progress-fill {
  height: 100%;
  background-color: #194E9E;
  border-radius: 10px;
}

/* Ticket Category Bar */
.ticket-category-bar {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 3;
  display: flex;
  align-items: center;
  background-color: #ffffff;
  padding: 8px 16px;
  box-shadow: 0 -4px 20px rgba(0, 0, 0, 0.1);
}

.category-buttons-row {
  display: flex;
  width: 100%;
  gap: 10px;
}

.sheet-category-btn {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: white;
  border: 1px solid #194E9E;
  color: #194E9E;
  padding: 8px 0;
  border-radius: 6px;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.sheet-category-btn.active {
  border-color: #194E9E;
  background-color: #194E9E;
  color: white;
}

/* Scanner (Manual) mode inversion */
.scanner-fullscreen-container.manual-bg .ticket-category-bar {
  background-color: #194E9E;
}

.scanner-fullscreen-container.manual-bg .sheet-category-btn {
  background-color: #194E9E;
  border-color: rgba(255, 255, 255, 0.3);
  color: white;
}

.scanner-fullscreen-container.manual-bg .sheet-category-btn.active {
  background-color: white;
  color: #194E9E;
  border-color: white;
}

/* Sliding Notification Card */
.notification-popup-card {
  position: absolute;
  bottom: 80px;
  left: 16px;
  right: 16px;
  background-color: white;
  border-radius: 12px;
  overflow: hidden;
  z-index: 100;
  color: #1e293b;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.25);
}

.notification-header {
  padding: 10px 16px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.notification-header.success {
  background-color: #d1fae5;
  color: #065f46;
}

.notification-header.pending {
  background-color: #dbeafe;
  color: #1e40af;
}

.notification-header.failed {
  background-color: #fee2e2;
  color: #991b1b;
}

.notification-header.already {
  background-color: #fef3c7;
  color: #92400e;
}

.header-icon {
  display: flex;
  align-items: center;
}

.status-svg {
  width: 18px;
  height: 18px;
}

.header-text {
  font-size: 11px;
  font-weight: 700;
}

.notification-body {
  padding: 20px 20px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.attendee-row {
  display: flex;
  align-items: center;
  width: 100%;
  padding-bottom: 14px;
  border-bottom: 1px solid #e2e8f0;
}

.event-name {
  font-size: 12px;
  font-weight: 500;
  color: #1e293b;
  margin: 0 0 6px 0;
}

.attendee-info {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.invoice-text {
  font-size: 16px;
  font-weight: 700;
  margin: 0;
  color: #0f172a;
}

.buyer-name {
  font-size: 14px;
  font-weight: 500;
  color: #0f172a;
  margin: 0;
}

.ticket-type-label {
  font-size: 13px;
  font-weight: 500;
  color: #0f172a;
  margin: 0;
}

.scan-time-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.scan-time-info {
  display: flex;
  flex-direction: column;
}

.time-label {
  font-size: 9px;
  color: #64748b;
}

.time-val {
  font-size: 12px;
  font-weight: 700;
}

.continue-btn {
  background-color: #194E9E;
  color: white;
  border: none;
  border-radius: 6px;
  padding: 6px 14px;
  font-size: 11px;
  font-weight: 600;
  cursor: pointer;
}

.camera-error-card .error-card-desc {
  font-size: 11px;
  color: #475569;
  line-height: 1.5;
}

.error-action-row {
  display: flex;
  gap: 8px;
  width: 100%;
}

.manual-switch-btn {
  background-color: #e2e8f0;
  color: #475569;
}

/* Transitions */
.popup-slide-enter-active,
.popup-slide-leave-active {
  transition: transform 0.3s cubic-bezier(0.16, 1, 0.3, 1), opacity 0.2s ease;
}

.popup-slide-enter-from,
.popup-slide-leave-to {
  transform: translateY(120%);
  opacity: 0;
}
</style>
