<script setup>
import { ref, computed, onMounted } from 'vue';

const props = defineProps({
  events: {
    type: Array,
    required: true
  }
});

const ticketType = ref('eticket'); // 'eticket' | 'invitation'
const searchQuery = ref('');
const selectedEvent = ref(null);
const showEventPicker = ref(false);
const showDetails = ref(false);
const selectedTicket = ref(null);

const allTickets = ref([
  { id: 1, eventId: 1, name: 'Budi Santoso',    invoice: 'INV-00123', seat: 'A-12', type: 'VIP',      phone: '08123456789', email: 'budi.santoso@email.com',  status: 'Checked In', ticketKind: 'eticket', time: '19:42' },
  { id: 2, eventId: 1, name: 'Siti Rahmawati',  invoice: 'INV-00124', seat: 'F-08', type: 'Festival', phone: '08987654321', email: 'siti.r@email.com',          status: 'Pending',    ticketKind: 'eticket', time: '' },
  { id: 3, eventId: 1, name: 'Ahmad Fauzi',     invoice: 'INV-00125', seat: 'B-03', type: 'VIP',      phone: '08211234567', email: 'ahmad.f@email.com',         status: 'Checked In',    ticketKind: 'invitation', time: '20:10' },
  { id: 4, eventId: 2, name: 'Rina Kusuma',     invoice: 'INV-00201', seat: 'C-10', type: 'Regular',  phone: '08567891234', email: 'rina.k@email.com',          status: 'Checked In', ticketKind: 'eticket', time: '18:15' },
  { id: 5, eventId: 2, name: 'Doni Pratama',    invoice: 'INV-00202', seat: 'D-05', type: 'Regular',  phone: '08312345678', email: 'doni.p@email.com',          status: 'Pending',    ticketKind: 'invitation', time: '' },
]);

onMounted(() => {
  if (props.events && props.events.length > 0) {
    selectedEvent.value = props.events[0];
  }
});

const filteredTickets = computed(() => {
  let list = allTickets.value;
  if (selectedEvent.value) {
    list = list.filter(t => t.eventId === selectedEvent.value.id);
  }
  list = list.filter(t => t.ticketKind === ticketType.value);
  if (searchQuery.value.trim()) {
    const q = searchQuery.value.toLowerCase();
    list = list.filter(t =>
      t.name.toLowerCase().includes(q) ||
      t.invoice.toLowerCase().includes(q) ||
      t.phone.includes(q)
    );
  }
  return list;
});

const getEventStats = (eventId) => {
  const evTickets = allTickets.value.filter(t => t.eventId === eventId);
  const paid = evTickets.length;
  const checkedIn = evTickets.filter(t => t.status === 'Checked In').length;
  return { paid, checkedIn };
};

const selectEvent = (ev) => {
  selectedEvent.value = ev;
  showEventPicker.value = false;
};

const doCheckIn = (ticket) => {
  ticket.status = 'Checked In';
  ticket.time = new Date().toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' }) + ' WIB';
};

// Dragging logic for Details and Event Picker Modals
const dragY = ref(0);
const activeDragModal = ref(null); // 'picker' or 'detail'
let startY = 0;
const isDragging = ref(false);

const handleDragStart = (e, modal) => {
  const listEl = e.target.closest('.modal-event-list') || e.target.closest('.detail-info-grid');
  if (listEl && listEl.scrollTop > 0) return;

  startY = e.type.startsWith('touch') ? e.touches[0].clientY : e.clientY;
  dragY.value = 0;
  activeDragModal.value = modal;
  isDragging.value = true;

  if (!e.type.startsWith('touch')) {
    window.addEventListener('mousemove', handleDragMove);
    window.addEventListener('mouseup', handleDragEnd);
  }
};

function handleDragMove(e) {
  if (!isDragging.value || !activeDragModal.value) return;
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
    if (activeDragModal.value === 'picker') {
      showEventPicker.value = false;
    } else if (activeDragModal.value === 'detail') {
      closeDetails();
    }
  }
  dragY.value = 0;
  activeDragModal.value = null;
}

