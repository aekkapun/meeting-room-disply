<template>
  <div class="container" ref="containerRef">
    <!-- ส่วนหัว -->
    <div class="header">
      <h1>การประชุมประจำวัน</h1>
      <div class="header-buttons">
        <button @click="toggleRoomFilter" class="filter-button">
          <svg xmlns="http://www.w3.org/2000/svg" class="filter-icon" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd" d="M3 3a1 1 0 011-1h12a1 1 0 011 1v3a1 1 0 01-.293.707L12 11.414V15a1 1 0 01-.293.707l-2 2A1 1 0 018 17v-5.586L3.293 6.707A1 1 0 013 6V3z" clip-rule="evenodd" />
          </svg>
          ตัวกรองห้องประชุม
        </button>
        <button @click="toggleFullscreen" class="fullscreen-button">
          <svg v-if="!isFullscreen" xmlns="http://www.w3.org/2000/svg" class="fullscreen-icon" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd" d="M3 4a1 1 0 011-1h4a1 1 0 010 2H6.414l2.293 2.293a1 1 0 01-1.414 1.414L5 6.414V8a1 1 0 01-2 0V4zm9 1a1 1 0 010-2h4a1 1 0 011 1v4a1 1 0 01-2 0V6.414l-2.293 2.293a1 1 0 11-1.414-1.414L13.586 5H12zm-9 7a1 1 0 012 0v1.586l2.293-2.293a1 1 0 011.414 1.414L6.414 15H8a1 1 0 010 2H4a1 1 0 01-1-1v-4zm13-1a1 1 0 011 1v4a1 1 0 01-1 1h-4a1 1 0 010-2h1.586l-2.293-2.293a1 1 0 011.414-1.414L15 13.586V12a1 1 0 011-1z" clip-rule="evenodd" />
          </svg>
          <svg v-else xmlns="http://www.w3.org/2000/svg" class="fullscreen-icon" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd" d="M5 4a1 1 0 00-1 1v4a1 1 0 01-2 0V5a3 3 0 013-3h4a1 1 0 010 2H5zm10 8a1 1 0 01-1 1h-4a1 1 0 010-2h4a1 1 0 001-1V7a1 1 0 112 0v4a3 3 0 01-3 3z" clip-rule="evenodd" />
          </svg>
          {{ isFullscreen ? 'ออกจากเต็มจอ' : 'แสดงเต็มจอ' }}
        </button>
      </div>
    </div>

    <!-- ส่วนตัวกรองห้องประชุม -->
    <div :class="['room-filter-panel', {'open': showRoomFilter}]">
      <div class="filter-container">
        <h3 class="filter-title">เลือกห้องประชุมที่ต้องการแสดง</h3>
        <div class="room-options">
          <div v-for="room in availableRooms" :key="room.id" class="room-option">
            <label class="room-checkbox">
              <input type="checkbox" :value="room.id" v-model="selectedRoomIds" @change="saveRoomPreferences">
              <span class="room-label">{{ room.name }}</span>
            </label>
          </div>
        </div>
      </div>
      <div class="filter-buttons">
        <button @click="selectAllRooms" class="btn btn-select-all">เลือกทั้งหมด</button>
        <button @click="clearRoomSelection" class="btn btn-clear">ยกเลิกทั้งหมด</button>
        <button @click="toggleRoomFilter" class="btn btn-apply">บันทึกและปิด</button>
      </div>
    </div>

    <!-- แสดงเวลาปัจจุบัน -->
    <div class="current-time">
      <span>{{ dateDisplay }}</span> | <span>{{ timeDisplay }}</span>
    </div>

    <!-- ส่วนแสดงการโหลด -->
    <div v-if="loading" class="loading-container">
      <div class="loader"></div>
      <p class="loading-text">กำลังโหลดข้อมูล...</p>
    </div>

    <!-- ส่วนแสดงข้อผิดพลาด -->
    <div v-else-if="error" class="error-container">
      <p class="error-title">เกิดข้อผิดพลาดในการโหลดข้อมูล</p>
      <p>{{ error }}</p>
      <button @click="fetchMeetings" class="btn btn-retry">ลองใหม่อีกครั้ง</button>
    </div>

    <!-- ข้อความเมื่อไม่มีการเลือกห้อง -->
    <div v-else-if="selectedRoomIds.length === 0" class="empty-selection">
      <p class="empty-title">ไม่ได้เลือกห้องประชุม</p>
      <p>กรุณาเลือกห้องประชุมที่ต้องการแสดงข้อมูล</p>
      <button @click="toggleRoomFilter" class="btn btn-select-room">เลือกห้องประชุม</button>
    </div>

    <!-- ข้อความเมื่อไม่พบการประชุมในห้องที่เลือก -->
    <div v-else-if="filteredMeetings.length === 0 && selectedRoomIds.length > 0" class="no-meetings">
      <p class="no-meetings-title">ไม่พบการประชุมในห้องที่เลือก</p>
      <p>กรุณาเลือกห้องประชุมอื่น หรือลองตรวจสอบในภายหลัง</p>
    </div>

    <!-- ตารางการประชุม -->
    <table v-else class="meeting-table">
      <thead>
      <tr>
        <th width="10%">เวลา</th>
        <th width="25%">หัวข้อการประชุม</th>
        <th width="20%">ห้องประชุม</th>
        <th width="20%">ผู้รับผิดชอบ</th>
        <th width="20%">สถานะ</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="(meeting, index) in filteredMeetings"
          :key="meeting.id"
          :class="meetingRowClass(meeting, index)">
        <td>
          {{ formatTime(meeting.startTime) }}
          <svg xmlns="http://www.w3.org/2000/svg" class="time-separator-icon" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd" d="M12.293 5.293a1 1 0 011.414 0l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414-1.414L14.586 11H3a1 1 0 110-2h11.586l-2.293-2.293a1 1 0 010-1.414z" clip-rule="evenodd" />
          </svg>
          {{ formatTime(meeting.endTime) }}
        </td>
        <td>{{ meeting.title }}</td>
        <td>{{ getRoomName(meeting.roomId) }}</td>
        <td>{{ meeting.organizer }}</td>
        <td><span :class="['status', statusClass(meeting.status)]">{{ getStatusText(meeting.status) }}</span></td>
      </tr>
      <tr v-if="filteredMeetings.length === 0">
        <td colspan="5" class="empty-message">ไม่มีการประชุมในวันนี้</td>
      </tr>
      </tbody>
    </table>

    <div class="footer">
      <p>อัพเดทล่าสุด: <span>{{ lastUpdated }}</span></p>
      <button @click="fetchMeetings" class="btn btn-refresh">รีเฟรช</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, onUnmounted } from 'vue';

