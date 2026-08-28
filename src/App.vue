<script setup>
import { ref, computed } from 'vue';
import Login from './components/Login.vue';
import EventList from './components/EventList.vue';
import Checkin from './components/Checkin.vue';
import CheckinReport from './components/CheckinReport.vue';
import OtsTickets from './components/OtsTickets.vue';

const currentScreen = ref('login'); // 'login' or 'app'
const activeTab = ref('checkin'); // 'event', 'checkin', 'report', 'ots'
const staffName = ref('');
const isSidebarOpen = ref(false);

const events = ref([
  {
    id: 1,
    title: 'Ngamen 0.5',
    price: 'Rp124.000',
    organizer: 'Maxpaincompany LTD',
    creatorLogo: 'https://images.unsplash.com/photo-1570295999919-56ceb5ecca61?auto=format&fit=crop&w=80&q=80',
    location: 'Karawang',
    date: 'Sat, 24 Aug 2024',
    sold: 150,
    total: 200,
    status: 'Live',
    image: 'https://images.unsplash.com/photo-1516450360452-9312f5e86fc7?auto=format&fit=crop&w=400&q=80'
  },
  {
    id: 2,
    title: 'SIKSAKUBUR - Tiga Dekade Melawan Tunduk',
    price: 'Rp8.000',
    organizer: 'Newhope.inc',
    creatorLogo: 'https://images.unsplash.com/photo-1535713875002-d1d0cf377fde?auto=format&fit=crop&w=80&q=80',
    location: 'IKJ',
    date: 'Sun, 25 Aug 2024',
    sold: 500,
    total: 1000,
    status: 'Live',
    image: 'https://images.unsplash.com/photo-1470225620780-dba8ba36b745?auto=format&fit=crop&w=400&q=80'
  },
  {
    id: 3,
    title: 'Straight Answer 30 Years Of Persistence',
    price: 'Rp85.000',
    organizer: 'Smartex Bomb Records',
    creatorLogo: 'https://images.unsplash.com/photo-1580489944761-15a19d654956?auto=format&fit=crop&w=80&q=80',
    location: 'Fatmawati',
    date: 'Sat, 31 Aug 2024',
    sold: 80,
    total: 100,
    status: 'Upcoming',
    image: 'https://images.unsplash.com/photo-1506157786151-b8491531f063?auto=format&fit=crop&w=400&q=80'
  },
  {
    id: 4,
    title: 'Intimate Show MORAD',
    price: 'Rp80.000',
    organizer: 'Morad Music Asia',
    creatorLogo: 'https://images.unsplash.com/photo-1438761681033-6461ffad8d80?auto=format&fit=crop&w=80&q=80',
    location: 'TBA',
    date: 'Wed, 04 Sep 2024',
    sold: 200,
    total: 200,
    status: 'Live',
    isSoldOut: true,
    image: 'https://images.unsplash.com/photo-1501386761578-eac5c94b800a?auto=format&fit=crop&w=400&q=80'
  },
  {
    id: 5,
    title: 'Rooted Relevant Collective',
    price: 'Rp350.000',
    organizer: 'Independent Organizer',
    creatorLogo: 'https://images.unsplash.com/photo-1628157582853-a796fa650a6a?auto=format&fit=crop&w=80&q=80',
    location: 'Jakarta',
    date: 'Fri, 13 Sep 2024',
    sold: 45,
    total: 100,
    status: 'Live',
    image: 'https://images.unsplash.com/photo-1492684223066-81342ee5ff30?auto=format&fit=crop&w=400&q=80'
  },
  {
    id: 6,
    title: 'ROEANG DUARIBU',
    price: 'Rp120.000',
    organizer: 'Creative Hub',
    creatorLogo: 'https://images.unsplash.com/photo-1472099645785-5658abf4ff4e?auto=format&fit=crop&w=80&q=80',
    location: 'Bandung',
    date: 'Sat, 21 Sep 2024',
    sold: 45,
    total: 100,
    status: 'Live',
    image: 'https://images.unsplash.com/photo-1504609773096-104ff2c73ba4?auto=format&fit=crop&w=400&q=80'
  }
]);

const selectedEvent = ref(null);

