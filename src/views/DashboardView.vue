<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { getSessionUser, logout } from '../stores/session'
import { supabase } from '../utils/supabase'
import '../assets/css/dashboard.css'

const router = useRouter()
const currentUser = ref('')
const userAvatar = ref(null)
const agendas = ref([])
const isLoading = ref(true)

const activeTab = ref('dashboard')
const showLogoutModal = ref(false)

// State Pencarian & Saran Real-time
const searchQuery = ref('')
const searchSuggestions = computed(() => {
  const query = searchQuery.value.trim().toLowerCase()
  if (!query) return []
  return agendas.value.filter(item => 
    (item.title && item.title.toLowerCase().includes(query)) ||
    (item.location && item.location.toLowerCase().includes(query)) ||
    (item.category && item.category.toLowerCase().includes(query))
  ).slice(0, 5)
})

function selectSearchResult(item) {
  searchQuery.value = ''
  openAgendaDetail(item)
}

// Modal Notifikasi
const modalState = ref({ show: false, title: '', message: '', type: 'info' })
function showNotification(titleText, messageText, typeStyle = 'info') {
  modalState.value = { show: true, title: titleText, message: messageText, type: typeStyle }
}
function closeNotification() { modalState.value.show = false }

// Detail & Hapus Agenda
const showDetailModal = ref(false)
const selectedAgendaDetail = ref(null)
const showDeleteConfirmModal = ref(false)
const agendaToDeleteId = ref(null)

function openAgendaDetail(item) {
  selectedAgendaDetail.value = item
  showDetailModal.value = true
}
function closeAgendaDetail() {
  showDetailModal.value = false
  selectedAgendaDetail.value = null
}
function confirmDeleteAgenda(item) {
  agendaToDeleteId.value = item.id
  showDeleteConfirmModal.value = true
}
async function executeDeleteAgenda() {
  if (!agendaToDeleteId.value) return
  try {
    const { error } = await supabase.from('agendas').delete().eq('id', agendaToDeleteId.value)
    if (error) throw error
    agendas.value = agendas.value.filter(item => item.id !== agendaToDeleteId.value)
    showDeleteConfirmModal.value = false
    showDetailModal.value = false
    showNotification('Berhasil!', 'Agenda berhasil dihapus dari sistem.', 'success')
  } catch (err) {
    console.error('Gagal menghapus agenda:', err.message)
    showNotification('Gagal', 'Terjadi kesalahan saat menghapus agenda.', 'error')
  } finally {
    agendaToDeleteId.value = null
  }
}

// Navigasi & Edit
function editAgenda(item) {
  router.push(`/tambah-jadwal?edit=${item.id}`)
}
function editConflictAgenda(timeStr, dateStr) {
  const targetAgenda = agendas.value.find(item => item.date === dateStr && item.time === timeStr)
  if (targetAgenda) router.push(`/tambah-jadwal?edit=${targetAgenda.id}`)
  else router.push('/tambah-jadwal')
}

// Kalender & Waktu
const years = ref(['2026', '2027', '2028'])
const selectedYear = ref('2026')
const monthsList = ['Januari', 'Februari', 'Maret', 'April', 'Mei', 'Juni', 'Juli', 'Agustus', 'September', 'Oktober', 'November', 'Desember']
const shortMonths = ['Jan', 'Feb', 'Mar', 'Apr', 'Mei', 'Jun', 'Jul', 'Ags', 'Sep', 'Okt', 'Nov', 'Des']
const realNow = new Date()
const currentMonthIndex = ref(realNow.getMonth())
const isTransitioning = ref(false)

selectedYear.value = String(realNow.getFullYear())
if (!years.value.includes(selectedYear.value)) years.value.push(selectedYear.value)

const currentMonthYear = computed(() => `${monthsList[currentMonthIndex.value]} ${selectedYear.value}`)

function triggerMonthTransition(callback) {
  isTransitioning.value = true
  setTimeout(() => { callback(); isTransitioning.value = false }, 250)
}
function nextMonth() {
  triggerMonthTransition(() => {
    if (currentMonthIndex.value < 11) currentMonthIndex.value++
    else {
      currentMonthIndex.value = 0
      const nextY = (parseInt(selectedYear.value) + 1).toString()
      if (years.value.includes(nextY)) selectedYear.value = nextY
    }
  })
}
function prevMonth() {
  triggerMonthTransition(() => {
    if (currentMonthIndex.value > 0) currentMonthIndex.value--
    else {
      currentMonthIndex.value = 11
      const prevY = (parseInt(selectedYear.value) - 1).toString()
      if (years.value.includes(prevY)) selectedYear.value = prevY
    }
  })
}
function goToToday() {
  triggerMonthTransition(() => {
    const now = new Date()
    currentMonthIndex.value = now.getMonth()
    selectedYear.value = String(now.getFullYear())
  })
}
function formatAgendaMonth(dateStr) {
  if (!dateStr) return 'Bulan'
  const parts = dateStr.split('-')
  return parts.length === 3 ? shortMonths[parseInt(parts[1], 10) - 1] || 'Bulan' : 'Bulan'
}

