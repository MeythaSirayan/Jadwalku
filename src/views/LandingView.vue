<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { login } from '../stores/session'
import { supabase } from '../utils/supabase' // Mengimpor koneksi Supabase
import '../assets/css/login.css'

const router = useRouter()

const email = ref('')
const password = ref('')
const remember = ref(false)
const showPassword = ref(false)
const hasError = ref(false)
const isLoading = ref(false)
const isExiting = ref(false) // State untuk efek transisi smooth

onMounted(() => {
  const remembered = localStorage.getItem('rememberedEmail')
  if (remembered) {
    email.value = remembered
    remember.value = true
  }
})

function clearError() {
  hasError.value = false
}

// Fungsi Login terkoneksi Supabase Database
async function handleLogin() {
  isLoading.value = true
  clearError()

  try {
    // Pastikan kueri mengarah langsung ke tabel users dengan benar
    const { data, error } = await supabase
      .from('users')
      .select('*')
      .eq('email', email.value)
      .eq('password', password.value)
      .maybeSingle()

    if (error) throw error

    if (data) {
      // Simpan sesi user yang aktif
      login(data.email) // atau data.name sesuai struktur tabelmu
      
      // Memicu efek transisi smooth sebelum pindah halaman
      isExiting.value = true
      
      setTimeout(() => {
        router.push('/dashboard')
      }, 300) // Waktu delay disesuaikan dengan durasi CSS transition
    } else {
      // Menampilkan error UI, bukan alert browser
      hasError.value = true
    }
  } catch (err) {
    console.error('Terjadi kesalahan:', err.message)
    hasError.value = true
  } finally {
    isLoading.value = false
  }
}

function handleForgotPassword() {
  alert('Silakan hubungi administrator kampus untuk pemulihan kata sandi.')
}
</script>