const handleLoginSuccess = (name) => {
  staffName.value = name;
  currentScreen.value = 'app';
  activeTab.value = 'checkin'; // Directly open camera/scanner as requested
};

const handleSelectEvent = (event) => {
  selectedEvent.value = event;
  activeTab.value = 'checkin'; // Switches to checkin (camera) immediately
};

const navigateTo = (tab) => {
  activeTab.value = tab;
  if (tab === 'checkin') {
    selectedEvent.value = null; // Shows All Event (without stats)
  }
  isSidebarOpen.value = false;
};

const handleLogout = () => {
  currentScreen.value = 'login';
  isSidebarOpen.value = false;
};

// Hide header & navbar when scanner is active
const isCheckinActive = computed(() => activeTab.value === 'checkin');
</script>

<template>
  <Login v-if="currentScreen === 'login'" @login-success="handleLoginSuccess" />
  <div v-else class="mobile-wrapper">
    <!-- Top Nav Bar — hidden smoothly when on Checkin tab -->
    <transition name="slide-top">
      <header v-show="!isCheckinActive" class="navbar-header">
        <div class="nav-left-group">
          <button class="nav-menu-btn" @click="isSidebarOpen = true">
            <!-- Hamburger Menu Icon -->
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" class="nav-icon">
              <path stroke-linecap="round" stroke-linejoin="round" d="M3.75 6.75h16.5M3.75 12h16.5m-16.5 5.25h16.5" />
            </svg>
          </button>
          <div class="navbar-logo-container">
            <img src="/media/logo.png" alt="Kolektix Logo" class="nav-logo" />
          </div>
        </div>

        <div class="nav-right-group" @click="navigateTo('profile')" style="cursor: pointer;">
          <div class="profile-circle" style="width: 32px; height: 32px; border-radius: 50%; background-color: rgba(255,255,255,0.2); border: 1.5px solid white; display: flex; align-items: center; justify-content: center;">
            <span style="color: white; font-size: 11px; font-weight: 700;">{{ staffName ? staffName.split(' ').map(n => n[0]).join('') : 'S' }}</span>
          </div>
        </div>
      </header>
    </transition>

    <!-- Main Content Area -->
    <main class="content-scroll-area">
      <EventList v-if="activeTab === 'event'" :events="events" @select-event="handleSelectEvent" />
      <Checkin v-else-if="activeTab === 'checkin'" :events="events" :selectedEvent="selectedEvent" @close-scanner="navigateTo('event')" />
      <CheckinReport v-else-if="activeTab === 'report'" :events="events" />
      <OtsTickets v-else-if="activeTab === 'ots'" :events="events" />
      
      <!-- Profile Tab Content -->
      <div v-else-if="activeTab === 'profile'" class="profile-tab-content">
        <div class="profile-card">
          <div class="profile-avatar">
            {{ staffName ? staffName.split(' ').map(n => n[0]).join('') : 'ST' }}
          </div>
          <h3>{{ staffName || 'Staff Scanner' }}</h3>
          <span class="role-badge">Staff Scanner</span>
        </div>

        <div class="profile-info-list">
          <div class="info-item">
            <span class="lbl">USERNAME</span>
            <span class="val">@{{ staffName ? staffName.toLowerCase().replace(' ', '') : 'staff' }}</span>
          </div>
          <div class="info-item">
            <span class="lbl">ROLE</span>
            <span class="val">Scanner Operator</span>
          </div>
          <div class="info-item">
            <span class="lbl">STATUS</span>
            <span class="val status-active">Aktif</span>
          </div>
        </div>

        <button class="logout-action-btn" @click="handleLogout">
          Keluar Akun (Logout)
        </button>
      </div>
    </main>

    <!-- Bottom Mobile Navigation Bar — hidden smoothly when on Checkin tab -->
    <transition name="slide-bottom">
    <nav v-show="!isCheckinActive" class="bottom-nav">
      <!-- Event Tab -->
      <button class="nav-tab" :class="{ active: activeTab === 'event' }" @click="navigateTo('event')">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="tab-icon">
          <rect x="3" y="4" width="18" height="16" rx="2" stroke-linecap="round" stroke-linejoin="round" />
          <line x1="3" y1="9" x2="21" y2="9" stroke-linecap="round" stroke-linejoin="round" />
        </svg>
        <span class="tab-label">Event</span>
      </button>

      <!-- Checkin Tab (Directly Opens Camera) -->
      <button class="nav-tab" :class="{ active: activeTab === 'checkin' }" @click="navigateTo('checkin')">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="tab-icon">
          <path d="M4 8V4h4M20 8V4h-4M4 16v4h4M20 16v4h-4" stroke-linecap="round" stroke-linejoin="round" />
          <line x1="6" y1="12" x2="18" y2="12" stroke-linecap="round" stroke-linejoin="round" />
        </svg>
        <span class="tab-label">Checkin</span>
      </button>

      <!-- Report Tab -->
      <button class="nav-tab" :class="{ active: activeTab === 'report' }" @click="navigateTo('report')">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="tab-icon">
          <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z" />
          <polyline points="14 2 14 8 20 8" />
          <line x1="16" y1="13" x2="8" y2="13" />
        </svg>
        <span class="tab-label">Report</span>
      </button>

      <!-- OTS Tab -->
      <button class="nav-tab" :class="{ active: activeTab === 'ots' }" @click="navigateTo('ots')">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="tab-icon">
          <rect x="2" y="6" width="20" height="12" rx="2" ry="2"/>
          <line x1="6" y1="6" x2="6" y2="18"/>
          <line x1="18" y1="6" x2="18" y2="18"/>
        </svg>
        <span class="tab-label">OTS</span>
      </button>

      <!-- Profile Tab -->
      <button class="nav-tab" :class="{ active: activeTab === 'profile' }" @click="navigateTo('profile')">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="tab-icon">
          <path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2" />
          <circle cx="12" cy="7" r="4" />
        </svg>
        <span class="tab-label">Profile</span>
      </button>
    </nav>
    </transition>

    <!-- Sidebar Sliding Drawer Menu -->
    <transition name="fade">
      <div v-if="isSidebarOpen" class="sidebar-overlay" @click="isSidebarOpen = false"></div>
    </transition>
    
    <transition name="slide-sidebar">
      <div v-if="isSidebarOpen" class="sidebar-drawer">
        <!-- Sidebar Welcome Greeting Card -->
        <div class="sidebar-greeting-card">
          <div class="greeting-avatar">
            {{ staffName.split(' ').map(n => n[0]).join('') }}
          </div>
          <div class="greeting-text">
            <h3>Hi, {{ staffName }}</h3>
            <p>Staff Scanner</p>
          </div>
        </div>

        <!-- Sidebar Navigation Buttons (matching mobile navbar) -->
        <nav class="sidebar-nav">
          <button class="sidebar-nav-item" :class="{ active: activeTab === 'event' }" @click="navigateTo('event')">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="nav-icon">
              <rect x="3" y="4" width="18" height="16" rx="2" stroke-linecap="round" stroke-linejoin="round" />
              <line x1="3" y1="9" x2="21" y2="9" stroke-linecap="round" stroke-linejoin="round" />
            </svg>
            <span>Event saya</span>
          </button>

          <button class="sidebar-nav-item" :class="{ active: activeTab === 'checkin' }" @click="navigateTo('checkin')">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="nav-icon">
              <path d="M4 8V4h4M20 8V4h-4M4 16v4h4M20 16v4h-4" stroke-linecap="round" stroke-linejoin="round" />
              <line x1="6" y1="12" x2="18" y2="12" stroke-linecap="round" stroke-linejoin="round" />
            </svg>
            <span>Check In Event</span>
          </button>

          <button class="sidebar-nav-item" :class="{ active: activeTab === 'report' }" @click="navigateTo('report')">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="nav-icon">
              <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z" />
              <polyline points="14 2 14 8 20 8" />
              <line x1="16" y1="13" x2="8" y2="13" />
            </svg>
            <span>Check In Report</span>
          </button>

          <button class="sidebar-nav-item" :class="{ active: activeTab === 'ots' }" @click="navigateTo('ots')">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="nav-icon">
              <rect x="2" y="6" width="20" height="12" rx="2" ry="2"/>
              <line x1="6" y1="6" x2="6" y2="18"/>
              <line x1="18" y1="6" x2="18" y2="18"/>
            </svg>
            <span>Ticket OTS</span>
          </button>
        </nav>

        <!-- Sidebar footer: close sidebar button -->
        <div class="sidebar-footer">
          <button class="sidebar-nav-item close-sidebar-btn" @click="isSidebarOpen = false">
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2.5" stroke="currentColor" class="nav-icon">
              <path stroke-linecap="round" stroke-linejoin="round" d="M6 18 18 6M6 6l12 12" />
            </svg>
            <span>Tutup Menu</span>
          </button>
        </div>
      </div>
    </transition>
  </div>
