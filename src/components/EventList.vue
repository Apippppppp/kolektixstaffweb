<script setup>
import { ref, computed } from 'vue';

const props = defineProps({
  events: {
    type: Array,
    required: true
  }
});

const emit = defineEmits(['select-event']);

const selectEvent = (event) => {
  emit('select-event', event);
};

const showTutorial = ref(false);
const dragY = ref(0);
let startY = 0;
const isDragging = ref(false);

const handleDragStart = (e) => {
  const contentEl = e.target.closest('.tutorial-content');
  if (contentEl && contentEl.scrollTop > 0) return;
  
  startY = e.type.startsWith('touch') ? e.touches[0].clientY : e.clientY;
  dragY.value = 0;
  isDragging.value = true;
  
  if (!e.type.startsWith('touch')) {
    window.addEventListener('mousemove', handleDragMove);
    window.addEventListener('mouseup', handleDragEnd);
  }
};

function handleDragMove(e) {
  if (!isDragging.value) return;
  const currentY = e.type.startsWith('touch') ? e.touches[0].clientY : e.clientY;
  const diffY = currentY - startY;
  if (diffY > 0) {
    dragY.value = diffY;
  }
}

function handleDragEnd() {
  if (!isDragging.value) return;
  isDragging.value = false;
  window.removeEventListener('mousemove', handleDragMove);
  window.removeEventListener('mouseup', handleDragEnd);
  
  if (dragY.value > 100) {
    showTutorial.value = false;
  }
  dragY.value = 0;
}

const tutorialStyle = computed(() => {
  return { 
    transform: `translateY(${dragY.value}px)`, 
    transition: isDragging.value ? 'none' : 'transform 0.2s ease' 
  };
});
</script>

<template>
  <div class="event-list-container">
    <div class="header-banner">
      <h2>Pilih Event</h2>
      <p>Pilih event yang ingin Anda kelola untuk mulai melakukan check-in penonton.</p>
      <button class="baca-selengkapnya-btn" @click="showTutorial = true">Baca Selengkapnya</button>
    </div>

    <div class="events-grid">
      <div 
        v-for="event in events" 
        :key="event.id" 
        class="event-card"
        @click="selectEvent(event)"
      >
        <div class="card-image-wrapper">
          <img :src="event.image" :alt="event.title" class="event-img" />
          <span class="status-pill" :class="event.status.toLowerCase()">
            <span class="dot"></span>
            {{ event.status }}
          </span>
        </div>
        <div class="card-body">
          <!-- Event title with marquee for long names -->
          <div class="event-title-wrap">
            <span
              class="event-title"
              :class="{ 'marquee-animate': event.title.length > 28 }"
            >
              {{ event.title }}
              <template v-if="event.title.length > 28">
                &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                {{ event.title }}
              </template>
            </span>
          </div>
          <!-- Organizer row with creator image + verified badge -->
          <div class="organizer-row">
            <div class="organizer-avatar-wrap">
              <img :src="event.creatorLogo" :alt="event.organizer" class="organizer-avatar" />
              <div class="verified-badge">
                <svg viewBox="0 0 20 20" fill="currentColor" class="verified-icon">
                  <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.857-9.809a.75.75 0 00-1.214-.882l-3.483 4.79-1.88-1.88a.75.75 0 10-1.06 1.061l2.5 2.5a.75.75 0 001.137-.089l4-5.5z" clip-rule="evenodd" />
                </svg>
              </div>
            </div>
            <span class="organizer-name">{{ event.organizer }}</span>
          </div>

          <div class="info-row">
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" class="info-icon info-icon-blue">
              <path stroke-linecap="round" stroke-linejoin="round" d="M15 10.5a3 3 0 1 1-6 0 3 3 0 0 1 6 0Z" />
              <path stroke-linecap="round" stroke-linejoin="round" d="M19.5 10.5c0 7.142-7.5 11.25-7.5 11.25S4.5 17.642 4.5 10.5a7.5 7.5 0 1 1 15 0Z" />
            </svg>
            <span>{{ event.location }}</span>
          </div>

          <div class="info-row">
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" class="info-icon info-icon-blue">
              <path stroke-linecap="round" stroke-linejoin="round" d="M6.75 3v2.25M17.25 3v2.25M3 18.75V7.5a2.25 2.25 0 0 1 2.25-2.25h13.5A2.25 2.25 0 0 1 21 7.5v11.25m-18 0A2.25 2.25 0 0 0 5.25 21h13.5A2.25 2.25 0 0 0 21 18.75m-18 0v-7.5A2.25 2.25 0 0 1 5.25 9h13.5A2.25 2.25 0 0 1 21 11.25v7.5" />
            </svg>
            <span>{{ event.date }}</span>
          </div>

          <div class="progress-section">
            <div class="progress-labels">
              <span>{{ event.sold }}/{{ event.total }} Tiket Terjual</span>
              <span>{{ Math.round((event.sold / event.total) * 100) }}%</span>
            </div>
            <div class="progress-bar-container">
              <div class="progress-bar-fill" :style="{ width: `${(event.sold / event.total) * 100}%` }"></div>
            </div>
          </div>

          <button class="select-btn">
            Mulai Scan Checkin
          </button>
        </div>
      </div>
    </div>

    <!-- Draggable Tutorial Bottom Sheet -->
    <transition name="modal-fade">
      <div class="modal-overlay" v-if="showTutorial" @click.self="showTutorial = false">
        <div 
          class="modal-sheet tutorial-sheet"
          :style="tutorialStyle"
          @touchstart="handleDragStart"
          @touchmove="handleDragMove"
          @touchend="handleDragEnd"
          @mousedown="handleDragStart"
        >
          <div class="modal-handle"></div>
          <div class="detail-modal-header">
            <h3 class="modal-title">Panduan Scan Check-In</h3>
            <button class="close-x-btn" @click="showTutorial = false">
              <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" class="close-icon">
                <path stroke-linecap="round" stroke-linejoin="round" d="M6 18 18 6M6 6l12 12" />
              </svg>
            </button>
          </div>

          <div class="tutorial-content">
            <div class="tutorial-step">
              <div class="step-num">1</div>
              <div class="step-text">
                <strong>Pilih Tipe Scan</strong>
                <p>Ketuk tab Reguler atau Invitation di scanner sesuai dengan kategori tiket penonton.</p>
              </div>
            </div>
            <div class="tutorial-step">
              <div class="step-num">2</div>
              <div class="step-text">
                <strong>Arahkan Kamera ke QR Code</strong>
                <p>Arahkan kamera HP ke e-ticket QR Code milik penonton hingga terdeteksi.</p>
              </div>
            </div>
            <div class="tutorial-step">
              <div class="step-num">3</div>
              <div class="step-text">
                <strong>Input Manual Jika Gagal</strong>
                <p>Apabila QR Code rusak, ketuk tab Manual dan masukkan kode invoice / tiket penonton.</p>
              </div>
            </div>
            <div class="tutorial-step">
              <div class="step-num">4</div>
              <div class="step-text">
                <strong>Verifikasi Status Popup</strong>
                <p>Pastikan muncul popup notifikasi "BERHASIL SCAN" berwarna hijau sebelum membiarkan penonton masuk.</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<style scoped>