// สร้าง ref สำหรับ container เพื่อใช้ในการทำ fullscreen
const containerRef = ref(null);
const isFullscreen = ref(false);

// ข้อมูลเวลา
const dateDisplay = ref('');
const timeDisplay = ref('');
const lastUpdated = ref('');

// ข้อมูลการประชุม
const meetings = ref([]);
const loading = ref(true);
const error = ref(null);
//const apiUrl = ref('http://localhost:3000/api/meetings'); // เปลี่ยนเป็น URL API ของเรา
const apiUrl = ref('http://localhost:3000/api'); // เปลี่ยนเป็น URL API หลัก

const fetchRooms = async () => {
  try {
    const response = await fetch(`${apiUrl.value}/rooms`);

    console.log(response)

    if (!response.ok) throw new Error('การเชื่อมต่อ API ล้มเหลว');

    const data = await response.json();
    availableRooms.value = data;

    console.log(availableRooms)

    // หลังจากโหลดห้องเสร็จ จึงโหลดการเลือกห้องจาก localStorage
    loadRoomPreferences();

    // ถ้าไม่มีห้องที่เลือกไว้ ให้เลือกทุกห้องเป็นค่าเริ่มต้น
    if (selectedRoomIds.value.length === 0) {
      selectAllRooms();
    }
  } catch (err) {
    // กรณีเกิดข้อผิดพลาด ใช้ค่าเริ่มต้น
    availableRooms.value = [
      { id: 'A', name: 'ห้องประชุม A' },
      // ...
    ];
    // ...
  }
};
// ข้อมูลตัวกรองห้องประชุม
const showRoomFilter = ref(false);
const availableRooms = ref([
  { id: 'A', name: 'ห้องประชุม A' },
  { id: 'B', name: 'ห้องประชุม B' },
  { id: 'C', name: 'ห้องประชุม C' },
  { id: 'D', name: 'ห้องประชุม D' },
  { id: 'E', name: 'ห้องประชุม E' },
]);
const selectedRoomIds = ref([]);