const pickerStyle = computed(() => {
  if (activeDragModal.value === 'picker') {
    return { 
      transform: `translateY(${dragY.value}px)`, 
      transition: isDragging.value ? 'none' : 'transform 0.2s ease' 
    };
  }
  return {};
});

const detailStyle = computed(() => {
  if (activeDragModal.value === 'detail') {
    return { 
      transform: `translateY(${dragY.value}px)`, 
      transition: isDragging.value ? 'none' : 'transform 0.2s ease' 
    };
  }
  return {};
});

const closeDetails = () => {
  showDetails.value = false;
};

const openDetails = (ticket) => {
  selectedTicket.value = ticket;
  showDetails.value = true;
};
</script>

<template>
  <div class="report-container">
    <!-- Top Three-Column Stats Box (Matching Image 1) -->
    <div class="stats-overview-card" v-if="selectedEvent">
      <div class="overview-col">
        <span class="overview-label">Tiket Terjual</span>
        <span class="overview-val">{{ getEventStats(selectedEvent.id).paid }}</span>
      </div>
      <div class="overview-divider"></div>
      <div class="overview-col">
        <span class="overview-label">Sudah Check-In</span>
        <span class="overview-val">{{ getEventStats(selectedEvent.id).checkedIn }}</span>
      </div>
      <div class="overview-divider"></div>
      <div class="overview-col">
        <span class="overview-label">Belum Check-In</span>
        <span class="overview-val">{{ getEventStats(selectedEvent.id).paid - getEventStats(selectedEvent.id).checkedIn }}</span>
      </div>
    </div>

    <!-- Top Event Selector Button (Blue style matching Image 1) -->
    <button class="select-event-btn" @click="showEventPicker = true">
      <div class="btn-left">
        <!-- Calendar SVG Icon -->
        <svg class="calendar-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <rect x="3" y="4" width="18" height="16" rx="2" stroke-linecap="round" stroke-linejoin="round" />
          <line x1="3" y1="9" x2="21" y2="9" stroke-linecap="round" stroke-linejoin="round" />
        </svg>
        <span class="event-btn-title">{{ selectedEvent ? selectedEvent.title : 'Pilih Event' }}</span>
      </div>
      <svg class="chevron-right-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
        <polyline points="9 18 15 12 9 6"></polyline>
      </svg>
    </button>

    <!-- Search Field -->
    <div class="search-wrapper">
      <svg class="search-icon" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" d="m21 21-5.197-5.197m0 0A7.5 7.5 0 1 0 5.196 5.196a7.5 7.5 0 0 0 10.607 10.607Z" />
      </svg>
      <input type="text" v-model="searchQuery" placeholder="Cari (Invoice, Nama, HP...)" class="search-input" />
    </div>

    <!-- Segmented Control -->
    <div class="segmented-control">
      <button class="segment-btn" :class="{ active: ticketType === 'eticket' }" @click="ticketType = 'eticket'">E-Ticket</button>
      <button class="segment-btn" :class="{ active: ticketType === 'invitation' }" @click="ticketType = 'invitation'">Invitation</button>
    </div>

    <!-- Stats summary line -->
    <div class="summary-info-row" v-if="selectedEvent">
      <span class="total-text">Menampilkan {{ filteredTickets.length }} tiket</span>
      <div class="checked-in-count">
        <span class="green-dot"></span>
        <span>{{ getEventStats(selectedEvent.id).checkedIn }} Checked In</span>
      </div>
    </div>

    <!-- Tickets List -->
    <div class="tickets-list">
      <div v-if="filteredTickets.length === 0" class="empty-tickets">
        <lottie-player
          src="/media/sad%20emotion.json"
          background="transparent"
          speed="1"
          loop
          autoplay
          class="empty-lottie"
        ></lottie-player>
        <p class="empty-label">Tidak ada tiket ditemukan</p>
      </div>

      <!-- Ticket Card Layout (Matching Image 2 and 3 exactly) -->
      <div
        v-for="ticket in filteredTickets"
        :key="ticket.id"
        class="ticket-card-mock"
      >
        <div class="ticket-top-row">
          <div>
            <h4 class="ticket-name-mock">{{ ticket.name }}</h4>
            <p class="ticket-invoice-mock">{{ ticket.invoice }}</p>
          </div>
          
          <!-- Dynamic Status Badge matching Image 2 & 3 -->
          <div v-if="ticket.status === 'Checked In'" class="ticket-status-badge-mock badge-checked-in-mock">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" class="badge-icon-mock">
              <polyline points="20 6 9 17 4 12"></polyline>
            </svg>
            <span>Sudah Check-In</span>
          </div>
          <div v-else class="ticket-status-badge-mock badge-pending-mock">
            <!-- Circular exclamation alert icon for pending -->
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" class="badge-icon-mock">
              <circle cx="12" cy="12" r="10"></circle>
              <line x1="12" y1="8" x2="12" y2="12"></line>
              <line x1="12" y1="16" x2="12.01" y2="16"></line>
            </svg>
            <span>Belum Check-In</span>
          </div>
        </div>

        <div class="ticket-details-grid-mock">
          <div class="detail-item-mock">
            <span class="detail-label-mock">Kursi</span>
            <span class="detail-val-mock">{{ ticket.seat }} <span class="detail-type-badge-mock">({{ ticket.type }})</span></span>
          </div>
          <div class="detail-item-mock">
            <span class="detail-label-mock">Telepon</span>
            <span class="detail-val-mock">{{ ticket.phone }}</span>
          </div>
          <div class="detail-item-mock full-width-mock">
            <span class="detail-label-mock">Email</span>
            <span class="detail-val-mock-email">{{ ticket.email }}</span>
          </div>
        </div>

        <div class="ticket-divider-mock"></div>

        <!-- Dynamic Action Footer matching Image 2 & 3 -->
        <div class="ticket-footer-mock" :class="{ 'pending-footer': ticket.status !== 'Checked In' }">
          <template v-if="ticket.status === 'Checked In'">
            <button class="lihat-detail-btn" @click="openDetails(ticket)">
              <span>Lihat Detail</span>
            </button>
          </template>
          <template v-else>
            <!-- Check In Action Button (Blue block on left) -->
            <button class="action-checkin-btn" @click="doCheckIn(ticket)">
              <!-- Grid Scanner Icon -->
              <svg class="grid-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
                <rect x="3" y="3" width="7" height="7" rx="1"></rect>
                <rect x="14" y="3" width="7" height="7" rx="1"></rect>
                <rect x="14" y="14" width="7" height="7" rx="1"></rect>
                <rect x="3" y="14" width="7" height="7" rx="1"></rect>
              </svg>
              <span>Check In</span>
            </button>
            <!-- More options detail button (Grey block on right) -->
            <button class="action-dots-btn" @click="openDetails(ticket)">
              <span>•••</span>
            </button>
          </template>
        </div>
      </div>
    </div>

    <!-- Draggable Event Picker Modal -->
    <transition name="modal-fade">
      <div class="modal-overlay" v-if="showEventPicker" @click.self="showEventPicker = false">
        <div 
          class="modal-sheet"
          :style="pickerStyle"
          @touchstart="handleDragStart($event, 'picker')"
          @touchmove="handleDragMove"
          @touchend="handleDragEnd"
          @mousedown="handleDragStart($event, 'picker')"
        >
          <div class="modal-handle"></div>
          <div class="detail-modal-header">
            <h3 class="modal-title">Pilih Event</h3>
            <button class="close-x-btn" @click="showEventPicker = false">
              <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" class="close-icon">
                <path stroke-linecap="round" stroke-linejoin="round" d="M6 18 18 6M6 6l12 12" />
              </svg>
            </button>
          </div>

          <div class="modal-list modal-event-list">
            <div
              v-for="ev in events"
              :key="ev.id"
              class="modal-item"
              :class="{ 'selected': selectedEvent && selectedEvent.id === ev.id }"
              @click="selectEvent(ev)"
            >
              <span>{{ ev.title }}</span>
            </div>
          </div>
        </div>
      </div>
    </transition>

    <!-- Draggable Ticket Detail Modal -->
    <transition name="modal-fade">
      <div class="modal-overlay" v-if="showDetails && selectedTicket" @click.self="showDetails = false">
        <div 
          class="modal-sheet detail-sheet"
          :style="detailStyle"
          @touchstart="handleDragStart($event, 'detail')"
          @touchmove="handleDragMove"
          @touchend="handleDragEnd"
          @mousedown="handleDragStart($event, 'detail')"
        >
          <div class="modal-handle"></div>
          <div class="detail-modal-header">
            <h3 class="modal-title">Detail Tiket</h3>
            <button class="close-x-btn" @click="showDetails = false">
              <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" class="close-icon">
                <path stroke-linecap="round" stroke-linejoin="round" d="M6 18 18 6M6 6l12 12" />
              </svg>
            </button>
          </div>

          <div class="detail-grid detail-info-grid">
            <div class="detail-row">
              <span class="lbl">Nama:</span>
              <span class="val">{{ selectedTicket.name }}</span>
            </div>
            <div class="detail-row">
              <span class="lbl">Invoice:</span>
              <span class="val">{{ selectedTicket.invoice }}</span>
            </div>
            <div class="detail-row">
              <span class="lbl">Kursi/Tipe:</span>
              <span class="val">{{ selectedTicket.seat }} ({{ selectedTicket.type }})</span>
            </div>
            <div class="detail-row">
              <span class="lbl">Email:</span>
              <span class="val">{{ selectedTicket.email }}</span>
            </div>
            <div class="detail-row">
              <span class="lbl">Status:</span>
              <span class="val" :style="{ color: selectedTicket.status === 'Checked In' ? '#16a34a' : '#ea580c' }">{{ selectedTicket.status }}</span>
            </div>
          </div>
          
          <button v-if="selectedTicket.status === 'Pending'" class="checkin-now-btn" @click="doCheckIn(selectedTicket); showDetails = false;">
            Check-In Sekarang
          </button>
        </div>
      </div>
    </transition>
  </div>
