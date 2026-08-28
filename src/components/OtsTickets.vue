<script setup>
import { ref, computed, onMounted } from 'vue';

const props = defineProps({
  events: {
    type: Array,
    required: true
  }
});

// State
const selectedEvent = ref(null);
const showEventPicker = ref(false);
const showCreateForm = ref(false);
const showDetailModal = ref(false);
const selectedTicket = ref(null);
const searchQuery = ref('');
const ticketTypeFilter = ref('all'); // 'all' | 'regular' | 'vip'

// New OTS ticket form
const form = ref({
  name: '',
  phone: '',
  email: '',
  seat: '',
  type: 'Regular',
  qty: 1,
  paymentMethod: 'Tunai',
});

// OTS ticket records (simulated offline sales)
const otsTickets = ref([
  { id: 1, eventId: 1, invoice: 'OTS-000101', name: 'Budi Santoso', seat: 'A-01', type: 'VIP', phone: '08123456789', email: 'budi@email.com', qty: 1, totalPrice: 'Rp248.000', paymentMethod: 'Tunai', status: 'Lunas', time: '19:30' },
  { id: 2, eventId: 1, invoice: 'OTS-000102', name: 'Dewi Lestari', seat: 'F-10', type: 'Regular', phone: '08998877665', email: 'dewi@email.com', qty: 2, totalPrice: 'Rp248.000', paymentMethod: 'QRIS', status: 'Lunas', time: '20:05' },
  { id: 3, eventId: 1, invoice: 'OTS-000103', name: 'Andi Pratama', seat: 'C-05', type: 'Regular', phone: '08111223344', email: 'andi@email.com', qty: 1, totalPrice: 'Rp124.000', paymentMethod: 'Transfer', status: 'Menunggu', time: '' },
  { id: 4, eventId: 2, invoice: 'OTS-000201', name: 'Siti Rahma', seat: 'B-03', type: 'VIP', phone: '08765432100', email: 'siti@email.com', qty: 1, totalPrice: 'Rp16.000', paymentMethod: 'Tunai', status: 'Lunas', time: '18:20' },
]);

onMounted(() => {
  if (props.events && props.events.length > 0) {
    selectedEvent.value = props.events[0];
  }
});