// Kalender Dinamis
const calendarGrid = computed(() => {
  const year = parseInt(selectedYear.value, 10)
  const month = currentMonthIndex.value
  const firstDay = new Date(year, month, 1).getDay()
  const startingDay = firstDay === 0 ? 6 : firstDay - 1 
  const daysInMonth = new Date(year, month + 1, 0).getDate()
  const daysArray = []
  
  for (let i = 0; i < startingDay; i++) daysArray.push(null)
  for (let i = 1; i <= daysInMonth; i++) daysArray.push(i)
  while (daysArray.length % 7 !== 0) daysArray.push(null)

  const weeks = []
  for (let i = 0; i < daysArray.length; i += 7) weeks.push(daysArray.slice(i, i + 7))
  return weeks
})

// Filter Data
const selectedCategoryFilter = ref(null)
function filterByCategory(categoryName) {
  selectedCategoryFilter.value = selectedCategoryFilter.value === categoryName ? null : categoryName
}
const filteredAgendas = computed(() => selectedCategoryFilter.value ? agendas.value.filter(item => item.category === selectedCategoryFilter.value) : agendas.value)

const agendasInActiveMonth = computed(() => {
  const monthNum = String(currentMonthIndex.value + 1).padStart(2, '0')
  return agendas.value.filter(item => item.date && item.date.startsWith(`${selectedYear.value}-${monthNum}`))
})

function getAgendasForDate(dayNumber) {
  const formattedDay = String(dayNumber).padStart(2, '0')
  const monthNum = String(currentMonthIndex.value + 1).padStart(2, '0')
  const targetDateStr = `${selectedYear.value}-${monthNum}-${formattedDay}`
  return filteredAgendas.value.filter(item => item.date === targetDateStr)
}
function isToday(dayNumber) {
  const now = new Date()
  return dayNumber === now.getDate() && currentMonthIndex.value === now.getMonth() && parseInt(selectedYear.value) === now.getFullYear()
}

// Statistik
const totalAgendaBulanIni = computed(() => agendasInActiveMonth.value.length)
const totalTugasMendekat = computed(() => agendasInActiveMonth.value.filter(i => i.category === 'Tugas').length)
const scheduleDensity = computed(() => {
  const count = agendasInActiveMonth.value.length
  if (count === 0) return { label: 'Senggang 🟢', statusText: 'Senggang', desc: 'Jadwalmu sangat longgar bulan ini! Waktu yang sempurna untuk me-recharge energi.' }
  if (count <= 3) return { label: 'Ringan 🟢', statusText: 'Santai', desc: 'Ritme akademikmu sangat santai. Cicil tugas kecil agar tidak menumpuk.' }
  if (count <= 7) return totalTugasMendekat.value >= 2 ? { label: 'Stabil 🟡', statusText: 'Fokus Tugas', desc: 'Ada tugas menumpuk. Alokasikan waktu khusus untuk menyelesaikannya.' } : { label: 'Sedang 🟡', statusText: 'Stabil', desc: 'Jadwal terkendali. Tetap jaga konsistensi belajar.' }
  return { label: 'Padat 🔴', statusText: 'Sibuk', desc: 'Jadwalmu sangat padat! Prioritaskan kegiatan mendesak dan jaga kesehatan.' }
})

const conflictStatus = computed(() => {
  if (agendas.value.length < 2) return { hasConflict: false, message: 'Tidak ada jadwal yang bertabrakan di akun ini.' }
  const timeSlotCount = {}
  for (const item of agendas.value) {
    const key = `${item.date}-${item.time}`
    timeSlotCount[key] = (timeSlotCount[key] || 0) + 1
    if (timeSlotCount[key] > 1) return { hasConflict: true, message: `Terdeteksi bentrok pada tanggal ${item.date} pukul ${item.time}!` }
  }
  return { hasConflict: false, message: 'Tidak ada jadwal yang bertabrakan di akun ini.' }
})

