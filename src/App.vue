<template>
  <div class="display-root" ref="containerRef">
    <div class="dlt-stripe"></div>
    <!-- Animated background -->
    <div class="bg-layer">
      <div class="bg-orb bg-orb-1"></div>
      <div class="bg-orb bg-orb-2"></div>
      <div class="bg-orb bg-orb-3"></div>
      <div class="bg-grid"></div>
    </div>

    <!-- Top bar -->
    <header class="top-bar">
      <div class="top-bar-left">
        <div class="logo-mark">
          <svg viewBox="0 0 64 64" fill="none" aria-hidden="true">
            <rect x="14" y="16" width="36" height="34" rx="7" stroke="currentColor" stroke-width="3" />
            <rect x="14" y="16" width="36" height="9" rx="4.5" fill="currentColor" />
            <rect x="22" y="11" width="3.5" height="9" rx="1.75" fill="currentColor" />
            <rect x="38.5" y="11" width="3.5" height="9" rx="1.75" fill="currentColor" />
            <g fill="currentColor" opacity="0.45">
              <rect x="20" y="30" width="7" height="6" rx="2" />
              <rect x="29.5" y="30" width="7" height="6" rx="2" />
              <rect x="20" y="39" width="7" height="6" rx="2" />
              <rect x="29.5" y="39" width="7" height="6" rx="2" />
              <rect x="39" y="39" width="7" height="6" rx="2" />
            </g>
            <rect x="37.5" y="28.5" width="10" height="9" rx="3" fill="#ed1c24" />
            <path d="M40.3 33.2l1.6 1.6 3.1-3.3" stroke="#fff" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" />
          </svg>
        </div>
        <div class="brand">
          <p class="brand-org">กรมการขนส่งทางบก</p>
          <h1 class="brand-title">ตารางการประชุม</h1>
          <p class="brand-date">{{ dateDisplay }}</p>
        </div>
      </div>

      <div class="clock-display">
        <div class="clock-time">{{ timeDisplay }}</div>
        <div class="clock-pulse"></div>
      </div>

      <div class="top-bar-right">
        <button @click="toggleRoomFilter" class="icon-btn" title="ตัวกรองห้องประชุม">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M22 3H2l8 9.46V19l4 2v-8.54L22 3z" />
          </svg>
        </button>
        <button @click="toggleFullscreen" class="icon-btn" title="แสดงเต็มจอ">
          <svg v-if="!isFullscreen" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M8 3H5a2 2 0 0 0-2 2v3m18 0V5a2 2 0 0 0-2-2h-3m0 18h3a2 2 0 0 0 2-2v-3M3 16v3a2 2 0 0 0 2 2h3" />
          </svg>
          <svg v-else viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M4 14h6v6m10-10h-6V4m0 6l7-7M3 21l7-7" />
          </svg>
        </button>
      </div>
    </header>

    <!-- Room filter panel -->
    <transition name="slide-down">
      <div v-if="showRoomFilter" class="filter-panel">
        <div class="filter-inner">
          <h3 class="filter-heading">เลือกห้องประชุมที่ต้องการแสดง</h3>
          <div class="filter-chips">
            <label v-for="room in availableRooms" :key="room.id" class="chip"
              :class="{ active: selectedRoomIds.includes(room.id) }">
              <input type="checkbox" :value="room.id" v-model="selectedRoomIds" @change="saveRoomPreferences" />
              <span class="chip-dot"></span>
              {{ room.name }}
            </label>
          </div>
          <div class="filter-actions">
            <button @click="selectAllRooms" class="action-btn secondary">เลือกทั้งหมด</button>
            <button @click="clearRoomSelection" class="action-btn danger">ยกเลิกทั้งหมด</button>
            <button @click="toggleRoomFilter" class="action-btn primary">ปิด</button>
          </div>
        </div>
      </div>
    </transition>

    <!-- Loading -->
    <div v-if="loading" class="state-box">
      <div class="spinner"></div>
      <p class="state-text">กำลังโหลดข้อมูล...</p>
    </div>

    <!-- Error -->
    <div v-else-if="error" class="state-box error">
      <svg class="state-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <circle cx="12" cy="12" r="10" />
        <line x1="15" y1="9" x2="9" y2="15" />
        <line x1="9" y1="9" x2="15" y2="15" />
      </svg>
      <p class="state-title">เกิดข้อผิดพลาด</p>
      <p class="state-text">{{ error }}</p>
      <button @click="fetchMeetings" class="action-btn primary mt-4">ลองใหม่</button>
    </div>

    <!-- No rooms selected -->
    <div v-else-if="selectedRoomIds.length === 0" class="state-box">
      <svg class="state-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z" />
        <circle cx="12" cy="10" r="3" />
      </svg>
      <p class="state-title">ยังไม่ได้เลือกห้องประชุม</p>
      <button @click="toggleRoomFilter" class="action-btn primary mt-4">เลือกห้องประชุม</button>
    </div>

    <!-- No meetings -->
    <div v-else-if="filteredMeetings.length === 0" class="state-box">
      <svg class="state-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <rect x="3" y="4" width="18" height="18" rx="2" ry="2" />
        <line x1="16" y1="2" x2="16" y2="6" />
        <line x1="8" y1="2" x2="8" y2="6" />
        <line x1="3" y1="10" x2="21" y2="10" />
      </svg>
      <p class="state-title">ไม่มีการประชุมในวันนี้</p>
      <p class="state-text">ห้องที่เลือกไม่มีกำหนดการประชุม</p>
    </div>

    <!-- Meeting list -->
    <div v-else class="meeting-scroll" ref="tableScrollRef" @mouseenter="pauseScroll = true"
      @mouseleave="pauseScroll = false" @touchstart="pauseScroll = true" @touchend="pauseScroll = false">

      <!-- Summary counters -->
      <div class="summary-bar">
        <div class="summary-item ongoing" v-if="countByStatus.ongoing > 0">
          <span class="summary-dot pulse"></span>
          <span class="summary-label">กำลังประชุม</span>
          <span class="summary-count">{{ countByStatus.ongoing }}</span>
        </div>
        <div class="summary-item upcoming" v-if="countByStatus.upcoming > 0">
          <span class="summary-dot"></span>
          <span class="summary-label">รอดำเนินการ</span>
          <span class="summary-count">{{ countByStatus.upcoming }}</span>
        </div>
        <div class="summary-item completed" v-if="countByStatus.completed > 0">
          <span class="summary-dot"></span>
          <span class="summary-label">เสร็จสิ้น</span>
          <span class="summary-count">{{ countByStatus.completed }}</span>
        </div>
      </div>

      <!-- Meeting cards -->
      <div class="meeting-list">
        <div v-for="(meeting, index) in filteredMeetings" :key="meeting.id" class="meeting-card"
          :class="meeting.status">
          <!-- Status accent bar -->
          <div class="card-accent"></div>

          <div class="card-body">
            <!-- Time column -->
            <div class="card-time">
              <span class="time-start">{{ formatTime(meeting.startTime) }}</span>
              <svg class="time-arrow" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
                <path d="M12 5v14M19 12l-7 7-7-7" />
              </svg>
              <span class="time-end">{{ formatTime(meeting.endTime) }}</span>
              <!-- Progress bar for ongoing -->
              <div v-if="meeting.status === 'ongoing'" class="time-progress">
                <div class="time-progress-bar" :style="{ width: getMeetingProgress(meeting) + '%' }"></div>
              </div>
            </div>

            <!-- Info column -->
            <div class="card-info">
              <h3 class="card-title">{{ meeting.title }}</h3>
              <div class="card-meta">
                <span class="meta-item">
                  <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                    <path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z" />
                    <circle cx="12" cy="10" r="3" />
                  </svg>
                  {{ getRoomName(meeting.roomId) }}
                </span>
                <span class="meta-item">
                  <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                    <path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2" />
                    <circle cx="12" cy="7" r="4" />
                  </svg>
                  {{ meeting.organizer }}
                </span>
              </div>
            </div>

            <!-- Status badge -->
            <div class="card-status">
              <span class="badge" :class="'badge-' + meeting.status">
                <span v-if="meeting.status === 'ongoing'" class="badge-pulse"></span>
                {{ getStatusText(meeting.status) }}
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Footer -->
    <footer class="bottom-bar">
      <span>อัพเดทล่าสุด: {{ lastUpdated }}</span>
      <span class="footer-sep">|</span>
      <span>ระบบจองห้องประชุม · กรมการขนส่งทางบก</span>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, onUnmounted } from 'vue';