</template>

<style scoped>
.report-container {
  padding: 16px;
  background-color: #f8fafc;
  min-height: calc(100vh - 56px - 60px);
  font-family: var(--font-sans);
}

/* Stats Overview top card (Matching Image 1) */
.stats-overview-card {
  background-color: white;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  padding: 16px;
  display: flex;
  justify-content: space-around;
  align-items: center;
  margin-bottom: 14px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.01);
}

.overview-col {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  flex: 1;
}

.overview-label {
  font-size: 10px;
  font-weight: 500;
  color: #94a3b8;
  text-align: center;
  white-space: nowrap;
}

.overview-val {
  font-size: 20px;
  font-weight: 800;
  color: #0f172a;
}

.overview-divider {
  width: 1px;
  height: 28px;
  background-color: #e2e8f0;
}

/* Event Selector Top Button (Matching Image 1) */
.select-event-btn {
  width: 100%;
  background-color: #194E9E;
  border: none;
  border-radius: 10px;
  padding: 14px 16px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 15px;
  font-weight: 700;
  color: white;
  cursor: pointer;
  margin-bottom: 14px;
  overflow: hidden;
}

.event-btn-title {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 220px;
}

.btn-left {
  display: flex;
  align-items: center;
  gap: 10px;
}

.calendar-icon {
  width: 20px;
  height: 20px;
}