.event-list-container {
  padding: 16px;
  background-color: #f8fafc;
  min-height: calc(100vh - 56px - 60px);
  font-family: var(--font-sans);
}

.header-banner {
  background: linear-gradient(135deg, #194E9E 0%, #0d3e91 100%);
  padding: 20px;
  border-radius: 12px;
  color: white;
  margin-bottom: 20px;
  box-shadow: 0 4px 14px rgba(25, 78, 158, 0.15);
  display: flex;
  flex-direction: column;
}

.header-banner h2 {
  font-size: 18px;
  font-weight: 700;
  margin-bottom: 6px;
}

.header-banner p {
  font-size: 12px;
  opacity: 0.9;
  line-height: 1.5;
}

/* Baca Selengkapnya Button */
.baca-selengkapnya-btn {
  margin-top: 12px;
  background-color: rgba(255, 255, 255, 0.15);
  border: 1px solid rgba(255, 255, 255, 0.3);
  color: white;
  border-radius: 6px;
  padding: 6px 14px;
  font-size: 11px;
  font-weight: 600;
  cursor: pointer;
  align-self: flex-start;
  transition: background-color 0.2s;
  font-family: var(--font-sans);
}

.baca-selengkapnya-btn:hover {
  background-color: rgba(255, 255, 255, 0.25);
}

.events-grid {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.event-card {
  background-color: white;
  border-radius: 12px;
  border: 1px solid #e2e8f0;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.02);
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
}

.event-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
}

.card-image-wrapper {
  position: relative;
  height: 140px;
}

.event-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.status-pill {
  position: absolute;
  top: 10px;
  left: 10px;
  background-color: white;
  padding: 4px 10px;
  border-radius: 20px;
  font-size: 10px;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 4px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.1);
}