const containerRef = ref(null);
const isFullscreen = ref(false);

// ===== Auto-scroll =====
const tableScrollRef = ref(null);
const pauseScroll = ref(false);
let scrollInterval = null;
const SCROLL_SPEED = 1;
const SCROLL_DELAY_MS = 3000;

const startAutoScroll = () => {
  if (scrollInterval) return;
  scrollInterval = setInterval(() => {
    const el = tableScrollRef.value;
    if (!el || pauseScroll.value) return;
    const maxScroll = el.scrollHeight - el.clientHeight;
    if (maxScroll <= 0) return;
    if (el.scrollTop >= maxScroll) {
      pauseScroll.value = true;
      setTimeout(() => {
        if (el) el.scrollTop = 0;
        pauseScroll.value = false;
      }, SCROLL_DELAY_MS);
    } else {
      el.scrollTop += SCROLL_SPEED;
    }
  }, 20);
};

const stopAutoScroll = () => {
  if (scrollInterval) {
    clearInterval(scrollInterval);
    scrollInterval = null;
  }
};

// ข้อมูลเวลา
const dateDisplay = ref('');
const timeDisplay = ref('');
const lastUpdated = ref('');

// ข้อมูลการประชุม — ดึงจาก Digital Signage API ของระบบจองห้องประชุม
// ตั้งค่าได้ด้วย VITE_SIGNAGE_API / VITE_SIGNAGE_KEY (ดู .env.example)
const meetings = ref([]);
const loading = ref(true);
const error = ref(null);