.chevron-right-icon {
  width: 16px;
  height: 16px;
}

.search-wrapper {
  position: relative;
  margin-bottom: 16px;
}

.search-icon {
  position: absolute;
  left: 14px;
  top: 14px;
  width: 18px;
  height: 18px;
  color: #94a3b8;
}

.search-input {
  width: 100%;
  padding: 12px 14px 12px 42px;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  font-size: 14px;
  outline: none;
  background-color: white;
}

.segmented-control {
  display: flex;
  background-color: #f1f5f9;
  padding: 4px;
  border-radius: 8px;
  margin-bottom: 16px;
}

.segment-btn {
  flex: 1;
  background: none;
  border: none;
  padding: 10px;
  font-size: 14px;
  font-weight: 600;
  color: #64748b;
  cursor: pointer;
  border-radius: 6px;
}

.segment-btn.active {
  background-color: white;
  color: #194E9E;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.05);
}

.summary-info-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 14px;
  font-weight: 500;
  color: #475569;
  margin-bottom: 16px;
  padding: 0 4px;
}

.checked-in-count {
  display: flex;
  align-items: center;
  gap: 6px;
}

.green-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background-color: #10b981;
}

.tickets-list {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

/* Empty state */
.empty-tickets {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40px 0;
  gap: 6px;
}
.empty-lottie {
  width: 120px;
  height: 120px;
}
.empty-label {
  font-size: 13px;
  color: #94a3b8;
  text-align: center;
}

/* Ticket Card Layout matching Image 2 & 3 */
.ticket-card-mock {
  background-color: white;
  border: 1px solid #e8eef4;
  border-radius: 10px;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 12px;
  box-shadow: 0 1px 4px rgba(0,0,0,0.04);
}

.ticket-top-row {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
}

.ticket-name-mock {
  font-size: 16px;
  font-weight: 700;
  color: #0f172a;
}

.ticket-invoice-mock {
  font-size: 13px;
  color: #94a3b8;
  margin-top: 2px;
}

.ticket-status-badge-mock {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 10px;
  font-weight: 700;
  padding: 3px 8px;
  border-radius: 20px;
  white-space: nowrap;
}

.badge-checked-in-mock {
  background-color: #065f46;
  color: #d1fae5;
}

.badge-pending-mock {
  background-color: #991b1b;
  color: #fee2e2;
}

.badge-icon-mock {
  width: 10px;
  height: 10px;
}

.ticket-details-grid-mock {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.detail-item-mock {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.detail-item-mock.full-width-mock {
  grid-column: span 2;
}

.detail-label-mock {
  font-size: 10px;
  font-weight: 500;
  color: #94a3b8;
}

.detail-val-mock {
  font-size: 13px;
  font-weight: 600;
  color: #0f172a;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.detail-val-mock-email {
  font-size: 13px;
  font-weight: 500;
  color: #0f172a;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.detail-type-badge-mock {
  color: #94a3b8;
  font-weight: 500;
}

.ticket-divider-mock {
  border-top: 1px dotted #e2e8f0;
}

.ticket-footer-mock {
  display: flex;
  width: 100%;
}

.ticket-footer-mock.pending-footer {
  display: flex;
  justify-content: space-between;
  gap: 10px;
}

.lihat-detail-btn {
  width: 100%;
  background-color: #194E9E;
  border: none;
  border-radius: 8px;
  padding: 10px 16px;
  text-align: center;
  font-size: 13px;
  font-weight: 700;
  color: white;
  cursor: pointer;
  transition: background-color 0.15s;
}

.lihat-detail-btn:hover {
  background-color: #1453b6;
}

.action-checkin-btn {
  flex: 1;
  background-color: #194E9E;
  color: white;
  border: none;
  border-radius: 10px;
  padding: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  font-size: 12px;
  font-weight: 700;
  cursor: pointer;
}

.grid-icon {
  width: 14px;
  height: 14px;
}

.action-dots-btn {
  background-color: #f1f5f9;
  color: #64748b;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  padding: 10px 16px;
  font-size: 12px;
  font-weight: 700;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.eye-icon {
  width: 14px;
  height: 14px;
}

/* Draggable Modals */
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
  color: #94a3b8;
  cursor: pointer;
  padding: 4px;
  display: flex;
  align-items: center;
}

.close-icon {
  width: 20px;
  height: 20px;
}

.modal-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.modal-item {
  padding: 12px;
  border-radius: 8px;
  border: 1px solid #e2e8f0;
  font-size: 13px;
  cursor: pointer;
}

.modal-item.selected {
  border-color: #194E9E;
  background-color: #f0f6ff;
  color: #194E9E;
  font-weight: 600;
}

.detail-grid {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.detail-row {
  display: flex;
  justify-content: space-between;
  font-size: 12px;
}

.detail-row .lbl {
  color: #64748b;
}

.detail-row .val {
  font-weight: 600;
  color: #1e293b;
}

.checkin-now-btn {
  background-color: #16a34a;
  color: white;
  border: none;
  border-radius: 8px;
  padding: 10px;
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
}

/* Modal Animations */
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