const filteredTickets = computed(() => {
  let list = otsTickets.value;
  if (selectedEvent.value) list = list.filter(t => t.eventId === selectedEvent.value.id);
  // Only show confirmed/lunas OTS tickets
  list = list.filter(t => t.status === 'Lunas');
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

const getStats = (eventId) => {
  const list = otsTickets.value.filter(t => t.eventId === eventId);
  const total = list.length;
  const lunas = list.filter(t => t.status === 'Lunas').length;
  return { total, lunas };
};

const selectEvent = (ev) => {
  selectedEvent.value = ev;
  showEventPicker.value = false;
};

const openDetail = (ticket) => {
  selectedTicket.value = ticket;
  showDetailModal.value = true;
};

const confirmPayment = (ticket) => {
  ticket.status = 'Lunas';
  ticket.time = new Date().toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' });
  showDetailModal.value = false;
};

const submitForm = () => {
  if (!form.value.name || !form.value.phone) {
    alert('Nama dan nomor telepon wajib diisi.');
    return;
  }
  if (!selectedEvent.value) {
    alert('Pilih event terlebih dahulu.');
    return;
  }
  const newId = otsTickets.value.length + 1;
  const invoice = 'OTS-' + String(Math.floor(100000 + Math.random() * 900000)).slice(0, 6);
  otsTickets.value.push({
    id: newId,
    eventId: selectedEvent.value.id,
    invoice,
    name: form.value.name,
    seat: form.value.seat || '-',
    type: form.value.type,
    phone: form.value.phone,
    email: form.value.email,
    qty: form.value.qty,
    totalPrice: '—',
    paymentMethod: form.value.paymentMethod,
    status: 'Menunggu',
    time: '',
  });
  // Reset form
  form.value = { name: '', phone: '', email: '', seat: '', type: 'Regular', qty: 1, paymentMethod: 'Tunai' };
  showCreateForm.value = false;
};

// Drag logic
const dragY = ref(0);
const activeDragModal = ref(null);
let startY = 0;
const isDragging = ref(false);

// QR modal refs (declared before handleDragEnd which references them)
const showQRModal = ref(false);
const selectedQRTicket = ref(null);

const openQR = (ticket) => {
  selectedQRTicket.value = ticket;
  showQRModal.value = true;
};

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
  if (diffY > 0) dragY.value = diffY;
}

function handleDragEnd() {
  if (!isDragging.value) return;
  isDragging.value = false;
  window.removeEventListener('mousemove', handleDragMove);
  window.removeEventListener('mouseup', handleDragEnd);
  if (dragY.value > 100) {
    if (activeDragModal.value === 'picker') showEventPicker.value = false;
    else if (activeDragModal.value === 'detail') showDetailModal.value = false;
    else if (activeDragModal.value === 'create') showCreateForm.value = false;
    else if (activeDragModal.value === 'qr') showQRModal.value = false;
  }
  dragY.value = 0;
  activeDragModal.value = null;
}

const sheetStyle = (modal) => computed(() => {
  if (activeDragModal.value === modal) {
    return { transform: `translateY(${dragY.value}px)`, transition: isDragging.value ? 'none' : 'transform 0.2s ease' };
  }
  return {};
});

const pickerStyle = sheetStyle('picker');
const detailStyle = sheetStyle('detail');
const createStyle = sheetStyle('create');
const qrStyle = sheetStyle('qr');
</script>

<template>
  <div class="ots-container">


    <!-- Event Selector -->
    <button class="select-event-btn" @click="showEventPicker = true">
      <div class="btn-left">
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

    <!-- Search -->
    <div class="search-wrapper">
      <svg class="search-icon" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" d="m21 21-5.197-5.197m0 0A7.5 7.5 0 1 0 5.196 5.196a7.5 7.5 0 0 0 10.607 10.607Z" />
      </svg>
      <input type="text" v-model="searchQuery" placeholder="Cari nama, invoice, atau HP..." class="search-input" />
    </div>

    <!-- Summary row + Add button -->
    <div class="summary-info-row" v-if="selectedEvent">
      <span>{{ filteredTickets.length }} tiket OTS</span>
      <button class="add-ticket-btn" @click="showCreateForm = true">+ Tambah OTS</button>
    </div>

    <!-- Ticket list -->
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
        <p class="empty-label">Belum ada tiket OTS untuk event ini</p>
      </div>

      <div v-for="ticket in filteredTickets" :key="ticket.id" class="ticket-card">
        <!-- Top row: name + status badge -->
        <div class="ticket-top-row">
          <div class="ticket-name-group">
            <h4 class="ticket-name">{{ ticket.name }}</h4>
            <p class="ticket-invoice">{{ ticket.invoice }}</p>
          </div>
          <div class="status-badge" :class="ticket.status === 'Lunas' ? 'badge-lunas' : 'badge-menunggu'">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" class="badge-icon">
              <polyline v-if="ticket.status === 'Lunas'" points="20 6 9 17 4 12"></polyline>
              <template v-else>
                <circle cx="12" cy="12" r="10"></circle>
                <line x1="12" y1="8" x2="12" y2="12"></line>
                <line x1="12" y1="16" x2="12.01" y2="16"></line>
              </template>
            </svg>
            <span>{{ ticket.status }}</span>
          </div>
        </div>

        <!-- Detail grid: seat, phone, payment -->
        <div class="ticket-detail-grid">
          <div class="detail-item">
            <span class="dlabel">Kursi</span>
            <span class="dval">{{ ticket.seat }} <span class="dval-sub">({{ ticket.type }})</span></span>
          </div>
          <div class="detail-item">
            <span class="dlabel">Telepon</span>
            <span class="dval">{{ ticket.phone }}</span>
          </div>
          <div class="detail-item">
            <span class="dlabel">Pembayaran</span>
            <span class="dval">{{ ticket.paymentMethod }}</span>
          </div>
          <div class="detail-item">
            <span class="dlabel">Total</span>
            <span class="dval">{{ ticket.totalPrice }}</span>
          </div>
        </div>

        <div class="ticket-divider"></div>

        <!-- Footer actions -->
        <div class="ticket-footer">
          <button class="lihat-detail-btn" @click="openDetail(ticket)">Lihat Detail</button>
          <button class="qr-icon-btn" @click="openQR(ticket)" title="Lihat QR">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="qr-icon-svg">
              <rect x="3" y="3" width="7" height="7" rx="1"/>
              <rect x="14" y="3" width="7" height="7" rx="1"/>
              <rect x="3" y="14" width="7" height="7" rx="1"/>
              <rect x="5" y="5" width="3" height="3" fill="currentColor" stroke="none"/>
              <rect x="16" y="5" width="3" height="3" fill="currentColor" stroke="none"/>
              <rect x="5" y="16" width="3" height="3" fill="currentColor" stroke="none"/>
              <path d="M14 14h2v2h-2zM18 14h2v2h-2zM14 18h2v2h-2zM18 18h2v2h-2z" fill="currentColor" stroke="none"/>
            </svg>
          </button>
        </div>
      </div>
    </div>

    <!-- Event Picker Modal -->
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
              :class="{ selected: selectedEvent && selectedEvent.id === ev.id }"
              @click="selectEvent(ev)"
            >{{ ev.title }}</div>
          </div>
        </div>
      </div>
    </transition>

    <!-- OTS Detail Modal -->
    <transition name="modal-fade">
      <div class="modal-overlay" v-if="showDetailModal && selectedTicket" @click.self="showDetailModal = false">
        <div
          class="modal-sheet"
          :style="detailStyle"
          @touchstart="handleDragStart($event, 'detail')"
          @touchmove="handleDragMove"
          @touchend="handleDragEnd"
          @mousedown="handleDragStart($event, 'detail')"
        >
          <div class="modal-handle"></div>
          <div class="detail-modal-header">
            <h3 class="modal-title">Detail Tiket OTS</h3>
            <button class="close-x-btn" @click="showDetailModal = false">
              <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" class="close-icon">
                <path stroke-linecap="round" stroke-linejoin="round" d="M6 18 18 6M6 6l12 12" />
              </svg>
            </button>
          </div>

          <div class="detail-info-grid detail-grid">
            <div class="detail-row"><span class="lbl">Invoice</span><span class="val">{{ selectedTicket.invoice }}</span></div>
            <div class="detail-row"><span class="lbl">Nama</span><span class="val">{{ selectedTicket.name }}</span></div>
            <div class="detail-row"><span class="lbl">Telepon</span><span class="val">{{ selectedTicket.phone }}</span></div>
            <div class="detail-row"><span class="lbl">Email</span><span class="val">{{ selectedTicket.email }}</span></div>
            <div class="detail-row"><span class="lbl">Kursi</span><span class="val">{{ selectedTicket.seat }} ({{ selectedTicket.type }})</span></div>
            <div class="detail-row"><span class="lbl">Jumlah Tiket</span><span class="val">{{ selectedTicket.qty }} tiket</span></div>
            <div class="detail-row"><span class="lbl">Total Bayar</span><span class="val">{{ selectedTicket.totalPrice }}</span></div>
            <div class="detail-row"><span class="lbl">Metode Bayar</span><span class="val">{{ selectedTicket.paymentMethod }}</span></div>
            <div class="detail-row">
              <span class="lbl">Status</span>
              <span class="val" :style="{ color: selectedTicket.status === 'Lunas' ? '#16a34a' : '#ea580c' }">{{ selectedTicket.status }}</span>
            </div>
          </div>

          <button v-if="selectedTicket.status !== 'Lunas'" class="checkin-now-btn" @click="confirmPayment(selectedTicket)">
            Konfirmasi Lunas
          </button>
        </div>
      </div>
    </transition>

    <!-- QR Code Modal -->
    <transition name="modal-fade">
      <div class="modal-overlay" v-if="showQRModal && selectedQRTicket" @click.self="showQRModal = false">
        <div
          class="modal-sheet qr-sheet"
          :style="qrStyle"
          @touchstart="handleDragStart($event, 'qr')"
          @touchmove="handleDragMove"
          @touchend="handleDragEnd"
          @mousedown="handleDragStart($event, 'qr')"
        >
          <div class="modal-handle"></div>
          <div class="detail-modal-header">
            <h3 class="modal-title">QR Tiket OTS</h3>
            <button class="close-x-btn" @click="showQRModal = false">
              <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" class="close-icon">
                <path stroke-linecap="round" stroke-linejoin="round" d="M6 18 18 6M6 6l12 12" />
              </svg>
            </button>
          </div>

          <!-- QR Code image generated from invoice -->
          <div class="qr-code-wrapper">
            <img
              :src="`https://api.qrserver.com/v1/create-qr-code/?size=180x180&data=${selectedQRTicket.invoice}`"
              :alt="selectedQRTicket.invoice"
              class="qr-code-img"
            />
          </div>

          <!-- Buyer info below QR -->
          <div class="qr-info">
            <p class="qr-buyer-name">{{ selectedQRTicket.name }}</p>
            <p class="qr-invoice">{{ selectedQRTicket.invoice }}</p>
          </div>
        </div>
      </div>
    </transition>

    <!-- Create OTS Form Modal -->
    <transition name="modal-fade">
      <div class="modal-overlay" v-if="showCreateForm" @click.self="showCreateForm = false">
        <div
          class="modal-sheet create-sheet"
          :style="createStyle"
          @touchstart="handleDragStart($event, 'create')"
          @touchmove="handleDragMove"
          @touchend="handleDragEnd"
          @mousedown="handleDragStart($event, 'create')"
        >
          <div class="modal-handle"></div>
          <div class="detail-modal-header">
            <h3 class="modal-title">Tambah Tiket OTS</h3>
            <button class="close-x-btn" @click="showCreateForm = false">
              <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" class="close-icon">
                <path stroke-linecap="round" stroke-linejoin="round" d="M6 18 18 6M6 6l12 12" />
              </svg>
            </button>
          </div>

          <div class="form-grid">
            <div class="form-field">
              <label>Nama Pembeli *</label>
              <input v-model="form.name" type="text" placeholder="Nama lengkap" class="form-input" />
            </div>
            <div class="form-field">
              <label>No. Telepon *</label>
              <input v-model="form.phone" type="tel" placeholder="08xxxxxxxxxx" class="form-input" />
            </div>
            <div class="form-field">
              <label>Email</label>
              <input v-model="form.email" type="email" placeholder="email@domain.com" class="form-input" />
            </div>
            <div class="form-field">
              <label>No. Kursi</label>
              <input v-model="form.seat" type="text" placeholder="Contoh: A-01" class="form-input" />
            </div>
            <div class="form-field">
              <label>Tipe Tiket</label>
              <select v-model="form.type" class="form-input">
                <option>Regular</option>
                <option>VIP</option>
                <option>Festival</option>
              </select>
            </div>
            <div class="form-field">
              <label>Metode Bayar</label>
              <select v-model="form.paymentMethod" class="form-input">
                <option>Tunai</option>
                <option>QRIS</option>
                <option>Transfer</option>
              </select>
            </div>
          </div>

          <button class="checkin-now-btn" @click="submitForm">Simpan Tiket OTS</button>
        </div>
      </div>
    </transition>
  </div>
</template>

<style scoped>
.ots-container {
  padding: 16px;
  background-color: #f8fafc;
  min-height: calc(100vh - 56px - 60px);
  font-family: var(--font-sans);
}

/* Stats card */
.stats-overview-card {
  background-color: white;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  padding: 16px;
  display: flex;
  justify-content: space-around;
  align-items: center;
  margin-bottom: 14px;
  box-shadow: 0 1px 4px rgba(0,0,0,0.03);
}
.overview-col {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
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

/* Event selector button */
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
.btn-left {
  display: flex;
  align-items: center;
  gap: 10px;
  min-width: 0;
}
.event-btn-title {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 210px;
}
.calendar-icon { width: 18px; height: 18px; flex-shrink: 0; }
.chevron-right-icon { width: 16px; height: 16px; flex-shrink: 0; }

/* Search */
.search-wrapper { position: relative; margin-bottom: 14px; }
.search-icon { position: absolute; left: 13px; top: 13px; width: 17px; height: 17px; color: #94a3b8; }
.search-input {
  width: 100%;
  padding: 11px 14px 11px 40px;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  font-size: 13px;
  outline: none;
  background-color: white;
}

/* Summary row */
.summary-info-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 13px;
  font-weight: 500;
  color: #475569;
  margin-bottom: 14px;
}
.add-ticket-btn {
  background-color: #194E9E;
  color: white;
  border: none;
  border-radius: 6px;
  padding: 6px 12px;
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
}

/* Tickets list */
.tickets-list { display: flex; flex-direction: column; gap: 12px; }
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

/* Ticket card */
.ticket-card {
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
.ticket-name-group { min-width: 0; }
.ticket-name {
  font-size: 15px;
  font-weight: 700;
  color: #0f172a;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.ticket-invoice { font-size: 12px; color: #94a3b8; margin-top: 2px; }

/* Status badge */
.status-badge {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 10px;
  font-weight: 700;
  padding: 3px 8px;
  border-radius: 20px;
  white-space: nowrap;
  flex-shrink: 0;
}
.badge-lunas    { background-color: #065f46; color: #d1fae5; }
.badge-menunggu { background-color: #991b1b; color: #fee2e2; }
.badge-icon { width: 10px; height: 10px; }

/* Detail grid */
.ticket-detail-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}
.detail-item { display: flex; flex-direction: column; gap: 3px; }
.dlabel { font-size: 10px; font-weight: 500; color: #94a3b8; }
.dval { font-size: 13px; font-weight: 600; color: #0f172a; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.dval-sub { color: #94a3b8; font-weight: 400; }

/* Divider */
.ticket-divider { border-top: 1px dotted #e2e8f0; }

/* Footer buttons */
.ticket-footer { display: flex; width: 100%; gap: 8px; align-items: center; }

.lihat-detail-btn {
  flex: 1;
  background-color: #194E9E;
  border: none;
  border-radius: 8px;
  padding: 10px;
  text-align: center;
  font-size: 13px;
  font-weight: 700;
  color: white;
  cursor: pointer;
  transition: background-color 0.15s;
}
.lihat-detail-btn:hover { background-color: #1453b6; }

/* QR icon button */
.qr-icon-btn {
  flex-shrink: 0;
  width: 38px;
  height: 38px;
  background-color: #f1f5f9;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  color: #475569;
  transition: background-color 0.15s, color 0.15s;
}
.qr-icon-btn:hover {
  background-color: #e8eef8;
  color: #194E9E;
}
.qr-icon-svg { width: 18px; height: 18px; }

.action-checkin-btn {
  flex: 1;
  background-color: #194E9E;
  color: white;
  border: none;
  border-radius: 8px;
  padding: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  font-size: 12px;
  font-weight: 700;
  cursor: pointer;
}
.grid-icon { width: 14px; height: 14px; }

.action-dots-btn {
  background-color: #f1f5f9;
  color: #64748b;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 10px 14px;
  font-size: 13px;
  font-weight: 700;
  cursor: pointer;
}

/* Modal sheet */
.modal-overlay {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
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
  gap: 14px;
  touch-action: none;
}
.create-sheet { max-height: 85vh; overflow-y: auto; }
.modal-handle {
  width: 40px;
  height: 4px;
  background-color: #cbd5e1;
  border-radius: 2px;
  align-self: center;
}
.detail-modal-header { display: flex; justify-content: space-between; align-items: center; }
.modal-title { font-size: 15px; font-weight: 700; color: #0f172a; }
.close-x-btn { background: none; border: none; cursor: pointer; padding: 4px; color: #94a3b8; }
.close-icon { width: 20px; height: 20px; }

.modal-list { display: flex; flex-direction: column; gap: 8px; }
.modal-item { padding: 12px; border-radius: 8px; border: 1px solid #e2e8f0; font-size: 13px; cursor: pointer; }
.modal-item.selected { border-color: #194E9E; background-color: #f0f6ff; color: #194E9E; font-weight: 600; }

/* Detail modal info */
.detail-grid { display: flex; flex-direction: column; gap: 10px; }
.detail-row { display: flex; justify-content: space-between; font-size: 12px; }
.detail-row .lbl { color: #64748b; font-weight: 400; }
.detail-row .val { font-weight: 600; color: #1e293b; }

.checkin-now-btn {
  background-color: #16a34a;
  color: white;
  border: none;
  border-radius: 8px;
  padding: 12px;
  font-size: 13px;
  font-weight: 700;
  cursor: pointer;
}

/* Create form */
.form-grid { display: flex; flex-direction: column; gap: 12px; }
.form-field { display: flex; flex-direction: column; gap: 4px; }
.form-field label { font-size: 11px; font-weight: 600; color: #475569; }
.form-input {
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 10px 12px;
  font-size: 13px;
  outline: none;
  background-color: white;
}

/* Modal animations */
.modal-fade-enter-active, .modal-fade-leave-active { transition: opacity 0.2s ease; }
.modal-fade-enter-active .modal-sheet, .modal-fade-leave-active .modal-sheet { transition: transform 0.25s cubic-bezier(0.16, 1, 0.3, 1); }
.modal-fade-enter-from, .modal-fade-leave-to { opacity: 0; }
.modal-fade-enter-from .modal-sheet, .modal-fade-leave-to .modal-sheet { transform: translateY(100%); }

/* QR modal specific */
.qr-sheet {
  align-items: center;
  padding-bottom: 28px;
}
.qr-code-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  padding: 18px;
  width: 100%;
}
.qr-code-img {
  width: 180px;
  height: 180px;
  display: block;
  image-rendering: pixelated;
}
.qr-info {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  text-align: center;
  width: 100%;
}
.qr-buyer-name {
  font-size: 16px;
  font-weight: 700;
  color: #0f172a;
}
.qr-invoice {
  font-size: 12px;
  font-weight: 500;
  color: #94a3b8;
  letter-spacing: 0.5px;
}
</style>