const apiUrl = import.meta.env.VITE_SIGNAGE_API || 'https://it-asset.dlt.go.th/meet-reserve/api/signage';
const apiKey = import.meta.env.VITE_SIGNAGE_KEY || '';
const refreshIntervalMs = Number(import.meta.env.VITE_REFRESH_SECONDS || 60) * 1000;

const apiFetch = async (path) => {
  const response = await fetch(`${apiUrl}${path}`, {
    headers: apiKey ? { 'X-Signage-Key': apiKey } : {},
  });
  if (!response.ok) {
    throw new Error(response.status === 401
      ? 'คีย์สำหรับจอแสดงผลไม่ถูกต้อง'
      : `การเชื่อมต่อ API ล้มเหลว (${response.status})`);
  }
  return response.json();
};

const showRoomFilter = ref(false);
const availableRooms = ref([]);
// โหลดห้องสำเร็จแล้วหรือยัง — กันไม่ให้ตัวเลือกของผู้ใช้ถูกเขียนทับตอน refresh
const roomsLoaded = ref(false);
const selectedRoomIds = ref([]);

const filteredMeetings = computed(() => {
  if (selectedRoomIds.value.length === 0) return [];
  const filtered = meetings.value.filter((m) => selectedRoomIds.value.includes(m.roomId));
  const ongoing = filtered.filter((m) => m.status === 'ongoing');
  const upcoming = filtered.filter((m) => m.status === 'upcoming');
  const completed = filtered.filter((m) => m.status === 'completed');
  ongoing.sort((a, b) => new Date(a.startTime) - new Date(b.startTime));
  upcoming.sort((a, b) => new Date(a.startTime) - new Date(b.startTime));
  completed.sort((a, b) => new Date(a.startTime) - new Date(b.startTime));
  return [...ongoing, ...upcoming, ...completed];
});

const countByStatus = computed(() => {
  const list = filteredMeetings.value;
  return {
    ongoing: list.filter((m) => m.status === 'ongoing').length,
    upcoming: list.filter((m) => m.status === 'upcoming').length,
    completed: list.filter((m) => m.status === 'completed').length,
  };
});

const getMeetingProgress = (meeting) => {
  const now = new Date();
  const start = new Date(meeting.startTime);
  const end = new Date(meeting.endTime);
  const total = end - start;
  const elapsed = now - start;
  return Math.min(100, Math.max(0, (elapsed / total) * 100));
};

const toggleRoomFilter = () => { showRoomFilter.value = !showRoomFilter.value; };

const toggleFullscreen = () => {
  if (!isFullscreen.value) {
    const el = containerRef.value;
    if (el.requestFullscreen) el.requestFullscreen();
    else if (el.webkitRequestFullscreen) el.webkitRequestFullscreen();
    else if (el.msRequestFullscreen) el.msRequestFullscreen();
  } else {
    if (document.exitFullscreen) document.exitFullscreen();
    else if (document.webkitExitFullscreen) document.webkitExitFullscreen();
    else if (document.msExitFullscreen) document.msExitFullscreen();
  }
};

const handleFullscreenChange = () => {
  isFullscreen.value = !!(document.fullscreenElement || document.webkitFullscreenElement || document.msFullscreenElement);
};

const selectAllRooms = () => {
  selectedRoomIds.value = availableRooms.value.map((r) => r.id);
  saveRoomPreferences();
};

const clearRoomSelection = () => {
  selectedRoomIds.value = [];
  saveRoomPreferences();
};

const saveRoomPreferences = () => {
  localStorage.setItem('selectedRoomIds', JSON.stringify(selectedRoomIds.value));
};

const loadRoomPreferences = () => {
  const saved = localStorage.getItem('selectedRoomIds');
  if (saved) {
    try {
      const ids = JSON.parse(saved);
      selectedRoomIds.value = ids.filter((id) => availableRooms.value.some((r) => r.id === id));
    } catch { selectedRoomIds.value = []; }
  } else {
    selectedRoomIds.value = [];
  }
};