// Notifikasi Otomatis
const dismissedNotifs = ref([])
function markAsCompleted(agendaKey) {
  const cleanKey = String(agendaKey)
  if (cleanKey && !dismissedNotifs.value.includes(cleanKey)) {
    dismissedNotifs.value = [...dismissedNotifs.value, cleanKey]
    if (currentUser.value) localStorage.setItem(`dismissedNotifs_${currentUser.value}`, JSON.stringify(dismissedNotifs.value))
  }
}

const automatedNotifications = computed(() => {
  const list = []
  const now = new Date()
  const todayStr = `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, '0')}-${String(now.getDate()).padStart(2, '0')}`
  const currentHourMin = `${String(now.getHours()).padStart(2, '0')}:${String(now.getMinutes()).padStart(2, '0')}`

  const timeSlots = {}
  agendas.value.forEach(item => {
    const key = `${item.date}-${item.time}`
    if (!timeSlots[key]) timeSlots[key] = []
    timeSlots[key].push(item)
  })

  Object.entries(timeSlots).forEach(([key, items]) => {
    const notifId = `conflict-${key}`
    if (items.length > 1) {
      const titles = items.map(i => `"${i.title}"`).join(' dan ')
      list.push({ id: notifId, agendaKey: notifId, isConflict: true, conflictTime: items[0].time, conflictDate: items[0].date, type: 'conflict', iconSvg: '<svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"></path><line x1="12" y1="9" x2="12" y2="13"></line><line x1="12" y1="17" x2="12.01" y2="17"></line></svg>', title: 'Jadwal Bentrok!', desc: `Agenda ${titles} bertabrakan pada tanggal ${items[0].date} pukul ${items[0].time}. Harap edit salah satu jadwal.`, timeInfo: `${items[0].date} • ${items[0].time}` })
    }
  })

  agendas.value.forEach(item => {
    if (!item.date || !item.time || dismissedNotifs.value.includes(String(item.id))) return
    if (item.date < todayStr || (item.date === todayStr && item.time < currentHourMin)) {
      list.push({ id: `missed-${item.id}`, agendaKey: String(item.id), isConflict: false, type: 'missed', iconSvg: '<svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"></circle><polyline points="12 6 12 12 16 14"></polyline></svg>', title: 'Agenda Terlewat', desc: `"${item.title}" pada tanggal ${item.date} (${item.time}) telah terlewat.`, timeInfo: `${item.date} • ${item.time}` })
    } else if (item.date === todayStr) {
      list.push({ id: `reminder-${item.id}`, agendaKey: String(item.id), isConflict: false, type: 'reminder', iconSvg: '<svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"></path><path d="M13.73 21a2 2 0 0 1-3.46 0"></path></svg>', title: `Pengingat: ${item.reminder || 'Agenda Hari Ini'}`, desc: `"${item.title}" dimulai hari ini pukul ${item.time} di ${item.location || 'Lokasi umum'}.`, timeInfo: `Hari ini • ${item.time}` })
    }
  })
  return list
})

// Inisiasi Data
onMounted(async () => {
  const user = getSessionUser()
  if (!user) { router.push('/login'); return }
  currentUser.value = user
  const savedDismissed = localStorage.getItem(`dismissedNotifs_${user}`)
  if (savedDismissed) {
    try { dismissedNotifs.value = JSON.parse(savedDismissed) } catch (e) { console.error('Gagal membaca notifikasi', e) }
  }
  await fetchUserProfile(user)
  await fetchAgendas(user)
})

async function fetchUserProfile(identifier) {
  try {
    let query = supabase.from('users').select('*')
    query = identifier.includes('@') ? query.eq('email', identifier) : query.eq('name', identifier)
    const { data, error } = await query.maybeSingle()
    if (error) return
    if (data) {
      if (data.years) {
        years.value = JSON.parse(data.years)
        if (data.active_year && years.value.includes(data.active_year)) selectedYear.value = data.active_year
      }
      if (data.avatar_url) userAvatar.value = data.avatar_url
    }
  } catch (err) { console.error('Gagal memuat profil:', err.message) }
}

async function fetchAgendas(email) {
  isLoading.value = true
  try {
    const { data, error } = await supabase.from('agendas').select('*').eq('user_email', email).order('date', { ascending: true })
    if (error) throw error
    if (data) agendas.value = data
  } catch (err) { console.error('Gagal memuat agenda:', err.message) }
  finally { isLoading.value = false }
}

async function handleYearChange(event) {
  const val = event.target.value
  selectedYear.value = val
  try { await supabase.from('users').update({ active_year: val }).eq('email', currentUser.value) } 
  catch (e) { console.log('Gagal memperbarui active year:', e.message) }
}

function executeLogout() { logout(); router.push('/login') }
</script>