// กรองการประชุมตามห้องที่เลือก
// กรองและจัดเรียงการประชุมตามห้องที่เลือกและสถานะ
const filteredMeetings = computed(() => {
  if (selectedRoomIds.value.length === 0) {
    return [];
  }

  // กรองห้องที่เลือก
  const filtered = meetings.value.filter(meeting =>
      selectedRoomIds.value.includes(meeting.roomId)
  );

  // แยกการประชุมตามสถานะ
  const ongoing = filtered.filter(meeting => meeting.status === 'ongoing');
  const upcoming = filtered.filter(meeting => meeting.status === 'upcoming');
  const completed = filtered.filter(meeting => meeting.status === 'completed');

  // เรียงลำดับตามเวลาเริ่มต้น
  ongoing.sort((a, b) => new Date(a.startTime) - new Date(b.startTime));
  upcoming.sort((a, b) => new Date(a.startTime) - new Date(b.startTime));
  completed.sort((a, b) => new Date(a.startTime) - new Date(b.startTime));

  // รวมกลับเป็นรายการเดียว: กำลังประชุม -> รอดำเนินการ -> เสร็จสิ้น
  return [...ongoing, ...upcoming, ...completed];
});

// เปิด/ปิดแผงตัวกรองห้อง
const toggleRoomFilter = () => {
  showRoomFilter.value = !showRoomFilter.value;
};

// เปิด/ปิดการแสดงผลแบบเต็มจอ
const toggleFullscreen = () => {
  if (!isFullscreen.value) {
    // เปิดเต็มจอ
    const element = containerRef.value;
    if (element.requestFullscreen) {
      element.requestFullscreen();
    } else if (element.mozRequestFullScreen) { // Firefox
      element.mozRequestFullScreen();
    } else if (element.webkitRequestFullscreen) { // Chrome, Safari, Opera
      element.webkitRequestFullscreen();
    } else if (element.msRequestFullscreen) { // IE/Edge
      element.msRequestFullscreen();
    }
  } else {
    // ปิดเต็มจอ
    if (document.exitFullscreen) {
      document.exitFullscreen();
    } else if (document.mozCancelFullScreen) { // Firefox
      document.mozCancelFullScreen();
    } else if (document.webkitExitFullscreen) { // Chrome, Safari, Opera
      document.webkitExitFullscreen();
    } else if (document.msExitFullscreen) { // IE/Edge
      document.msExitFullscreen();
    }
  }
};

// ตรวจจับการเปลี่ยนแปลงสถานะเต็มจอ
const handleFullscreenChange = () => {
  isFullscreen.value = !!(
      document.fullscreenElement ||
      document.mozFullScreenElement ||
      document.webkitFullscreenElement ||
      document.msFullscreenElement
  );
};

// เลือกห้องทั้งหมด
const selectAllRooms = () => {
  selectedRoomIds.value = availableRooms.value.map(room => room.id);
  saveRoomPreferences();
};

// ยกเลิกการเลือกห้องทั้งหมด
const clearRoomSelection = () => {
  selectedRoomIds.value = [];
  saveRoomPreferences();
};

// บันทึกการเลือกห้องไว้ใน localStorage
const saveRoomPreferences = () => {
  localStorage.setItem('selectedRoomIds', JSON.stringify(selectedRoomIds.value));
};

// โหลดการเลือกห้องจาก localStorage
const loadRoomPreferences = () => {
  const savedRooms = localStorage.getItem('selectedRoomIds');
  if (savedRooms) {
    try {
      const savedRoomIds = JSON.parse(savedRooms);
      // ตรวจสอบว่า room ที่บันทึกไว้ยังมีอยู่ในระบบหรือไม่
      selectedRoomIds.value = savedRoomIds.filter(id =>
          availableRooms.value.some(room => room.id === id)
      );
    } catch (e) {
      console.error('Error parsing saved room preferences', e);
      selectedRoomIds.value = [];
    }
  } else {
    selectedRoomIds.value = [];
  }
};

// หาชื่อห้องจาก roomId
const getRoomName = (roomId) => {
  const room = availableRooms.value.find(r => r.id === roomId);
  return room ? room.name : roomId;
};