const getRoomName = (roomId) => {
  const room = availableRooms.value.find((r) => r.id === roomId);
  return room ? room.name : roomId;
};

/**
 * /rooms ส่งทั้งรายชื่อห้องและตารางของแต่ละห้องมาในครั้งเดียว
 * จึงเรียกแค่ endpoint เดียวต่อรอบ refresh
 */
const fetchMeetings = async () => {
  if (meetings.value.length === 0) loading.value = true;
  error.value = null;
  try {
    const { data } = await apiFetch('/rooms');

    availableRooms.value = data.map((room) => ({ id: room.id, name: room.name }));
    if (!roomsLoaded.value) {
      roomsLoaded.value = true;
      loadRoomPreferences();
      if (selectedRoomIds.value.length === 0) selectAllRooms();
    }

    meetings.value = data.flatMap((room) => room.meetings.map((m) => ({
      id: m.id,
      title: m.title,
      roomId: m.room_id,
      roomName: m.room_name,
      startTime: m.start_time,
      endTime: m.end_time,
      organizer: m.organizer,
      orgName: m.org_name,
      status: m.status,
    })));
    meetings.value.sort((a, b) => new Date(a.startTime) - new Date(b.startTime));
    lastUpdated.value = timeDisplay.value;
  } catch (err) {
    console.error('Error fetching meetings:', err);
    // ระหว่าง refresh ถ้าดึงไม่สำเร็จ ให้คงข้อมูลเดิมบนจอไว้ แทนที่จะขึ้นจอว่าง
    if (meetings.value.length === 0) {
      error.value = err.message || 'ไม่สามารถโหลดข้อมูลการประชุมได้';
    }
  } finally {
    loading.value = false;
  }
};

const getMeetingStatus = (startTime, endTime) => {
  const now = new Date();
  const start = new Date(startTime);
  const end = new Date(endTime);
  if (now < start) return 'upcoming';
  if (now >= start && now <= end) return 'ongoing';
  return 'completed';
};

const updateClock = () => {
  const now = new Date();
  dateDisplay.value = now.toLocaleDateString('th-TH', { year: 'numeric', month: 'long', day: 'numeric', weekday: 'long' });
  timeDisplay.value = now.toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit', second: '2-digit' });
  if (now.getSeconds() === 0) updateMeetingStatuses();
};

let timeInterval;

const updateMeetingStatuses = () => {
  meetings.value.forEach((m) => { m.status = getMeetingStatus(m.startTime, m.endTime); });
};

const formatTime = (isoTime) => {
  return new Date(isoTime).toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' });
};

const getStatusText = (status) => {
  switch (status) {
    case 'ongoing': return 'กำลังประชุม';
    case 'upcoming': return 'รอดำเนินการ';
    case 'completed': return 'เสร็จสิ้น';
    default: return '';
  }
};

let refreshInterval;

onMounted(() => {
  document.addEventListener('fullscreenchange', handleFullscreenChange);
  document.addEventListener('webkitfullscreenchange', handleFullscreenChange);
  document.addEventListener('MSFullscreenChange', handleFullscreenChange);
  updateClock();
  timeInterval = setInterval(updateClock, 1000);
  fetchMeetings();
  refreshInterval = setInterval(fetchMeetings, refreshIntervalMs);
  setTimeout(() => { startAutoScroll(); }, 500);
});

onUnmounted(() => {
  document.removeEventListener('fullscreenchange', handleFullscreenChange);
  document.removeEventListener('webkitfullscreenchange', handleFullscreenChange);
  document.removeEventListener('MSFullscreenChange', handleFullscreenChange);
  clearInterval(timeInterval);
  clearInterval(refreshInterval);
  stopAutoScroll();
});

watch(selectedRoomIds, () => { fetchMeetings(); });
</script>

<style>
/* ===== Design Tokens — Navy (ระบบจองห้องประชุม / กรมการขนส่งทางบก) ===== */
:root {
  --bg-deep: #0a1530;
  --bg-surface: rgba(255, 255, 255, 0.07);
  --bg-card: rgba(255, 255, 255, 0.06);
  --bg-glass: rgba(255, 255, 255, 0.04);
  --border-subtle: rgba(255, 255, 255, 0.12);
  --border-glow: rgba(93, 137, 217, 0.45);
  --accent: #5d89d9;
  --accent-bright: #8fb0ea;
  --accent-dim: #1d3a82;
  --dlt-red: #ed1c24;
  --green: #34d399;
  --green-dim: rgba(52, 211, 153, 0.14);
  --yellow: #fbbf24;
  --yellow-dim: rgba(251, 191, 36, 0.14);
  --muted: rgba(255, 255, 255, 0.35);
  --text: #f8fafc;
  --text-secondary: rgba(255, 255, 255, 0.62);
  --radius: 18px;
  --radius-sm: 12px;
}