</template>

<style scoped>
.mobile-wrapper {
  width: 100%;
  max-width: 100%;
  height: 100vh;
  height: 100dvh;
  background-color: #fcfcfd;
  display: flex;
  flex-direction: column;
  position: relative;
  overflow: hidden;
}

@media (min-width: 480px) {
  .mobile-wrapper {
    max-width: 410px;
    height: 88vh;
    max-height: 840px;
    border-radius: 30px;
    box-shadow: 0 12px 30px rgba(0, 0, 0, 0.1);
    margin: auto;
  }
}

/* Navbar styles */
.navbar-header {
  background-color: var(--primary-base);
  height: 56px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 16px;
  flex-shrink: 0;
}

.nav-left-group {
  display: flex;
  align-items: center;
  gap: 12px;
}

.nav-menu-btn {
  background: none;
  border: none;
  color: white;
  cursor: pointer;
  display: flex;
  align-items: center;
}

.nav-icon {
  width: 24px;
  height: 24px;
}

.navbar-logo-container {
  display: flex;
  align-items: center;
  justify-content: center;
}

.nav-logo {
  max-height: 28px;
  width: auto;
}

/* Content Scrollable area */
.content-scroll-area {
  flex-grow: 1;
  overflow-y: auto;
  scrollbar-width: none;
}