<template>
  <div class="page-login" :class="{ 'fade-out': isExiting }">
    <!-- SISI KIRI: BRANDING & FITUR -->
    <div class="left-panel">
      <div class="brand">
        <div class="brand-icon" style="display: flex; align-items: center; color: #FFFFFF;">
          <!-- Ikon Kalender SVG Line -->
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <rect x="3" y="4" width="18" height="18" rx="2" ry="2"></rect>
            <line x1="16" y1="2" x2="16" y2="6"></line>
            <line x1="8" y1="2" x2="8" y2="6"></line>
            <line x1="3" y1="10" x2="21" y2="10"></line>
          </svg>
        </div>
        <div class="brand-name">JadwalKu</div>
      </div>

      <div class="left-content">
        <div class="badge">
          <span style="display: inline-flex; align-items: center;">
            <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="margin-right: 4px;">
              <path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"></path>
            </svg>
          </span>
          Kalender akademik pintar
        </div>
        <h1>Kuliah lebih teratur, tanpa jadwal bertabrakan.</h1>
        <p>Susun perkuliahan, rapat organisasi, dan tenggat tugas dalam satu kalender yang otomatis mendeteksi konflik.</p>

        <div class="feature-list">
          <div class="feature-item">
            <div class="feature-icon" style="display: flex; align-items: center; justify-content: center;">
              <!-- Ikon Ceklis SVG Line -->
              <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
                <polyline points="20 6 9 17 4 12"></polyline>
              </svg>
            </div>
            <span>Deteksi konflik sebelum agenda disimpan</span>
          </div>
          <div class="feature-item">
            <div class="feature-icon" style="display: flex; align-items: center; justify-content: center;">
              <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
                <polyline points="20 6 9 17 4 12"></polyline>
              </svg>
            </div>
            <span>Ringkasan beban akademik mingguan</span>
          </div>
          <div class="feature-item">
            <div class="feature-icon" style="display: flex; align-items: center; justify-content: center;">
              <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
                <polyline points="20 6 9 17 4 12"></polyline>
              </svg>
            </div>
            <span>Pengingat tugas dan ujian tepat waktu</span>
          </div>
        </div>
      </div>

      <div class="testimonial-box">
        <p class="testimonial-text">"JadwalKu membantu saya menghindari rapat BEM yang bersamaan dengan kelas praktikum."</p>
        <div class="testimonial-author">Nadia • Mahasiswa Sistem Informasi</div>
      </div>
    </div>

    <!-- SISI KANAN: FORM LOGIN -->
    <div class="right-panel">
      <div class="login-card">
        <div class="mobile-brand">
          <div class="brand-icon" style="display: flex; align-items: center; color: #FFFFFF;">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <rect x="3" y="4" width="18" height="18" rx="2" ry="2"></rect>
              <line x1="16" y1="2" x2="16" y2="6"></line>
              <line x1="8" y1="2" x2="8" y2="6"></line>
              <line x1="3" y1="10" x2="21" y2="10"></line>
            </svg>
          </div>
          <div class="brand-name">JadwalKu</div>
        </div>

        <div class="login-header">
          <div class="login-subtitle">Selamat datang kembali</div>
          <h2>Masuk ke akun mahasiswa</h2>
          <p>Gunakan email terdaftar untuk melanjutkan ke kalender akademik Anda.</p>
        </div>

        <div v-if="hasError" class="error-alert" role="alert">
          <div class="error-icon" style="display: flex; align-items: center; color: #DC2626;">
            <!-- Ikon Peringatan / Segitiga SVG Line -->
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"></path>
              <line x1="12" y1="9" x2="12" y2="13"></line>
              <line x1="12" y1="17" x2="12.01" y2="17"></line>
            </svg>
          </div>
          <div>
            <div class="error-title">Email atau kata sandi belum tepat</div>
            <div class="error-desc">Periksa kembali email dan kata sandi, lalu coba lagi.</div>
          </div>
        </div>

        <form @submit.prevent="handleLogin">
          <div class="form-group">
            <label class="form-label" for="email">Email</label>
            <input
              id="email"
              v-model="email"
              type="email"
              class="form-input"
              :class="{ error: hasError }"
              placeholder="Masukkan Email (cth: nama@email.com)"
              required
              @input="clearError"
            >
          </div>

          <div class="form-group">
            <label class="form-label" for="password">Kata sandi</label>
            <div class="input-container">
              <input
                id="password"
                v-model="password"
                :type="showPassword ? 'text' : 'password'"
                class="form-input"
                :class="{ error: hasError }"
                placeholder="Masukkan Password"
                required
                @input="clearError"
              >
              <button
                type="button"
                class="password-toggle"
                :aria-label="showPassword ? 'Sembunyikan kata sandi' : 'Tampilkan kata sandi'"
                @click="showPassword = !showPassword"
              >
                <!-- Ikon Gembok / Mata SVG Line -->
                <svg v-if="!showPassword" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"></path>
                  <circle cx="12" cy="12" r="3"></circle>
                </svg>
                <svg v-else width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19m-6.72-1.07a3 3 0 1 1-4.24-4.24"></path>
                  <line x1="1" y1="1" x2="23" y2="23"></line>
                </svg>
              </button>
            </div>
            <span v-if="hasError" class="error-helper">⚠️ Periksa kembali email atau kata sandi Anda.</span>
          </div>

          <div class="form-options">
            <label class="remember-me">
              <input v-model="remember" type="checkbox"> Ingat saya
            </label>
            <a href="#" class="forgot-password" @click.prevent="handleForgotPassword">Lupa kata sandi?</a>
          </div>

          <button type="submit" class="btn-submit" :disabled="isLoading">
            <span style="display: inline-flex; align-items: center;">
              <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <line x1="5" y1="12" x2="19" y2="12"></line>
                <polyline points="12 5 19 12 12 19"></polyline>
              </svg>
            </span> 
            {{ isLoading ? 'Memeriksa Database...' : 'Masuk ke JadwalKu' }}
          </button>

          <!-- Tautan Tambahan untuk Pindah ke Halaman Daftar -->
          <div style="text-align: center; margin-top: 16px; font-size: 13px; color: #64748B;">
            Belum punya akun? 
            <router-link to="/register" style="color: #2563EB; font-weight: 600; text-decoration: none;">Daftar di sini</router-link>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>