html, body, #app {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  background: var(--bg-deep);
}

/* ===== Root container ===== */
.display-root {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  background: linear-gradient(140deg, #0a1530 0%, #12234d 55%, #182f66 100%);
  color: var(--text);
  font-family: 'Noto Sans Thai', 'Noto Sans', sans-serif;
}

/* ===== Animated background ===== */
.bg-layer {
  position: absolute;
  inset: 0;
  z-index: 0;
  overflow: hidden;
  pointer-events: none;
}

.bg-orb {
  position: absolute;
  border-radius: 50%;
  filter: blur(120px);
  opacity: 0.40;
  animation: float 20s ease-in-out infinite;
}

.bg-orb-1 {
  width: 600px;
  height: 600px;
  background: radial-gradient(circle, #1d3a82 0%, transparent 70%);
  top: -15%;
  left: -10%;
  animation-delay: 0s;
}

.bg-orb-2 {
  width: 500px;
  height: 500px;
  background: radial-gradient(circle, #3563c4 0%, transparent 70%);
  bottom: -20%;
  right: -8%;
  animation-delay: -7s;
}

.bg-orb-3 {
  width: 350px;
  height: 350px;
  background: radial-gradient(circle, #ed1c24 0%, transparent 70%);
  top: 40%;
  left: 50%;
  animation-delay: -14s;
  opacity: 0.18;
}

@keyframes float {
  0%, 100% { transform: translate(0, 0) scale(1); }
  33% { transform: translate(30px, -20px) scale(1.05); }
  66% { transform: translate(-20px, 15px) scale(0.95); }
}

.bg-grid {
  position: absolute;
  inset: 0;
  background-image: radial-gradient(rgba(255, 255, 255, 0.10) 1px, transparent 1px);
  background-size: 26px 26px;
  mask-image: linear-gradient(to bottom, rgba(0, 0, 0, 0.9), rgba(0, 0, 0, 0.25));
}

/* ===== Top bar ===== */
.top-bar {
  position: relative;
  z-index: 10;
  margin-top: 4px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 32px;
  background: var(--bg-surface);
  backdrop-filter: blur(24px);
  -webkit-backdrop-filter: blur(24px);
  border-bottom: 1px solid var(--border-subtle);
  flex-shrink: 0;
}

.top-bar-left {
  display: flex;
  align-items: center;
  gap: 16px;
}

.dlt-stripe {
  position: absolute;
  inset: 0 0 auto 0;
  height: 4px;
  z-index: 20;
  background: linear-gradient(90deg, #1d3a82 0%, #1d3a82 70%, var(--dlt-red) 70%, var(--dlt-red) 100%);
}

.logo-mark {
  width: 60px;
  height: 60px;
  border-radius: 16px;
  background: linear-gradient(135deg, #1d3a82, #0a1530);
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 6px 20px rgba(10, 21, 48, 0.55);
  border: 1px solid rgba(255, 255, 255, 0.18);
}

.logo-mark svg {
  width: 40px;
  height: 40px;
  color: white;
}

.brand-org {
  font-size: 0.78em;
  font-weight: 700;
  letter-spacing: 0.22em;
  text-transform: uppercase;
  color: var(--dlt-red);
  margin: 0 0 2px;
}

.brand-title {
  font-size: 2.2em;
  font-weight: 800;
  margin: 0;
  color: var(--text);
  line-height: 1.15;
}

.brand-date {
  font-size: 1.1em;
  color: var(--text-secondary);
  margin: 0;
}

/* Clock */
.clock-display {
  display: flex;
  align-items: center;
  gap: 12px;
}

.clock-time {
  font-size: 3.5em;
  font-weight: 700;
  font-variant-numeric: tabular-nums;
  letter-spacing: 0.04em;
  background: linear-gradient(135deg, var(--accent-dim), #3563c4);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.clock-pulse {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: var(--green);
  box-shadow: 0 0 8px rgba(5, 150, 105, 0.4);
  animation: pulse 2s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; transform: scale(1); }
  50% { opacity: 0.4; transform: scale(0.8); }
}

/* Top bar buttons */
.top-bar-right {
  display: flex;
  gap: 8px;
}

.icon-btn {
  width: 46px;
  height: 46px;
  border-radius: 12px;
  border: 1px solid var(--border-subtle);
  background: var(--bg-glass);
  color: var(--accent-bright);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.25s ease;
  padding: 0;
}

.icon-btn svg {
  width: 22px;
  height: 22px;
}

.icon-btn:hover {
  background: rgba(93, 137, 217, 0.10);
  border-color: var(--accent);
  box-shadow: 0 4px 12px rgba(93, 137, 217, 0.15);
  transform: translateY(-1px);
}

/* ===== Filter panel ===== */
.filter-panel {
  position: relative;
  z-index: 10;
  background: var(--bg-surface);
  backdrop-filter: blur(24px);
  -webkit-backdrop-filter: blur(24px);
  border-bottom: 1px solid var(--border-subtle);
}

.filter-inner {
  padding: 20px 32px;
}

.filter-heading {
  font-size: 1.1em;
  font-weight: 600;
  color: var(--accent-bright);
  margin-bottom: 14px;
}

.filter-chips {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 16px;
}

.chip {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 18px;
  border-radius: 999px;
  border: 1px solid var(--border-subtle);
  background: var(--bg-glass);
  color: var(--text-secondary);
  cursor: pointer;
  transition: all 0.25s ease;
  font-size: 1em;
  font-family: inherit;
}

.chip input { display: none; }

.chip-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: var(--muted);
  transition: all 0.25s;
}

.chip.active {
  background: rgba(93, 137, 217, 0.10);
  border-color: var(--accent);
  color: var(--accent-dim);
}

.chip.active .chip-dot {
  background: var(--accent);
  box-shadow: 0 0 8px var(--accent);
}

.chip:hover {
  border-color: var(--accent);
  transform: translateY(-1px);
}

.filter-actions {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}

.action-btn {
  padding: 10px 22px;
  border-radius: 10px;
  border: none;
  font-size: 0.95em;
  font-weight: 600;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.25s;
}

.action-btn.primary {
  background: linear-gradient(135deg, #1d3a82, #3563c4);
  color: white;
  box-shadow: 0 4px 14px rgba(10, 21, 48, 0.45);
}

.action-btn.primary:hover {
  box-shadow: 0 6px 20px rgba(93, 137, 217, 0.30);
  transform: translateY(-1px);
}

.action-btn.secondary {
  background: var(--bg-glass);
  color: var(--accent);
  border: 1px solid var(--border-subtle);
}

.action-btn.secondary:hover {
  background: rgba(93, 137, 217, 0.08);
  border-color: var(--accent);
}

.action-btn.danger {
  background: transparent;
  color: #f87171;
  border: 1px solid rgba(248, 113, 113, 0.3);
}

.action-btn.danger:hover {
  background: rgba(248, 113, 113, 0.1);
}

/* Transition */
.slide-down-enter-active, .slide-down-leave-active {
  transition: all 0.3s ease;
}
.slide-down-enter-from, .slide-down-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

/* ===== State boxes (loading / error / empty) ===== */
.state-box {
  position: relative;
  z-index: 5;
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 16px;
  padding: 40px;
}

.state-icon {
  width: 64px;
  height: 64px;
  color: var(--accent);
  opacity: 0.6;
}

.state-box.error .state-icon { color: #f87171; }

.state-title {
  font-size: 1.6em;
  font-weight: 700;
  color: var(--text);
}

.state-text {
  font-size: 1.1em;
  color: var(--text-secondary);
}

.spinner {
  width: 48px;
  height: 48px;
  border: 3px solid var(--border-subtle);
  border-top-color: var(--accent);
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.mt-4 { margin-top: 16px; }

/* ===== Meeting scroll area ===== */
.meeting-scroll {
  position: relative;
  z-index: 5;
  flex: 1;
  overflow-y: auto;
  padding: 16px 32px 8px;
  scrollbar-width: none;
  -ms-overflow-style: none;
}

.meeting-scroll::-webkit-scrollbar { display: none; }

/* ===== Summary bar ===== */
.summary-bar {
  display: flex;
  gap: 16px;
  margin-bottom: 16px;
  flex-wrap: wrap;
}

.summary-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 20px;
  border-radius: var(--radius-sm);
  background: var(--bg-glass);
  border: 1px solid var(--border-subtle);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
}

.summary-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
}

.summary-item.ongoing .summary-dot {
  background: var(--green);
  box-shadow: 0 0 8px rgba(5, 150, 105, 0.4);
}

.summary-dot.pulse {
  animation: pulse 2s ease-in-out infinite;
}

.summary-item.upcoming .summary-dot {
  background: var(--yellow);
  box-shadow: 0 0 6px rgba(217, 119, 6, 0.3);
}

.summary-item.completed .summary-dot {
  background: var(--muted);
}

.summary-label {
  font-size: 1.15em;
  color: var(--text-secondary);
}

.summary-count {
  font-size: 1.5em;
  font-weight: 700;
  color: var(--text);
}

/* ===== Meeting cards ===== */
.meeting-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.meeting-card {
  display: flex;
  border-radius: var(--radius);
  background: var(--bg-card);
  border: 1px solid var(--border-subtle);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  overflow: hidden;
  transition: all 0.3s ease;
}

.meeting-card:hover {
  border-color: var(--border-glow);
  transform: translateX(4px);
  box-shadow: 0 4px 20px rgba(93, 137, 217, 0.10);
}

/* Status-specific card styles */
.meeting-card.ongoing {
  background: linear-gradient(135deg, rgba(5, 150, 105, 0.06) 0%, var(--bg-card) 100%);
  border-color: rgba(5, 150, 105, 0.25);
  box-shadow: 0 2px 16px rgba(5, 150, 105, 0.08);
}

.meeting-card.upcoming {
  background: linear-gradient(135deg, rgba(93, 137, 217, 0.05) 0%, var(--bg-card) 100%);
  border-color: rgba(93, 137, 217, 0.18);
}

.meeting-card.completed {
  opacity: 0.5;
}

.meeting-card.completed:hover {
  opacity: 0.7;
}

/* Accent bar */
.card-accent {
  width: 5px;
  flex-shrink: 0;
}

.meeting-card.ongoing .card-accent {
  background: linear-gradient(180deg, var(--green), #10b981);
  box-shadow: 2px 0 10px rgba(5, 150, 105, 0.2);
}

.meeting-card.upcoming .card-accent {
  background: linear-gradient(180deg, var(--accent), var(--accent-dim));
}

.meeting-card.completed .card-accent {
  background: var(--muted);
}

/* Card body */
.card-body {
  flex: 1;
  display: flex;
  align-items: center;
  padding: 22px 28px;
  gap: 32px;
}

/* Time column */
.card-time {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-width: 120px;
  flex-shrink: 0;
}

.time-start, .time-end {
  font-size: 1.8em;
  font-weight: 700;
  font-variant-numeric: tabular-nums;
  color: var(--text);
  line-height: 1.3;
}

.time-arrow {
  width: 16px;
  height: 16px;
  color: var(--muted);
  margin: 2px 0;
}

.time-progress {
  width: 100%;
  height: 3px;
  background: rgba(255, 255, 255, 0.14);
  border-radius: 99px;
  margin-top: 6px;
  overflow: hidden;
}

.time-progress-bar {
  height: 100%;
  background: linear-gradient(90deg, var(--green), #34d399);
  border-radius: 99px;
  transition: width 1s ease;
  box-shadow: 0 1px 6px rgba(5, 150, 105, 0.25);
}

/* Info column */
.card-info {
  flex: 1;
  min-width: 0;
}

.card-title {
  font-size: 1.9em;
  font-weight: 700;
  margin: 0 0 8px;
  color: var(--text);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.meeting-card.ongoing .card-title {
  color: #047857;
}

.card-meta {
  display: flex;
  gap: 24px;
  flex-wrap: wrap;
}

.meta-item {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 1.3em;
  color: var(--text-secondary);
}

.meta-item svg {
  width: 18px;
  height: 18px;
  flex-shrink: 0;
  opacity: 0.6;
}

/* Status badge */
.card-status {
  flex-shrink: 0;
}

.badge {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 10px 22px;
  border-radius: 999px;
  font-size: 1.15em;
  font-weight: 700;
  white-space: nowrap;
  letter-spacing: 0.02em;
}

.badge-ongoing {
  background: rgba(5, 150, 105, 0.10);
  color: #047857;
  border: 1px solid rgba(5, 150, 105, 0.25);
  box-shadow: 0 2px 8px rgba(5, 150, 105, 0.10);
}

.badge-upcoming {
  background: var(--yellow-dim);
  color: var(--yellow);
  border: 1px solid rgba(251, 191, 36, 0.35);
}

.badge-completed {
  background: rgba(255, 255, 255, 0.06);
  color: rgba(255, 255, 255, 0.55);
  border: 1px solid rgba(255, 255, 255, 0.12);
}

.badge-pulse {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: var(--green);
  box-shadow: 0 0 6px rgba(5, 150, 105, 0.4);
  animation: pulse 2s ease-in-out infinite;
}

/* ===== Footer ===== */
.bottom-bar {
  position: relative;
  z-index: 10;
  text-align: center;
  padding: 10px 32px;
  font-size: 0.9em;
  color: var(--muted);
  border-top: 1px solid var(--border-subtle);
  background: var(--bg-surface);
  backdrop-filter: blur(24px);
  -webkit-backdrop-filter: blur(24px);
  flex-shrink: 0;
}

.footer-sep {
  margin: 0 8px;
  opacity: 0.3;
}

/* ===== Responsive: TV (1920px+) ===== */
@media (min-width: 1920px) {
  .brand-title { font-size: 2.8em; }
  .brand-date { font-size: 1.3em; }
  .clock-time { font-size: 4.5em; }
  .logo-mark { width: 64px; height: 64px; }
  .logo-mark svg { width: 34px; height: 34px; }

  .card-body { padding: 26px 36px; gap: 40px; }
  .time-start, .time-end { font-size: 2.2em; }
  .card-title { font-size: 2.4em; }
  .meta-item { font-size: 1.5em; }
  .meta-item svg { width: 22px; height: 22px; }
  .badge { font-size: 1.35em; padding: 12px 28px; }
  .badge-pulse { width: 10px; height: 10px; }

  .summary-label { font-size: 1.3em; }
  .summary-count { font-size: 1.8em; }
  .summary-item { padding: 14px 24px; gap: 12px; }

  .meeting-scroll { padding: 24px 56px 14px; }
  .meeting-list { gap: 14px; }
  .top-bar { padding: 22px 56px; }
  .card-accent { width: 6px; }
  .bottom-bar { font-size: 1.05em; }
}

/* ===== Responsive: Large TV (2560px+) ===== */
@media (min-width: 2560px) {
  .brand-title { font-size: 3.5em; }
  .clock-time { font-size: 5.5em; }
  .card-title { font-size: 3em; }
  .time-start, .time-end { font-size: 2.6em; }
  .meta-item { font-size: 1.8em; }
  .meta-item svg { width: 26px; height: 26px; }
  .badge { font-size: 1.7em; padding: 14px 32px; }
  .card-body { padding: 30px 44px; gap: 48px; }
  .meeting-list { gap: 18px; }
  .summary-label { font-size: 1.5em; }
  .summary-count { font-size: 2.2em; }
}

/* ===== Responsive: Tablet landscape (1024-1366px) ===== */
@media (max-width: 1366px) {
  .top-bar { padding: 14px 24px; }
  .brand-title { font-size: 1.5em; }
  .clock-time { font-size: 2.4em; }
  .meeting-scroll { padding: 14px 24px 8px; }
  .card-body { padding: 16px 20px; gap: 20px; }
  .card-title { font-size: 1.4em; }
  .time-start, .time-end { font-size: 1.3em; }
}

/* ===== Responsive: Tablet portrait (768-1024px) ===== */
@media (max-width: 1024px) {
  .top-bar {
    flex-wrap: wrap;
    gap: 12px;
    padding: 12px 20px;
  }

  .top-bar-left { order: 1; }
  .clock-display { order: 3; width: 100%; justify-content: center; padding-top: 4px; }
  .top-bar-right { order: 2; }

  .brand-title { font-size: 1.3em; }
  .clock-time { font-size: 2em; }

  .meeting-scroll { padding: 12px 16px 8px; }
  .card-body { flex-wrap: wrap; padding: 14px 16px; gap: 12px; }
  .card-time { flex-direction: row; gap: 8px; min-width: unset; }
  .time-arrow { transform: rotate(-90deg); width: 14px; height: 14px; margin: 0; }
  .card-info { width: 100%; order: 3; }
  .card-status { order: 2; }

  .summary-bar { gap: 10px; }
  .summary-item { padding: 8px 14px; }
}

/* ===== Responsive: Small tablet / phone (< 768px) ===== */
@media (max-width: 768px) {
  .top-bar { padding: 10px 14px; }
  .logo-mark { width: 40px; height: 40px; border-radius: 10px; }
  .logo-mark svg { width: 22px; height: 22px; }
  .brand-title { font-size: 1.15em; }
  .brand-date { font-size: 0.85em; }
  .clock-time { font-size: 1.7em; }
  .icon-btn { width: 40px; height: 40px; }

  .meeting-scroll { padding: 10px 12px 6px; }
  .meeting-list { gap: 8px; }

  .card-body { padding: 12px 14px; gap: 10px; }
  .card-title { font-size: 1.15em; }
  .time-start, .time-end { font-size: 1.1em; }
  .meta-item { font-size: 0.9em; }
  .badge { font-size: 0.85em; padding: 6px 14px; }

  .filter-inner { padding: 14px 16px; }
  .chip { padding: 8px 14px; font-size: 0.9em; }
}

/* ===== Fullscreen overrides ===== */
:fullscreen .display-root,
:-webkit-full-screen .display-root {
  border-radius: 0;
}
</style>