// ดึงข้อมูลการประชุมจาก API
const fetchMeetings = async () => {
  loading.value = true;
  error.value = null;

  try {
    // ในการใช้งานจริง รับพารามิเตอร์ห้องที่เลือกเพื่อส่งไป API
    const roomParams = selectedRoomIds.value.join(',');
   // const response = await fetch(`${apiUrl.value}?rooms=${roomParams}`);
    const response = await fetch(`${apiUrl.value}/meetings?rooms=${roomParams}`);
    if (!response.ok) throw new Error('การเชื่อมต่อ API ล้มเหลว');

    const data = await response.json();
    meetings.value = data;

    // เรียงลำดับตามเวลาเริ่มต้น (ถ้า API ไม่เรียงให้)
    meetings.value.sort((a, b) => new Date(a.startTime) - new Date(b.startTime));

    lastUpdated.value = timeDisplay.value;
  } catch (err) {
    console.error('Error fetching meetings:', err);
    error.value = 'ไม่สามารถโหลดข้อมูลการประชุมได้ กรุณาลองใหม่อีกครั้ง';

    // สำหรับการทดสอบ ถ้ามีข้อผิดพลาดให้ใช้ข้อมูลจำลอง
    if (process.env.NODE_ENV === 'development') {
      const now = new Date();
      const todayDate = now.toISOString().split('T')[0];

      meetings.value = [
        {
          id: 1,
          title: 'ประชุมทีมการตลาด',
          roomId: 'A',
          startTime: `${todayDate}T09:00:00`,
          endTime: `${todayDate}T10:00:00`,
          organizer: 'คุณวิชัย สุขสมบัติ',
          status: getMeetingStatus(`${todayDate}T09:00:00`, `${todayDate}T10:00:00`)
        },
        {
          id: 2,
          title: 'การวางแผนโครงการใหม่',
          roomId: 'B',
          startTime: `${todayDate}T10:30:00`,
          endTime: `${todayDate}T11:30:00`,
          organizer: 'คุณสมหญิง ใจดี',
          status: getMeetingStatus(`${todayDate}T10:30:00`, `${todayDate}T11:30:00`)
        },
        {
          id: 3,
          title: 'ประชุมฝ่ายขาย',
          roomId: 'C',
          startTime: `${todayDate}T13:00:00`,
          endTime: `${todayDate}T14:30:00`,
          organizer: 'คุณสมชาย มั่นคง',
          status: getMeetingStatus(`${todayDate}T13:00:00`, `${todayDate}T14:30:00`)
        },
        {
          id: 4,
          title: 'ประชุมติดตามความคืบหน้า',
          roomId: 'A',
          startTime: `${todayDate}T15:00:00`,
          endTime: `${todayDate}T16:00:00`,
          organizer: 'คุณนภา สมบูรณ์',
          status: getMeetingStatus(`${todayDate}T15:00:00`, `${todayDate}T16:00:00`)
        },
        {
          id: 5,
          title: 'ประชุมสรุปงานประจำวัน',
          roomId: 'D',
          startTime: `${todayDate}T16:30:00`,
          endTime: `${todayDate}T17:30:00`,
          organizer: 'คุณประเสริฐ รักงาน',
          status: getMeetingStatus(`${todayDate}T16:30:00`, `${todayDate}T17:30:00`)
        },
        {
          id: 6,
          title: 'ประชุมวางแผนงบประมาณ',
          roomId: 'E',
          startTime: `${todayDate}T14:00:00`,
          endTime: `${todayDate}T16:00:00`,
          organizer: 'คุณสุชาติ การเงิน',
          status: getMeetingStatus(`${todayDate}T14:00:00`, `${todayDate}T16:00:00`)
        }
      ];

      error.value = null; // ยกเลิกข้อความผิดพลาด
      console.warn('ใช้ข้อมูลทดสอบแทน API');
    }
  } finally {
    loading.value = false;
  }
};

// ตรวจสอบสถานะของการประชุมจากเวลาเริ่มต้นและเวลาสิ้นสุด
const getMeetingStatus = (startTime, endTime) => {
  const now = new Date();
  const start = new Date(startTime);
  const end = new Date(endTime);

  if (now < start) {
    return 'upcoming';
  } else if (now >= start && now <= end) {
    return 'ongoing';
  } else {
    return 'completed';
  }
};

