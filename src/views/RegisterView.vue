<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { supabase } from '../utils/supabase'
import '../assets/css/login.css'     // Menggunakan gaya dasar login
import '../assets/css/register.css' // Mengimpor CSS modal terpisah

const router = useRouter()

const name = ref('')
const email = ref('')
const password = ref('')
const errorMessage = ref('')
const isLoading = ref(false)
const showSuccessModal = ref(false)

async function handleRegister() {
  const inputName = name.value.trim()
  const inputEmail = email.value.trim()
  const inputPassword = password.value.trim()

  isLoading.value = true
  errorMessage.value = ''

  try {
    const { data: existingUser } = await supabase
      .from('users')
      .select('email')
      .eq('email', inputEmail)
      .single()

    if (existingUser) {
      errorMessage.value = 'Email ini sudah terdaftar. Silakan langsung masuk.'
      isLoading.value = false
      return
    }

    const { error } = await supabase
      .from('users')
      .insert([{ name: inputName, email: inputEmail, password: inputPassword }])

    if (error) throw error

    isLoading.value = false
    showSuccessModal.value = true

  } catch (err) {
    console.error('Gagal mendaftar:', err.message)
    errorMessage.value = 'Terjadi kesalahan saat mendaftar. Silakan coba lagi.'
    isLoading.value = false
  }
}

function closeModalAndRedirect() {
  showSuccessModal.value = false
  router.push('/login')
}
</script>

<template>
  <div class="page-login">
    <!-- SISI KIRI: BRANDING -->
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
          Bergabung dengan JadwalKu
        </div>
        <h1>Atur jadwal akademikmu dengan lebih mudah dan cerdas.</h1>
        <p>Buat akun dalam hitungan detik dan nikmati pengelolaan jadwal tanpa bentrok.</p>

        <div class="feature-list">
          <div class="feature-item">
            <div class="feature-icon" style="display: flex; align-items: center; justify-content: center;">
              <!-- Ikon Ceklis SVG Line -->
              <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
                <polyline points="20 6 9 17 4 12"></polyline>
              </svg>
            </div>
            <span>Pendaftaran instan dan mandiri</span>
          </div>
          <div class="feature-item">
            <div class="feature-icon" style="display: flex; align-items: center; justify-content: center;">
              <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
                <polyline points="20 6 9 17 4 12"></polyline>
              </svg>
            </div>
            <span>Sinkronisasi data langsung ke database</span>
          </div>
        </div>
      </div>

      <div class="testimonial-box">
        <p class="testimonial-text">"Sangat membantu merapikan jadwal kuliah dan organisasi dalam satu platform."</p>
        <div class="testimonial-author">Mahasiswa Aktif</div>
      </div>
    </div>

    <!-- SISI KANAN: FORM REGISTER -->
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
          <div class="login-subtitle">Pendaftaran Akun Baru</div>
          <h2>Buat akun mahasiswa</h2>
          <p>Masukkan data diri Anda untuk mulai menggunakan kalender akademik.</p>
        </div>

        <div v-if="errorMessage" class="error-alert" role="alert">
          <div class="error-icon" style="display: flex; align-items: center; color: #DC2626;">
            <!-- Ikon Peringatan SVG Line -->
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"></path>
              <line x1="12" y1="9" x2="12" y2="13"></line>
              <line x1="12" y1="17" x2="12.01" y2="17"></line>
            </svg>
          </div>
          <div>
            <div class="error-title">Pendaftaran Gagal</div>
            <div class="error-desc">{{ errorMessage }}</div>
          </div>
        </div>

        <form @submit.prevent="handleRegister">
          <div class="form-group">
            <label class="form-label" for="name">Nama Lengkap</label>
            <input
              id="name"
              v-model="name"
              type="text"
              class="form-input"
              placeholder="Masukkan Nama Anda"
              required
            >
          </div>

          <div class="form-group">
            <label class="form-label" for="email">Email</label>
            <input
              id="email"
              v-model="email"
              type="email"
              class="form-input"
              placeholder="nama@email.com"
              required
            >
          </div>

          <div class="form-group">
            <label class="form-label" for="password">Kata Sandi</label>
            <input
              id="password"
              v-model="password"
              type="password"
              class="form-input"
              placeholder="Buat kata sandi baru"
              required
            >
          </div>

          <button type="submit" class="btn-submit" :disabled="isLoading">
            <span style="display: inline-flex; align-items: center;">
              <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <line x1="5" y1="12" x2="19" y2="12"></line>
                <polyline points="12 5 19 12 12 19"></polyline>
              </svg>
            </span> 
            {{ isLoading ? 'Mendaftarkan akun...' : 'Daftar Sekarang' }}
          </button>

          <div style="text-align: center; margin-top: 16px; font-size: 13px; color: #64748B;">
            Sudah punya akun? 
            <router-link to="/login" style="color: #2563EB; font-weight: 600; text-decoration: none;">Masuk di sini</router-link>
          </div>
        </form>
      </div>
    </div>

    <!-- MODAL SUKSES KUSTOM -->
    <div v-if="showSuccessModal" class="modal-overlay">
      <div class="modal-card">
        <div class="modal-icon-container" style="display: flex; justify-content: center; color: #16A34A; margin-bottom: 16px;">
          <!-- Ikon Sukses / Centang dalam Lingkaran SVG Line -->
          <div style="width: 52px; height: 52px; background: #DCFCE7; border-radius: 50%; display: flex; align-items: center; justify-content: center;">
            <svg width="26" height="26" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
              <polyline points="20 6 9 17 4 12"></polyline>
            </svg>
          </div>
        </div>
        <h3>Pendaftaran Berhasil!</h3>
        <p>Akunmu sudah terdaftar di sistem. Silakan masuk menggunakan email dan kata sandi yang baru saja dibuat.</p>
        <div class="modal-actions">
          <button type="button" class="btn-primary" @click="closeModalAndRedirect">
            Masuk Sekarang
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