.status-pill.live {
  color: #16a34a;
}
.status-pill.live .dot {
  background-color: #16a34a;
}

.status-pill.upcoming {
  color: #ea580c;
}
.status-pill.upcoming .dot {
  background-color: #ea580c;
}

.dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
}

.card-body {
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

/* Event title marquee */
.event-title-wrap {
  overflow: hidden;
  white-space: nowrap;
}

.event-title {
  display: inline-block;
  font-size: 15px;
  font-weight: 700;
  color: #0f172a;
  white-space: nowrap;
}

.marquee-animate {
  animation: ev-marquee 12s linear infinite;
}

@keyframes ev-marquee {
  0%   { transform: translateX(0); }
  100% { transform: translateX(-50%); }
}

.event-organizer {
  font-size: 11px;
  color: #64748b;
  margin-bottom: 4px;
}

.info-row {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 12px;
  color: #475569;
}

.info-icon {
  width: 16px;
  height: 16px;
  color: #64748b;
  flex-shrink: 0;
}

.info-icon-blue {
  color: #194E9E;
}

/* Organizer row */
.organizer-row {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 2px;
}

.organizer-avatar-wrap {
  position: relative;
  flex-shrink: 0;
}

.organizer-avatar {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  object-fit: cover;
  border: 1.5px solid #e2e8f0;
}

.verified-badge {
  position: absolute;
  bottom: -2px;
  right: -3px;
  width: 13px;
  height: 13px;
  background-color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.verified-icon {
  width: 13px;
  height: 13px;
  color: #194E9E;
}

.organizer-name {
  font-size: 11px;
  font-weight: 600;
  color: #475569;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.progress-section {
  margin-top: 8px;
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.progress-labels {
  display: flex;
  justify-content: space-between;
  font-size: 11px;
  font-weight: 600;
  color: #475569;
}

.progress-bar-container {
  height: 6px;
  background-color: #f1f5f9;
  border-radius: 10px;
  overflow: hidden;
}

.progress-bar-fill {
  height: 100%;
  background-color: #194E9E;
  border-radius: 10px;
}

.select-btn {
  margin-top: 12px;
  background-color: #194E9E;
  color: white;
  border: none;
  border-radius: 8px;
  padding: 10px;
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
  text-align: center;
  transition: background-color 0.2s;
}

.select-btn:hover {
  background-color: #1453b6;
}

/* Modals & Drag Sheet */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0,0,0,0.4);
  z-index: 100;
  display: flex;
  align-items: flex-end;
  justify-content: center;
}

.modal-sheet {
  background-color: white;
  border-top-left-radius: 16px;
  border-top-right-radius: 16px;
  padding: 20px;
  width: 100%;
  max-width: 410px;
  display: flex;
  flex-direction: column;
  gap: 16px;
  touch-action: none;
}

.modal-handle {
  width: 40px;
  height: 4px;
  background-color: #cbd5e1;
  border-radius: 2px;
  align-self: center;
  margin-bottom: 8px;
}

.detail-modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.modal-title {
  font-size: 15px;
  font-weight: 700;
  color: #0f172a;
}

.close-x-btn {
  background: none;
  border: none;
  color: #cbd5e1;
  cursor: pointer;
  padding: 4px;
  display: flex;
  align-items: center;
}

.close-icon {
  width: 20px;
  height: 20px;
  color: #94a3b8;
}

.tutorial-content {
  display: flex;
  flex-direction: column;
  gap: 16px;
  margin-top: 8px;
}

.tutorial-step {
  display: flex;
  gap: 12px;
}

.step-num {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background-color: #f0f6ff;
  color: #194E9E;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  font-weight: 700;
  flex-shrink: 0;
}

.step-text strong {
  font-size: 13px;
  color: #0f172a;
  display: block;
}

.step-text p {
  font-size: 11px;
  color: #64748b;
  margin-top: 2px;
  line-height: 1.4;
}

/* Modal Fade/Slide Animation */
.modal-fade-enter-active,
.modal-fade-leave-active {
  transition: opacity 0.2s ease;
}

.modal-fade-enter-active .modal-sheet,
.modal-fade-leave-active .modal-sheet {
  transition: transform 0.25s cubic-bezier(0.16, 1, 0.3, 1);
}

.modal-fade-enter-from,
.modal-fade-leave-to {
  opacity: 0;
}

.modal-fade-enter-from .modal-sheet,
.modal-fade-leave-to .modal-sheet {
  transform: translateY(100%);
}
</style>