// อัพเดทเวลาปัจจุบัน
const updateClock = () => {
  const now = new Date();

  // วันที่ไทย
  dateDisplay.value = now.toLocaleDateString('th-TH', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    weekday: 'long'
  });

  // เวลา
  timeDisplay.value = now.toLocaleTimeString('th-TH', {
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit'
  });

  // อัพเดทสถานะการประชุมทุกนาที
  const seconds = now.getSeconds();
  if (seconds === 0) {
    updateMeetingStatuses();
  }
};

// Interval สำหรับอัพเดทเวลา
let timeInterval;

// อัพเดทสถานะการประชุมตามเวลาปัจจุบัน
const updateMeetingStatuses = () => {
  if (meetings.value.length > 0) {
    meetings.value.forEach(meeting => {
      meeting.status = getMeetingStatus(meeting.startTime, meeting.endTime);
    });
  }
};

// แปลงเวลาให้อยู่ในรูปแบบ HH:MM
const formatTime = (isoTime) => {
  const date = new Date(isoTime);
  return date.toLocaleTimeString('th-TH', {
    hour: '2-digit',
    minute: '2-digit'
  });
};

// แสดงข้อความสถานะ
const getStatusText = (status) => {
  switch(status) {
    case 'ongoing': return 'กำลังประชุม';
    case 'upcoming': return 'รอดำเนินการ';
    case 'completed': return 'เสร็จสิ้น';
    default: return '';
  }
};

// กำหนด class สำหรับสถานะ
const statusClass = (status) => {
  switch(status) {
    case 'ongoing': return 'status-ongoing';
    case 'upcoming': return 'status-upcoming';
    case 'completed': return 'status-completed';
    default: return '';
  }
};

// กำหนด class สำหรับแถวของการประชุม
const meetingRowClass = (meeting, index) => {
  if (meeting.status === 'ongoing') {
    return 'current';
  } else if (meeting.status === 'upcoming') {
    return 'upcoming';
  }
  return '';
};

// Interval สำหรับรีเฟรชข้อมูลอัตโนมัติ
let refreshInterval;

// เมื่อคอมโพเนนต์ถูกโหลด
onMounted(() => {
  // เพิ่ม event listener สำหรับตรวจจับการเปลี่ยนแปลงเต็มจอ
  document.addEventListener('fullscreenchange', handleFullscreenChange);
  document.addEventListener('mozfullscreenchange', handleFullscreenChange);
  document.addEventListener('webkitfullscreenchange', handleFullscreenChange);
  document.addEventListener('MSFullscreenChange', handleFullscreenChange);

  // โหลดการเลือกห้องจาก localStorage
  loadRoomPreferences();
  // ดึงข้อมูลห้องประชุม
  fetchRooms();
  // เริ่มอัพเดทเวลาทุกวินาที
  updateClock();
  timeInterval = setInterval(updateClock, 1000);

  // ดึงข้อมูลการประชุม
  fetchMeetings();

  // ตั้งค่าให้รีเฟรชข้อมูลทุก 5 นาที
  refreshInterval = setInterval(fetchMeetings, 5 * 60 * 1000);
});

// เมื่อคอมโพเนนต์ถูกทำลาย
onUnmounted(() => {
  // ลบ event listener เมื่อคอมโพเนนต์ถูกทำลาย
  document.removeEventListener('fullscreenchange', handleFullscreenChange);
  document.removeEventListener('mozfullscreenchange', handleFullscreenChange);
  document.removeEventListener('webkitfullscreenchange', handleFullscreenChange);
  document.removeEventListener('MSFullscreenChange', handleFullscreenChange);

  clearInterval(timeInterval);
  clearInterval(refreshInterval);
});

// เมื่อ selectedRoomIds เปลี่ยน ให้ดึงข้อมูลใหม่
watch(selectedRoomIds, () => {
  // ดึงข้อมูลใหม่เมื่อมีการเปลี่ยนห้องที่เลือก
  fetchMeetings();
});
</script>
<style scoped>
/* ===== สไตล์ทั่วไป ===== */
body {
  font-family: 'Prompt', 'Kanit', sans-serif;
  margin: 0;
  padding: 0;
}

.container {
  width: 100%;
  max-width: 1920px;
  margin: 0 auto;
  padding: 20px;
  box-sizing: border-box;
}