.content-scroll-area::-webkit-scrollbar {
  display: none;
}

/* Bottom Tab Navigation Bar */
.bottom-nav {
  height: 60px;
  border-top: 1px solid #e2e8f0;
  background-color: white;
  display: flex;
  justify-content: space-around;
  align-items: center;
  flex-shrink: 0;
  z-index: 10;
  border-top-left-radius: 16px;
  border-top-right-radius: 16px;
  box-shadow: 0 -4px 12px rgba(0, 0, 0, 0.05);
}

.nav-tab {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: none;
  border: none;
  cursor: pointer;
  color: #64748b;
  gap: 4px;
  position: relative;
  height: 100%;
  flex: 1;
}

.nav-tab.active::before {
  content: '';
  position: absolute;
  top: 0;
  left: 25%;
  right: 25%;
  height: 3px;
  background-color: #194E9E;
  border-radius: 0 0 3px 3px;
}

.tab-icon {
  width: 20px;
  height: 20px;
}

.tab-label {
  font-size: 10px;
  font-weight: 500;
}

.nav-tab.active {
  color: #194E9E;
}

/* Sidebar sliding drawer */
.sidebar-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.4);
  z-index: 99;
}

.sidebar-drawer {
  position: absolute;
  top: 0;
  left: 0;
  width: 280px;
  height: 100%;
  background-color: #03204e;
  color: white;
  z-index: 100;
  display: flex;
  flex-direction: column;
  box-shadow: 4px 0 16px rgba(0,0,0,0.15);
  font-family: var(--font-sans);
}

.sidebar-greeting-card {
  padding: 24px 20px;
  display: flex;
  align-items: center;
  gap: 12px;
  background-color: rgba(255,255,255,0.03);
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
}

.greeting-avatar {
  width: 42px;
  height: 42px;
  border-radius: 50%;
  background-color: rgba(255,255,255,0.1);
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 16px;
  color: white;
}

.greeting-text h3 {
  font-size: 14px;
  margin: 0;
  font-weight: 600;
}

.greeting-text p {
  font-size: 11px;
  color: rgba(255,255,255,0.6);
  margin: 2px 0 0 0;
}

.sidebar-nav {
  padding: 16px 10px;
  display: flex;
  flex-direction: column;
  gap: 4px;
  flex-grow: 1;
}