<template>
  <div class="page-dashboard">
    <div class="app-container">
      
      <!-- SIDEBAR KIRI -->
      <aside class="sidebar">
        <div>
          <div class="sidebar-brand">
            <span class="brand-icon" style="display: flex; align-items: center; color: #FFFFFF;">
              <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <rect x="3" y="4" width="18" height="18" rx="2" ry="2"></rect>
                <line x1="16" y1="2" x2="16" y2="6"></line>
                <line x1="8" y1="2" x2="8" y2="6"></line>
                <line x1="3" y1="10" x2="21" y2="10"></line>
              </svg>
            </span>
            <span class="brand-name">JadwalKu</span>
          </div>

          <ul class="sidebar-menu">
            <li>
              <a href="#" class="menu-item" :class="{ active: activeTab === 'dashboard' }" @click.prevent="activeTab = 'dashboard'">
                <span style="display: inline-flex; align-items: center;">
                  <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="7" height="9"></rect><rect x="14" y="3" width="7" height="5"></rect><rect x="14" y="12" width="7" height="9"></rect><rect x="3" y="16" width="7" height="5"></rect></svg>
                </span> Dashboard
              </a>
            </li>
            <li>
              <a href="#" class="menu-item" :class="{ active: activeTab === 'agenda' }" @click.prevent="activeTab = 'agenda'">
                <span style="display: inline-flex; align-items: center;">
                  <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path><polyline points="14 2 14 8 20 8"></polyline><line x1="16" y1="13" x2="8" y2="13"></line><line x1="16" y1="17" x2="8" y2="17"></line><polyline points="10 9 9 9 8 9"></polyline></svg>
                </span> Agenda Kuliah
              </a>
            </li>
            <li>
              <a href="#" class="menu-item" :class="{ active: activeTab === 'notifikasi' }" @click.prevent="activeTab = 'notifikasi'" style="position: relative;">
                <span style="display: inline-flex; align-items: center;">
                  <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"></path><path d="M13.73 21a2 2 0 0 1-3.46 0"></path></svg>
                </span> Notifikasi
                <span v-if="automatedNotifications.length > 0" style="position: absolute; right: 12px; background: #EF4444; color: #FFFFFF; font-size: 10px; font-weight: 700; padding: 2px 6px; border-radius: 10px;">
                  {{ automatedNotifications.length }}
                </span>
              </a>
            </li>
            <li>
              <router-link to="/profil" class="menu-item">
                <span style="display: inline-flex; align-items: center;">
                  <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="3"></circle><path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06-.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z"></path></svg>
                </span> Pengaturan Profil
              </router-link>
            </li>
          </ul>

          <div class="sidebar-section-title">
            <span>TAHUN AKADEMIK</span>
          </div>
          <div class="sidebar-filter">
            <select :value="selectedYear" @change="handleYearChange" class="semester-select">
              <option v-for="yr in years" :key="yr" :value="yr">Tahun {{ yr }}</option>
            </select>
          </div>

          <div class="sidebar-section-title" style="display: flex; justify-content: space-between; align-items: center;">
            <span>KATEGORI FILTER</span>
            <button v-if="selectedCategoryFilter" @click="selectedCategoryFilter = null" style="background: none; border: none; color: #F87171; cursor: pointer; font-size: 10px; font-weight: 600;">Reset</button>
          </div>
          <ul class="category-list">
            <li @click="filterByCategory('Kuliah')" class="category-hover-item" :style="selectedCategoryFilter === 'Kuliah' ? 'background: #1E293B; padding: 6px 8px; border-radius: 6px; border-left: 3px solid #8B5CF6; cursor: pointer;' : 'padding: 6px 8px; cursor: pointer; border-radius: 6px;'">
              <span class="dot purple"></span> Kuliah <span class="count">{{ agendas.filter(i => i.category === 'Kuliah').length }}</span>
            </li>
            <li @click="filterByCategory('Organisasi')" class="category-hover-item" :style="selectedCategoryFilter === 'Organisasi' ? 'background: #1E293B; padding: 6px 8px; border-radius: 6px; border-left: 3px solid #3B82F6; cursor: pointer;' : 'padding: 6px 8px; cursor: pointer; border-radius: 6px;'">
              <span class="dot blue"></span> Organisasi <span class="count">{{ agendas.filter(i => i.category === 'Organisasi').length }}</span>
            </li>
            <li @click="filterByCategory('Tugas')" class="category-hover-item" :style="selectedCategoryFilter === 'Tugas' ? 'background: #1E293B; padding: 6px 8px; border-radius: 6px; border-left: 3px solid #10B981; cursor: pointer;' : 'padding: 6px 8px; cursor: pointer; border-radius: 6px;'">
              <span class="dot green"></span> Tugas <span class="count">{{ agendas.filter(i => i.category === 'Tugas').length }}</span>
            </li>
          </ul>
        </div>

        <div>
          <div class="sidebar-conflict-box" :style="conflictStatus.hasConflict ? 'margin-top: 24px; background-color: #7F1D1D; border-color: #991B1B;' : 'margin-top: 24px;'">
            <div class="conflict-title" :style="conflictStatus.hasConflict ? 'color: #FEE2E2; display: flex; align-items: center; gap: 6px;' : 'display: flex; align-items: center; gap: 6px;'">
              <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"></path>
                <line x1="12" y1="9" x2="12" y2="13"></line>
                <line x1="12" y1="17" x2="12.01" y2="17"></line>
              </svg>
              {{ conflictStatus.hasConflict ? 'Peringatan Bentrok!' : 'Status Konflik' }}
            </div>
            <div class="conflict-desc" :style="conflictStatus.hasConflict ? 'color: #FECACA;' : ''">
              {{ conflictStatus.message }}
            </div>
          </div>

          <div class="sidebar-footer" style="margin-top: 16px; padding-top: 16px; border-top: 1px solid #1E293B;">
            <router-link to="/profil" class="user-profile clickable-profile" style="margin-bottom: 12px; display: flex; align-items: center; gap: 10px; text-decoration: none;">
              <div class="user-avatar">
                <img v-if="userAvatar" :src="userAvatar" alt="Avatar" />
                <span v-else>{{ currentUser ? currentUser.charAt(0).toUpperCase() : 'M' }}</span>
              </div>
              <div style="min-width: 0; flex: 1; overflow: hidden;">
                <div class="user-name" style="font-size: 12px; font-weight: 600; color: #FFFFFF; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">
                  {{ currentUser || 'Mahasiswa' }}
                </div>
                <div class="user-role" style="font-size: 11px; color: #94A3B8;">Akun Aktif</div>
              </div>
            </router-link>
            <button class="btn-logout-sidebar" @click="showLogoutModal = true">Keluar</button>
          </div>
        </div>
      </aside>

      <!-- KONTEN UTAMA -->
      <main class="main-content">
        <header class="top-navbar">
          <div class="search-container">
            <span class="search-icon" style="display: flex; align-items: center; color: #94A3B8;">
              <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line></svg>
            </span>
            <input type="text" class="search-input" v-model="searchQuery" placeholder="Cari agenda atau mata kuliah..." autocomplete="off">
            <ul v-if="searchSuggestions.length > 0" class="search-suggestions-dropdown">
              <li v-for="item in searchSuggestions" :key="item.id" class="search-suggestion-item" @click="selectSearchResult(item)">
                <div class="suggestion-title" style="display: flex; align-items: center; gap: 6px;">
                  <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"></path><path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"></path></svg>
                  {{ item.title }}
                </div>
                <div class="suggestion-meta">📅 {{ item.date }} • ⏰ {{ item.time }} • 📍 {{ item.location || 'Lokasi umum' }}</div>
              </li>
            </ul>
            <div v-else-if="searchQuery.trim().length > 0" class="search-suggestions-dropdown" style="padding: 14px; text-align: center; font-size: 12px; color: #64748B;">
              Tidak ada agenda yang cocok.
            </div>
          </div>

          <div class="user-nav-actions">
            <button class="notif-btn" @click="activeTab = 'notifikasi'" title="Notifikasi" style="position: relative; display: flex; align-items: center; justify-content: center;">
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"></path><path d="M13.73 21a2 2 0 0 1-3.46 0"></path></svg>
              <span v-if="automatedNotifications.length > 0" style="position: absolute; top: 4px; right: 4px; width: 8px; height: 8px; background: #EF4444; border-radius: 50%;"></span>
            </button>
            <router-link to="/profil" class="user-profile clickable-profile" style="text-decoration: none;">
              <div class="user-avatar">
                <img v-if="userAvatar" :src="userAvatar" alt="Avatar" />
                <span v-else>{{ currentUser ? currentUser.charAt(0).toUpperCase() : 'M' }}</span>
              </div>
              <div class="user-info">
                <div class="user-name">{{ currentUser || 'Mahasiswa' }}</div>
                <div class="user-role">Mahasiswa Aktif</div>
              </div>
            </router-link>
          </div>
        </header>

        <div class="dashboard-body">
          <!-- TAB 1: DASHBOARD -->
          <div v-if="activeTab === 'dashboard'" class="content-section">
            <div class="welcome-banner">
              <div>
                <div class="welcome-sub">TAHUN AKTIF: {{ selectedYear }} <span v-if="selectedCategoryFilter" style="color: #2563EB; font-weight: 700;">| Filter: {{ selectedCategoryFilter }}</span></div>
                <h1>Dashboard Akademik</h1>
                <p>Kelola jadwal kuliah, organisasi, dan tugasmu dengan lancar.</p>
              </div>
              <div class="action-buttons">
                <router-link to="/tambah-jadwal" class="btn-primary" style="text-decoration: none; display: inline-flex; align-items: center; gap: 6px;">
                  <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><line x1="12" y1="5" x2="12" y2="19"></line><line x1="5" y1="12" x2="19" y2="12"></line></svg>
                  Tambah Agenda
                </router-link>
              </div>
            </div>

            <div class="stats-row">
              <div class="stat-card">
                <div class="stat-icon purple-bg" style="display: flex; align-items: center; justify-content: center; color: #8B5CF6;">
                  <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="4" width="18" height="18" rx="2" ry="2"></rect><line x1="16" y1="2" x2="16" y2="6"></line><line x1="8" y1="2" x2="8" y2="6"></line><line x1="3" y1="10" x2="21" y2="10"></line></svg>
                </div>
                <div><div class="stat-num">{{ totalAgendaBulanIni }}</div><div class="stat-label">Agenda Bulan Ini</div></div>
                <span class="stat-badge purple-text">Aktif</span>
              </div>
              <div class="stat-card">
                <div class="stat-icon orange-bg" style="display: flex; align-items: center; justify-content: center; color: #F97316;">
                  <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path><polyline points="14 2 14 8 20 8"></polyline><line x1="16" y1="13" x2="8" y2="13"></line><line x1="16" y1="17" x2="8" y2="17"></line></svg>
                </div>
                <div><div class="stat-num">{{ totalTugasMendekat }}</div><div class="stat-label">Tugas / Ujian Mendekat</div></div>
                <span class="stat-badge orange-text">Segera</span>
              </div>
              <div class="stat-card">
                <div class="stat-icon green-bg" style="display: flex; align-items: center; justify-content: center; color: #10B981;">
                  <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="22 12 18 12 15 21 9 3 6 12 2 12"></polyline></svg>
                </div>
                <div><div class="stat-num" style="font-size: 16px; margin-top: 4px;">{{ scheduleDensity.label }}</div><div class="stat-label">Kepadatan Jadwal</div></div>
                <span class="stat-badge green-text">{{ scheduleDensity.statusText }}</span>
              </div>
            </div>

            <div class="calendar-layout-grid">
              <div class="calendar-card-main">
                <div class="cal-main-header">
                  <div class="month-navigation">
                    <button class="nav-arrow" @click="prevMonth">&lt;</button>
                    <h2>{{ currentMonthYear }}</h2>
                    <button class="nav-arrow" @click="nextMonth">&gt;</button>
                  </div>
                  <div class="cal-view-switches">
                    <button class="view-btn active" @click="goToToday">Hari Ini</button>
                    <button class="view-btn" @click="activeTab = 'agenda'">Daftar</button>
                  </div>
                </div>
                
                <div v-if="isLoading" style="text-align: center; padding: 40px; color: #64748B;">Memuat kalender...</div>
                <div v-else>
                  <table class="monthly-table" :class="{ 'calendar-fade-transition': isTransitioning }">
                    <thead><tr><th>Sen</th><th>Sel</th><th>Rab</th><th>Kam</th><th>Jum</th><th>Sab</th><th>Min</th></tr></thead>
                    <tbody>
                      <tr v-for="(week, weekIndex) in calendarGrid" :key="weekIndex">
                        <td v-for="(day, dayIndex) in week" :key="dayIndex">
                          <template v-if="day">
                            <span class="date-num" :class="{ 'today-circle': isToday(day) }">{{ day }}</span>
                            <div v-for="item in getAgendasForDate(day)" :key="item.id" class="event-pill blue-pill" @click="openAgendaDetail(item)">{{ item.title }}</div>
                          </template>
                          <template v-else><span class="date-num text-muted">-</span></template>
                        </td>
                      </tr>
                    </tbody>
                  </table>
                </div>
              </div>

              <div class="right-sidebar-panel">
                <div class="upcoming-header"><h3>Agenda Terdekat</h3><span class="badge-count">{{ filteredAgendas.length }}</span></div>
                <div v-if="filteredAgendas.length === 0" style="font-size: 12px; color: #64748B; padding: 10px 0;">Belum ada agenda terdekat.</div>
                <div v-else>
                  <div v-for="item in filteredAgendas.slice(0, 3)" :key="item.id" class="upcoming-item" @click="openAgendaDetail(item)" style="cursor: pointer;">
                    <div class="up-date">{{ item.date.split('-')[2] }}<br><small>{{ formatAgendaMonth(item.date) }}</small></div>
                    <div><div class="up-title">{{ item.title }}</div><div class="up-location">{{ item.time }} • {{ item.location || 'Lokasi umum' }}</div></div>
                  </div>
                </div>
                <div class="ai-suggestion-box">
                  <div class="ai-title" style="display: flex; align-items: center; gap: 6px;">
                    <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"></path></svg>
                    Saran JadwalKu
                  </div>
                  <div class="ai-desc">{{ scheduleDensity.desc }}</div>
                </div>
              </div>
            </div>
          </div>

          <!-- TAB 2: AGENDA -->
          <div v-if="activeTab === 'agenda'" class="content-section">
            <div class="welcome-banner">
              <div><div class="welcome-sub">MANAJEMEN DATABASE</div><h1>Daftar Seluruh Agenda</h1><p>Jadwal tersimpan di akun privat Anda.</p></div>
            </div>
            <div class="calendar-card-main">
              <div v-if="filteredAgendas.length === 0" style="text-align: center; padding: 40px; color: #64748B;">Belum ada agenda tersimpan.</div>
              <div v-else style="display: flex; flex-direction: column; gap: 12px;">
                <div v-for="item in filteredAgendas" :key="item.id" @click="openAgendaDetail(item)" style="display: flex; justify-content: space-between; align-items: center; background: #F8FAFC; padding: 16px; border-radius: 10px; border: 1px solid #E2E8F0; cursor: pointer;">
                  <div>
                    <div style="font-weight: 700; font-size: 15px; color: #0F172A;">{{ item.title }}</div>
                    <div style="font-size: 13px; color: #64748B;">📍 {{ item.location || '-' }} • ⏰ {{ item.time }} • Kategori: {{ item.category }}</div>
                  </div>
                  <div><span class="event-pill blue-pill">{{ item.date }}</span></div>
                </div>
              </div>
            </div>
          </div>

          <!-- TAB 3: NOTIFIKASI -->
          <div v-if="activeTab === 'notifikasi'" class="content-section">
            <div class="welcome-banner">
              <div><div class="welcome-sub">PUSAT INFORMASI</div><h1>Notifikasi & Pengingat Otomatis</h1><p>Pantau pengingat detail jadwal, agenda terlewat, serta peringatan konflik secara real-time.</p></div>
            </div>
            <div class="calendar-card-main" style="padding: 24px;">
              <div v-if="automatedNotifications.length === 0" style="text-align: center; padding: 40px; color: #64748B;">🎉 Tidak ada pemberitahuan atau peringatan saat ini. Semua jadwal aman!</div>
              <div v-else class="notif-list-container">
                <div v-for="notif in automatedNotifications" :key="notif.id" class="notif-card-item" :class="{'notif-conflict': notif.type === 'conflict', 'notif-missed': notif.type === 'missed', 'notif-reminder': notif.type === 'reminder'}">
                  <div class="notif-icon-box" v-html="notif.iconSvg"></div>
                  <div class="notif-content-box">
                    <div class="notif-header-row"><span class="notif-title">{{ notif.title }}</span><span class="notif-badge-time">{{ notif.timeInfo }}</span></div>
                    <p class="notif-desc">{{ notif.desc }}</p>
                  </div>
                  <div class="notif-action-box">
                    <button v-if="notif.isConflict" type="button" class="notif-btn-action btn-action-conflict" @click="editConflictAgenda(notif.conflictTime, notif.conflictDate)">✏️ Edit Jadwal</button>
                    <button v-else type="button" class="notif-btn-action btn-action-completed" @click="markAsCompleted(notif.agendaKey)">✓ Selesai</button>
                  </div>
                </div>
              </div>
            </div>
          </div>

        </div>
      </main>
    </div>

    <!-- MODAL DETAIL AGENDA -->
    <div v-if="showDetailModal" class="modal-overlay" id="modalDetailAgenda">
      <div class="modal-detail-card">
        <div class="modal-detail-header">
          <div class="modal-header-top"><span class="badge-category">{{ selectedAgendaDetail?.category }}</span><button class="btn-close-circle" @click="closeAgendaDetail">✕</button></div>
          <h3>{{ selectedAgendaDetail?.title }}</h3><p>Detail lengkap informasi agenda kegiatan Anda</p>
        </div>
        <div class="modal-detail-body">
          <div class="detail-grid-two">
            <div class="detail-box"><span class="detail-box-icon">📅</span><div><span class="detail-label">Tanggal</span><span class="detail-value">{{ selectedAgendaDetail?.date }}</span></div></div>
            <div class="detail-box"><span class="detail-box-icon">⏰</span><div><span class="detail-label">Waktu</span><span class="detail-value">{{ selectedAgendaDetail?.time }}</span></div></div>
          </div>
          <div class="detail-info-list">
            <div class="detail-info-item"><span class="info-icon">📍</span><div><span class="info-label">Lokasi / Ruangan</span><span class="info-value">{{ selectedAgendaDetail?.location || '-' }}</span></div></div>
            <div class="detail-info-item"><span class="info-icon">🔁</span><div><span class="info-label">Pengulangan</span><span class="info-value">{{ selectedAgendaDetail?.repeat_type || 'Tidak diulang' }}</span></div></div>
          </div>
          <div v-if="selectedAgendaDetail?.description" class="detail-note-box"><span class="note-title">📝 CATATAN / DESKRIPSI</span><p>{{ selectedAgendaDetail?.description }}</p></div>
        </div>
        <div class="modal-detail-footer">
          <button type="button" class="btn-ghost-danger" @click="confirmDeleteAgenda(selectedAgendaDetail)" style="display: inline-flex; align-items: center; gap: 6px;">
            <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path></svg>
            Hapus Agenda
          </button>
          <div class="footer-right-btns">
            <button type="button" class="btn-outline" @click="editAgenda(selectedAgendaDetail)" style="display: inline-flex; align-items: center; gap: 6px;">
              <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"></path><path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"></path></svg>
              Edit
            </button>
            <button type="button" class="btn-primary" @click="closeAgendaDetail">Tutup</button>
          </div>
        </div>
      </div>
    </div>

    <!-- MODAL HAPUS & NOTIF -->
    <div v-if="showDeleteConfirmModal" class="modal-overlay">
      <div class="modal-card" style="text-align: center;">
        <div style="font-size: 42px; margin-bottom: 12px; display: flex; justify-content: center; color: #DC2626;">
          <svg width="40" height="40" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path></svg>
        </div>
        <h3 style="margin-bottom: 8px;">Hapus Agenda?</h3><p style="font-size: 13px; color: #64748B; margin-bottom: 24px;">Yakin ingin menghapus agenda ini?</p>
        <div style="display: flex; gap: 10px;"><button class="btn-outline" style="flex: 1;" @click="showDeleteConfirmModal = false">Batal</button><button style="flex: 1; padding: 10px; border-radius: 8px; background: #DC2626; color: #FFF; border: none; cursor: pointer;" @click="executeDeleteAgenda">Ya, Hapus</button></div>
      </div>
    </div>
    
    <div v-if="modalState.show" class="modal-overlay">
      <div class="modal-card" style="text-align: center;">
        <div style="display: flex; justify-content: center; margin-bottom: 16px;">
          <div v-if="modalState.type === 'success'" style="width: 52px; height: 52px; background: #DCFCE7; color: #16A34A; border-radius: 50%; display: flex; align-items: center; justify-content: center;">
            <svg width="26" height="26" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="20 6 9 17 4 12"></polyline></svg>
          </div>
          <div v-else style="width: 52px; height: 52px; background: #FEE2E2; color: #DC2626; border-radius: 50%; display: flex; align-items: center; justify-content: center;">
            <svg width="26" height="26" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>
          </div>
        </div>
        <h3 style="margin-bottom: 8px;">{{ modalState.title }}</h3><p style="font-size: 13px; color: #64748B; margin-bottom: 24px;">{{ modalState.message }}</p>
        <button class="btn-primary" style="width: 100%;" @click="closeNotification">OK</button>
      </div>
    </div>

    <!-- MODAL LOGOUT -->
    <div v-if="showLogoutModal" class="modal-overlay">
      <div class="modal-card" style="text-align: center;">
        <div style="font-size: 42px; margin-bottom: 12px; display: flex; justify-content: center; color: #DC2626;">
          <svg width="40" height="40" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"></path><polyline points="16 17 21 12 16 7"></polyline><line x1="21" y1="12" x2="9" y2="12"></line></svg>
        </div>
        <h3 style="margin-bottom: 8px;">Keluar dari Akun?</h3>
        <div style="display: flex; gap: 10px;"><button class="btn-outline" style="flex: 1;" @click="showLogoutModal = false">Batal</button><button class="btn-primary" style="flex: 1; background: #DC2626;" @click="executeLogout">Ya, Keluar</button></div>
      </div>
    </div>

  </div>
</template>