/* ปรับสไตล์เมื่อเปิดเต็มจอ */
:fullscreen .container {
  padding: 30px;
  background-color: #f0f4f8;
  max-width: none;
}

:-webkit-full-screen .container {
  padding: 30px;
  background-color: #f0f4f8;
  max-width: none;
}

:-ms-fullscreen .container {
  padding: 30px;
  background-color: #f0f4f8;
  max-width: none;
}

/* ===== ส่วนหัว ===== */
.header {
  background-color: #1a5276;
  color: white;
  padding: 20px;
  text-align: center;
  border-radius: 10px 10px 0 0;
  margin-bottom: 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.header h1 {
  margin: 0;
  font-size: 2em;
}

.header-buttons {
  display: flex;
  gap: 10px;
}

.filter-button, .fullscreen-button {
  background-color: white;
  color: #1a5276;
  border: none;
  border-radius: 5px;
  padding: 8px 15px;
  font-size: 1rem;
  cursor: pointer;
  display: flex;
  align-items: center;
  transition: background-color 0.2s;
}

.filter-button:hover, .fullscreen-button:hover {
  background-color: #e6e6e6;
}

.filter-icon, .fullscreen-icon {
  width: 20px;
  height: 20px;
  margin-right: 8px;
}

/* ===== แผงตัวกรองห้องประชุม ===== */
.room-filter-panel {
  background-color: white;
  border-left: 1px solid #ddd;
  border-right: 1px solid #ddd;
  border-bottom: 1px solid #ddd;
  padding: 0;
  transition: all 0.3s ease;
  max-height: 0;
  overflow: hidden;
}

.room-filter-panel.open {
  padding: 20px;
  max-height: 500px;
}

.filter-container {
  margin-bottom: 15px;
}

.filter-title {
  font-size: 1.2rem;
  margin-bottom: 10px;
  font-weight: 600;
}

.room-options {
  display: flex;
  flex-wrap: wrap;
  margin: -5px;
}

.room-option {
  margin: 5px;
}

.room-checkbox {
  display: flex;
  align-items: center;
  background-color: #f8f9fa;
  padding: 8px 15px;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.room-checkbox:hover {
  background-color: #e9ecef;
}

.room-checkbox input {
  margin-right: 8px;
}

.filter-buttons {
  display: flex;
  justify-content: flex-end;
}

.btn {
  padding: 8px 15px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 0.9rem;
  margin-left: 10px;
  transition: background-color 0.2s;
}

.btn-select-all {
  background-color: #2874a6;
  color: white;
}

.btn-select-all:hover {
  background-color: #1a5276;
}

.btn-clear {
  background-color: #6c757d;
  color: white;
}

.btn-clear:hover {
  background-color: #5a6268;
}

.btn-apply {
  background-color: #28a745;
  color: white;
}

.btn-apply:hover {
  background-color: #218838;
}

.btn-retry, .btn-select-room {
  background-color: #2874a6;
  color: white;
  font-size: 1rem;
  padding: 10px 20px;
}

.btn-refresh {
  background-color: #2874a6;
  color: white;
  margin-top: 10px;
}

/* ===== ส่วนแสดงเวลา ===== */
.current-time {
  background-color: #f8f9fa;
  padding: 15px;
  text-align: right;
  font-size: 1.5rem;
  font-weight: 600;
  border-left: 1px solid #ddd;
  border-right: 1px solid #ddd;
  color: #1a5276;
}

/* ===== ส่วนแสดงการโหลด ===== */
.loading-container {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 50px;
  background-color: white;
  border: 1px solid #ddd;
  border-top: none;
  border-radius: 0 0 10px 10px;
}

.loader {
  display: inline-block;
  width: 50px;
  height: 50px;
  border: 5px solid #f3f3f3;
  border-top: 5px solid #3498db;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.loading-text {
  margin-left: 20px;
  font-size: 1.2rem;
  color: #666;
}

/* ===== ส่วนแสดงข้อผิดพลาด ===== */
.error-container {
  padding: 30px;
  text-align: center;
  background-color: #f8d7da;
  border: 1px solid #f5c6cb;
  border-top: none;
  border-radius: 0 0 10px 10px;
}

.error-title {
  color: #dc3545;
  font-size: 1.4rem;
  margin-bottom: 15px;
  font-weight: 600;
}

/* ===== ข้อความเมื่อไม่มีการเลือกห้อง ===== */
.empty-selection {
  text-align: center;
  padding: 50px;
  background-color: #e2f3ff;
  border: 1px solid #b8daff;
  border-top: none;
  border-radius: 0 0 10px 10px;
}

.empty-title {
  font-size: 1.4rem;
  margin-bottom: 15px;
  font-weight: 600;
  color: #6c757d;
}

/* ===== ข้อความเมื่อไม่พบการประชุม ===== */
.no-meetings {
  text-align: center;
  padding: 50px;
  background-color: #fff3cd;
  border: 1px solid #ffeeba;
  border-top: none;
  border-radius: 0 0 10px 10px;
}

.no-meetings-title {
  font-size: 1.4rem;
  margin-bottom: 15px;
  font-weight: 600;
  color: #6c757d;
}

/* ===== ตารางการประชุม ===== */
.meeting-table {
  width: 100%;
  border-collapse: collapse;
  background-color: white;
  border: 1px solid #ddd;
  border-top: none;
  border-radius: 0 0 10px 10px;
  overflow: hidden;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.meeting-table th {
  background-color: #2874a6;
  color: white;
  padding: 15px;
  text-align: left;
  font-weight: 600;
  font-size: 1.2em;
}

.meeting-table td {
  padding: 15px;
  border-top: 1px solid #e9ecef;
  text-align: left;
}

.meeting-table tr:nth-child(even) {
  background-color: #f2f2f2;
}

.meeting-table tr:nth-child(odd) {
  background-color: #ffffff;
}

.meeting-table tbody tr:hover {
  background-color: #e6f7ff;
}

/* แถวที่กำลังประชุมหรือรอการประชุม */
.meeting-table tr.current {
  background-color: #d4edda;
  font-weight: bold;
  animation: highlight 2s infinite;
}

.meeting-table tr.upcoming {
  background-color: #fff3cd;
}

.time-separator-icon {
  width: 20px;
  height: 16px;
  margin: 0 5px;
  vertical-align: middle;
  color: #2874a6;
}

/* สถานะการประชุม */
.status {
  padding: 6px 12px;
  border-radius: 20px;
  font-size: 0.9rem;
  font-weight: 500;
  display: inline-block;
}

.status-ongoing {
  background-color: #28a745;
  color: white;
}

.status-upcoming {
  background-color: #ffc107;
  color: black;
}

.status-completed {
  background-color: #6c757d;
  color: white;
}

.empty-message {
  text-align: center;
  padding: 30px;
  color: #6c757d;
  font-size: 1.2rem;
}

/* ===== ส่วนท้าย ===== */
.footer {
  margin-top: 20px;
  text-align: center;
  font-size: 0.9rem;
  color: #6c757d;
  padding: 10px;
}

/* ===== แอนิเมชัน ===== */
@keyframes highlight {
  0% { background-color: #d4edda; }
  50% { background-color: #c3e6cb; }
  100% { background-color: #d4edda; }
}

/* ===== รองรับการแสดงผลบนอุปกรณ์ต่างๆ (Responsive) ===== */
@media (max-width: 1200px) {
  .header {
    flex-direction: column;
    gap: 15px;
  }

  .header h1 {
    font-size: 2rem;
  }
}

@media (max-width: 992px) {
  .meeting-table th, .meeting-table td {
    padding: 10px;
  }

  .current-time {
    font-size: 1.2rem;
  }
}

@media (max-width: 768px) {
  .room-options {
    flex-direction: column;
  }

  .room-option {
    width: 100%;
  }

  .meeting-table {
    display: block;
    overflow-x: auto;
    font-size: 0.9em;
  }

  .meeting-table th:nth-child(3),
  .meeting-table td:nth-child(3) {
    display: none;
  }

  .header {
    flex-direction: column;
  }

  .filter-button {
    margin-top: 10px;
  }
}

@media (max-width: 576px) {
  .header h1 {
    font-size: 1.5rem;
  }

  .filter-button, .fullscreen-button {
    font-size: 0.8rem;
    padding: 6px 10px;
  }

  .filter-icon, .fullscreen-icon {
    width: 16px;
    height: 16px;
  }

  .meeting-table th:nth-child(4),
  .meeting-table td:nth-child(4) {
    display: none;
  }
}
</style>