.sidebar-nav-item {
  width: 100%;
  background: none;
  border: none;
  padding: 12px 14px;
  display: flex;
  align-items: center;
  gap: 12px;
  border-radius: 8px;
  color: rgba(255, 255, 255, 0.7);
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
  text-align: left;
}

.sidebar-nav-item:hover, .sidebar-nav-item.active {
  background-color: rgba(255, 255, 255, 0.08);
  color: white;
}

.sidebar-footer {
  padding: 10px;
  border-top: 1px solid rgba(255, 255, 255, 0.08);
}

.logout-btn {
  color: #fca5a5 !important;
}

.logout-btn:hover {
  background-color: rgba(239, 68, 68, 0.1) !important;
}

/* Sidebar Animation */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.2s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

.slide-sidebar-enter-active, .slide-sidebar-leave-active {
  transition: transform 0.3s cubic-bezier(0.16, 1, 0.3, 1);
}
.slide-sidebar-enter-from, .slide-sidebar-leave-to {
  transform: translateX(-100%);
}

/* Profile Tab Styles */
.profile-tab-content {
  padding: 24px 16px;
  display: flex;
  flex-direction: column;
  gap: 20px;
  background-color: #f8fafc;
  min-height: calc(100vh - 56px - 60px);
  font-family: var(--font-sans);
}

.profile-card {
  background-color: white;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  padding: 24px;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  gap: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.02);
}

.profile-avatar {
  width: 64px;
  height: 64px;
  border-radius: 50%;
  background-color: #f0f6ff;
  color: #194E9E;
  font-size: 22px;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px solid #bfdbfe;
  margin-bottom: 4px;
}

.profile-card h3 {
  font-size: 16px;
  font-weight: 700;
  color: #0f172a;
}

.role-badge {
  background-color: #eff6ff;
  color: #1e40af;
  font-size: 10px;
  font-weight: 600;
  padding: 4px 10px;
  border-radius: 20px;
  border: 1px solid #bfdbfe;
}

.profile-info-list {
  background-color: white;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  padding: 8px 16px;
  display: flex;
  flex-direction: column;
  box-shadow: 0 2px 8px rgba(0,0,0,0.02);
}

.info-item {
  display: flex;
  justify-content: space-between;
  padding: 12px 0;
  border-bottom: 1px solid #f1f5f9;
  font-size: 12px;
}

.info-item:last-child {
  border-bottom: none;
}

.info-item .lbl {
  color: #64748b;
  font-weight: 500;
}

.info-item .val {
  color: #0f172a;
  font-weight: 600;
}

.info-item .val.status-active {
  color: #16a34a;
}

.logout-action-btn {
  background-color: #fee2e2;
  color: #b91c1c;
  border: 1px solid #fecaca;
  border-radius: 8px;
  padding: 12px;
  font-size: 13px;
  font-weight: 700;
  cursor: pointer;
  transition: background-color 0.2s;
  text-align: center;
  margin-top: 8px;
}

.logout-action-btn:hover {
  background-color: #fca5a5;
}

/* Close sidebar button style */
.close-sidebar-btn {
  color: #64748b;
}
.close-sidebar-btn:hover {
  background-color: #f1f5f9;
  color: #0f172a;
}

/* ─── Smooth show/hide for header (slides up) ─── */
.slide-top-enter-active,
.slide-top-leave-active {
  transition: max-height 0.35s cubic-bezier(0.4, 0, 0.2, 1),
              opacity 0.3s ease;
  overflow: hidden;
}
.slide-top-enter-from,
.slide-top-leave-to {
  max-height: 0 !important;
  opacity: 0;
}
.slide-top-enter-to,
.slide-top-leave-from {
  max-height: 56px;
  opacity: 1;
}

/* ─── Smooth show/hide for bottom nav (slides down) ─── */
.slide-bottom-enter-active,
.slide-bottom-leave-active {
  transition: max-height 0.35s cubic-bezier(0.4, 0, 0.2, 1),
              opacity 0.3s ease;
  overflow: hidden;
}
.slide-bottom-enter-from,
.slide-bottom-leave-to {
  max-height: 0 !important;
  opacity: 0;
}
.slide-bottom-enter-to,
.slide-bottom-leave-from {
  max-height: 60px;
  opacity: 1;
}
</style>

